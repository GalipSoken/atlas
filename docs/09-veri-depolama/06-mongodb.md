# 9.6 MongoDB

## Nedir?
**MongoDB**, en yaygın **document (belge)** veritabanıdır. Veriyi tablo satırları yerine **JSON benzeri belgeler** (içeride BSON — binary JSON) olarak saklar.

| İlişkisel | MongoDB |
|---|---|
| Database | Database |
| Tablo | **Collection** (koleksiyon) |
| Satır | **Document** (belge) |
| Sütun | Alan (field) |
| Primary key | `_id` |
| JOIN | Gömme (embedding) veya `$lookup` |

## Hangi problemi çözer?
- **Esnek şema:** Aynı koleksiyondaki belgelerin alanları farklı olabilir; yeni alan eklemek `ALTER TABLE` gerektirmez.
- **İç içe veri:** Müşteri ve adresleri, telefonları, tercihleri tek belgede durur; okumak için join gerekmez.
- **Geliştirici uyumu:** Uygulamadaki nesne/JSON yapısı veritabanına doğrudan yansır.
- **Yatay ölçekleme:** Sharding ile veri birden çok sunucuya bölünebilir.

## Nasıl çalışır?

**Belge örneği**
```json
{
  "_id": ObjectId("670b..."),
  "musteri_no": 123,
  "unvan": "Örnek A.Ş.",
  "segment": "KOBİ",
  "iletisim": { "il": "İstanbul", "telefonlar": ["0212...", "0532..."] },
  "urunler": [
    { "tur": "kredi", "tutar": 250000, "acilis": ISODate("2026-03-01") },
    { "tur": "pos", "terminal_sayisi": 3 }
  ],
  "guncelleme": ISODate("2026-10-13T09:00:00Z")
}
```

**Modelleme kararı: gömmek mi, referans mı?**
| Gömme (embed) | Referans (reference) |
|---|---|
| Birlikte okunan veri ("müşteri ve adresleri") | Bağımsız büyüyen veri ("müşteri ve milyonlarca işlemi") |
| 1'e az ilişki | 1'e çok veya çoka çok ilişki |
| Tek sorguda hızlı okuma | Tekrarı önler |

Belge boyutu üst sınırı 16 MB'dır; sınırsız büyüyen diziler gömülmez.

**Mimari**
- **Replica set:** Bir birincil (primary) ve birden çok ikincil (secondary) kopya; birincil çökerse otomatik seçim (failover).
- **Sharding:** Koleksiyon bir **shard key**'e göre sunuculara bölünür.
- **Read/write concern:** Yazmanın kaç kopyaya ulaştığında onaylanacağı (`w: "majority"`) ayarlanabilir; tutarlılık ile hız arasında seçim.

## Mini örnek
`mongosh` kabuğunda:
```javascript
use crm

db.musteriler.insertOne({ musteri_no: 123, unvan: "Örnek A.Ş.", segment: "KOBİ" })

// Sorgu: İstanbul'daki KOBİ'ler, sadece iki alan
db.musteriler.find(
  { segment: "KOBİ", "iletisim.il": "İstanbul" },
  { unvan: 1, musteri_no: 1, _id: 0 }
).sort({ unvan: 1 }).limit(10)

// Güncelleme: diziye ürün ekle
db.musteriler.updateOne(
  { musteri_no: 123 },
  { $push: { urunler: { tur: "kart", limit: 50000 } }, $set: { guncelleme: new Date() } }
)

// Index
db.musteriler.createIndex({ segment: 1, "iletisim.il": 1 })

// Aggregation pipeline: segment bazında toplam kredi
db.musteriler.aggregate([
  { $unwind: "$urunler" },
  { $match: { "urunler.tur": "kredi" } },
  { $group: { _id: "$segment", toplam: { $sum: "$urunler.tutar" }, adet: { $sum: 1 } } },
  { $sort: { toplam: -1 } }
])
```

Python'dan (pymongo) ve pandas'a aktarma:
```python
from pymongo import MongoClient
import pandas as pd

client = MongoClient("mongodb://kullanici:sifre@mongo.sirket.local:27017/")
kol = client["crm"]["musteriler"]

belgeler = list(kol.find({"segment": "KOBİ"}, {"_id": 0}))
df = pd.json_normalize(belgeler)          # iç içe alanlar: iletisim.il gibi sütunlar olur
```

## Nerede kullanılır?
- Müşteri 360 profilleri, kampanya ve tercih verisi
- Esnek yapılı başvuru formları ve onay akışı belgeleri
- Uygulama logları, olay kayıtları, API cevaplarının saklanması
- İçerik yönetimi, ürün katalogları

## Benzerleri ve farkları
- **PostgreSQL `jsonb`:** İlişkisel veritabanı içinde belge saklama ve sorgulama. Çoğu zaman ayrı bir MongoDB'ye gerek bırakmaz; ilişkisel ve belge verisi aynı yerde, ACID ile durur.
- **Couchbase, Amazon DocumentDB, Azure Cosmos DB:** Diğer belge veritabanları (DocumentDB ve Cosmos DB MongoDB API uyumluluğu sunar).
- **Analitik için:** MongoDB'deki veri genellikle lakehouse veya veri ambarına aktarılıp orada analiz edilir; aggregation pipeline operasyonel raporlar için uygundur.
- **Lisans notu:** MongoDB Community Server, SSPL lisanslıdır; kurumsal kullanımda lisans koşulları kontrol edilmelidir.

**Dikkat:** "Şema yok" şemasız tasarım demek değildir. Uygulamada şema disiplini (JSON Schema doğrulaması, Pydantic modelleri) olmazsa koleksiyonlar zamanla tutarsız hâle gelir.

## İlgili sayfalar
- [9.5 NoSQL türleri](05-nosql-turleri.md)
- [9.11 Schema-on-write ve schema-on-read](11-schema-on-write-read.md)
- [16.4 BSON](../16-formatlar/04-bson.md)
