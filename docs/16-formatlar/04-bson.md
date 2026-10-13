# 16.4 BSON

## Nedir?
**BSON (Binary JSON)**, JSON benzeri belgelerin **ikili (binary)** olarak kodlanmış hâlidir. MongoDB tarafından geliştirilmiştir ve MongoDB belgeleri hem diskte hem ağ üzerinde BSON olarak saklanır ve taşınır ([9.6](../09-veri-depolama/06-mongodb.md)).

Kullanıcı genellikle BSON'u doğrudan görmez: Uygulama JSON benzeri nesnelerle çalışır, sürücü (driver) BSON'a çevirir.

## Hangi problemi çözer?
JSON'un veritabanı için yetersiz kaldığı noktalar:
| JSON sorunu | BSON çözümü |
|---|---|
| Tarih tipi yok | `Date` tipi (UTC milisaniye) |
| Sayılar tek tip (number), para tutarında hassasiyet kaybı | `int32`, `int64`, `double`, **`Decimal128`** (yüksek hassasiyetli ondalık) |
| İkili veri (dosya, resim) taşınamaz | `Binary` tipi |
| Benzersiz kimlik tipi yok | `ObjectId` (12 bayt; zaman damgası içerir) |
| Metni baştan sona ayrıştırmak yavaş | Alan uzunlukları önceden yazılı; belge içinde hızlı gezinme |

## Nasıl çalışır?
- Belge; **toplam uzunluk + (tip baytı, alan adı, değer) dizisi + sonlandırıcı** şeklinde kodlanır.
- Her değerin önünde tip bilgisi bulunur; okuyucu hangi tipte değer geldiğini bilir.
- Uzunluk bilgisi sayesinde istenmeyen alanlar atlanarak hızla ilerlenir (traversal).
- Bu ek bilgiler yüzünden **küçük belgelerde BSON, JSON'dan büyük** olabilir; amacı küçüklük değil, hız ve zengin tiplerdir.

**BSON tipleri (sık kullanılanlar)**
| BSON tipi | Açıklama | JSON karşılığı (Extended JSON) |
|---|---|---|
| `ObjectId` | 12 baytlık benzersiz kimlik | `{"$oid": "670b1f..."}` |
| `Date` | UTC zaman | `{"$date": "2026-10-13T06:30:00Z"}` |
| `Int32` / `Int64` | Tamsayılar | `{"$numberLong": "123"}` |
| `Double` | Ondalıklı (float) | sayı |
| `Decimal128` | Hassas ondalık (para) | `{"$numberDecimal": "1250000.50"}` |
| `Binary` | İkili veri | `{"$binary": {...}}` |
| `String`, `Boolean`, `Null`, `Array`, `Document` | JSON'daki karşılıkları | aynı |
| `Timestamp` | MongoDB iç kullanımı (replikasyon) | |

**Extended JSON:** BSON'a özgü tiplerin JSON metninde kaybolmaması için kullanılan gösterim (`$oid`, `$date`, `$numberDecimal`). `mongoexport` ve MongoDB araçları bu biçimi kullanır.

**Sınır:** Tek bir BSON belgesi en fazla **16 MB** olabilir (daha büyük dosyalar için MongoDB'nin GridFS mekanizması).

## Mini örnek
**Python (pymongo ile gelen `bson` modülü)**
```python
from datetime import datetime, timezone
from decimal import Decimal
from bson import ObjectId, Decimal128, encode, decode
from bson.json_util import dumps, loads

belge = {
    "_id": ObjectId(),
    "musteri_no": 123,
    "tutar": Decimal128(Decimal("1250000.50")),      # para: hassas ondalık
    "acilis": datetime(2026, 10, 13, 9, 30, tzinfo=timezone.utc),
}

ikili = encode(belge)                    # BSON bayt dizisi
print(len(ikili), "bayt")
geri = decode(ikili)

print(dumps(belge))                      # Extended JSON (tipler korunur)
# {"_id": {"$oid": "..."}, "musteri_no": 123, "tutar": {"$numberDecimal": "1250000.50"}, "acilis": {"$date": "..."}}

print(belge["_id"].generation_time)      # ObjectId'nin içindeki oluşturulma zamanı
```

**MongoDB araçları**
```bash
mongodump  --uri "mongodb://.../crm" --collection musteriler --out yedek/   # BSON dosyaları (.bson)
mongorestore --uri "mongodb://.../crm_test" yedek/crm
bsondump yedek/crm/musteriler.bson | head                                  # BSON'u JSON olarak incele
mongoexport --uri "mongodb://.../crm" -c musteriler --jsonFormat=canonical -o musteriler.json
```

**Analize aktarmak:** BSON tipleri pandas'a geçerken dönüştürülmelidir.
```python
import pandas as pd
df = pd.json_normalize(list(kol.find({}, {"_id": 0})))
df["tutar"] = df["tutar"].map(lambda d: d.to_decimal() if d is not None else None)   # Decimal128 → Decimal
```

## Nerede kullanılır?
- MongoDB'nin depolama ve ağ protokolü formatı
- MongoDB yedekleri (`mongodump`)
- Bazı mesajlaşma ve gömülü sistemlerde JSON benzeri ikili veri değişimi

## Benzerleri ve farkları
| Format | Fark |
|---|---|
| **JSON** | Metin, evrensel, daha az tip ([16.3](03-json.md)) |
| **MessagePack** | Daha kompakt ikili JSON; tip zenginliği BSON'dan az |
| **CBOR** | IETF standardı ikili JSON benzeri format (IoT, güvenlik protokolleri) |
| **PostgreSQL `jsonb`** | Benzer fikir: JSON'un veritabanı içinde ayrıştırılmış ikili saklanması; ama tip sistemi JSON ile aynı |
| **Protobuf / Avro** | Şemalı ikili formatlar; BSON şemasızdır |

## İlgili sayfalar
- [9.6 MongoDB](../09-veri-depolama/06-mongodb.md)
- [16.3 JSON](03-json.md)
