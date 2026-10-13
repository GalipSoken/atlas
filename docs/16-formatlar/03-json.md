# 16.3 JSON ve JSON Lines

## Nedir?
- **JSON (JavaScript Object Notation):** İnsan tarafından okunabilir, hiyerarşik (iç içe) veri formatı. Web API'lerinin, birçok konfigürasyon dosyasının ve NoSQL veritabanlarının ortak dili. Standart: RFC 8259.
- **JSON Lines (JSONL / NDJSON):** Her satırda **bağımsız bir JSON nesnesi** bulunan dosya formatı. Büyük veri, log ve akışlar için JSON'un satır bazlı hâli.

## Hangi problemi çözer?
- **JSON:** İç içe yapıları (müşteri → adresler → telefonlar) doğal biçimde temsil eder; neredeyse tüm diller doğrudan okur/yazar; API sözleşmelerinin standardıdır ([7.3](../07-api/03-rest-json.md)).
- **JSON Lines:** Tek büyük JSON dizisi (`[...]`) dosyanın tamamı belleğe alınmadan okunamaz ve sonuna kayıt eklemek zordur. JSONL'de dosya satır satır okunur, sonuna ekleme yapılır, bölünüp paralel işlenebilir.

## Nasıl çalışır?

**JSON veri tipleri**
| Tip | Örnek | Python |
|---|---|---|
| Nesne (object) | `{"ad": "Ali", "yas": 30}` | `dict` |
| Dizi (array) | `[1, 2, 3]` | `list` |
| Metin (string) | `"KOBİ"` (her zaman çift tırnak) | `str` |
| Sayı (number) | `42`, `1250000.50`, `1.2e6` | `int` / `float` |
| Mantıksal | `true`, `false` | `True`, `False` |
| Boş | `null` | `None` |

**Kurallar ve tuzaklar**
| Kural / tuzak | Açıklama |
|---|---|
| Anahtarlar çift tırnaklı metindir | `{ad: 1}` veya `{'ad': 1}` geçersiz |
| Yorum satırı yok | `// açıklama` geçersiz (JSONC/JSON5 lehçeleri hariç) |
| Sondaki virgül yok | `[1, 2,]` geçersiz |
| **Tarih tipi yok** | ISO 8601 metin kullanılır: `"2026-10-13T09:30:00+03:00"` |
| **Ondalık hassasiyet** | Sayılar çoğu dilde float'a çevrilir; para tutarlarında yuvarlama hatası. Hassas tutarlar metin (`"1250000.50"`) veya kuruş cinsinden tamsayı olarak taşınabilir |
| Büyük tamsayılar | JavaScript 2^53'ten büyük tamsayıları kaybeder; uzun ID'ler metin olarak taşınmalı |
| Karakter kodlaması | UTF-8; Python'da `ensure_ascii=False` ile Türkçe karakterler okunur kalır |

**JSON ile JSON Lines**
```json
[
  {"kart_no": "4543...1234", "tutar": 2500.0, "ts": "2026-10-13T14:05:00+03:00"},
  {"kart_no": "5412...9876", "tutar": 120.5,  "ts": "2026-10-13T14:05:02+03:00"}
]
```
```jsonl
{"kart_no": "4543...1234", "tutar": 2500.0, "ts": "2026-10-13T14:05:00+03:00"}
{"kart_no": "5412...9876", "tutar": 120.5, "ts": "2026-10-13T14:05:02+03:00"}
```

**JSON Schema:** JSON belgelerinin yapısını ve kurallarını tanımlayan standart; API girdilerini ve konfigürasyonları doğrulamak için kullanılır.
```json
{
  "type": "object",
  "required": ["musteri_id", "tutar"],
  "properties": {
    "musteri_id": {"type": "integer"},
    "tutar": {"type": "number", "exclusiveMinimum": 0},
    "segment": {"enum": ["Bireysel", "KOBİ", "Ticari"]}
  }
}
```

## Mini örnek
**Python standart kütüphanesi**
```python
import json

veri = {"musteri_id": 123, "segment": "KOBİ", "krediler": [{"tutar": 250000}]}
metin = json.dumps(veri, ensure_ascii=False, indent=2)     # Python → JSON metni
geri = json.loads(metin)                                    # JSON metni → Python

with open("musteri.json", "w", encoding="utf-8") as f:
    json.dump(veri, f, ensure_ascii=False, indent=2)

# JSON Lines: satır satır yaz ve oku (bellek dostu)
with open("islemler.jsonl", "a", encoding="utf-8") as f:
    f.write(json.dumps({"kart_no": "x", "tutar": 10}, ensure_ascii=False) + "\n")

with open("islemler.jsonl", encoding="utf-8") as f:
    for satir in f:
        olay = json.loads(satir)
```

**pandas ile iç içe JSON'u tabloya çevirmek**
```python
import pandas as pd

kayitlar = [
    {"musteri_id": 1, "iletisim": {"il": "İstanbul"}, "urunler": [{"tur": "kredi", "tutar": 100}]},
    {"musteri_id": 2, "iletisim": {"il": "Ankara"},   "urunler": [{"tur": "kart",  "tutar": 50}]},
]
duz = pd.json_normalize(kayitlar)                      # iletisim.il sütunu oluşur
urunler = pd.json_normalize(kayitlar, record_path="urunler", meta=["musteri_id"])

df = pd.read_json("islemler.jsonl", lines=True)        # JSON Lines okuma
df.to_json("cikti.jsonl", orient="records", lines=True, force_ascii=False)
```

**Komut satırında `jq`**
```bash
jq . musteri.json                                      # güzel yazdır
jq '.krediler[] | .tutar' musteri.json                 # alan seç
jq -c 'select(.tutar > 1000)' islemler.jsonl           # JSONL'de filtre
curl -s https://api.sirket.local/v1/kredi | jq '.[] | {id, tutar}'
```

**DuckDB ile JSON / JSONL üzerinde SQL**
```sql
SELECT kart_no, SUM(tutar) FROM read_json_auto('islemler.jsonl') GROUP BY kart_no;
```

**PostgreSQL `jsonb`**
```sql
SELECT veri->>'segment' AS segment, (veri->'krediler'->0->>'tutar')::numeric AS ilk_kredi
FROM musteri_belge
WHERE veri @> '{"segment": "KOBİ"}';
```

## Nerede kullanılır?
- **JSON:** REST API istek ve cevapları, uygulama ve araç konfigürasyonları (`package.json`, VS Code ayarları, Grafana panoları), MongoDB belgeleri (BSON olarak), PostgreSQL `jsonb` sütunları
- **JSON Lines:** Yapılandırılmış loglar ([14.1](../14-gozlemlenebilirlik/01-log-metrik-trace.md)), olay dökümleri, Kafka mesajlarının dosyaya aktarımı, bronze katmanda ham API verisi, LLM eğitim/değerlendirme veri setleri

## Benzerleri ve farkları
| Format | Fark |
|---|---|
| **YAML** | JSON'un üst kümesi sayılır; yorum ve daha okunaklı yazım, konfigürasyon için ([16.6](06-yaml.md)) |
| **BSON** | JSON'un ikili hâli, ek veri tipleriyle (tarih, ondalık); MongoDB ([16.4](04-bson.md)) |
| **XML** | Daha ağır, öznitelik ve şema (XSD) desteği; eski sistemler ([16.9](09-digerleri.md)) |
| **Parquet** | Büyük tablo verisi için çok daha verimli ([16.5](05-parquet.md)) |
| **Protobuf / Avro** | Şemalı, ikili, küçük ve hızlı; servisler arası ve Kafka ([7.6](../07-api/06-diger-yaklasimlar.md)) |

## İlgili sayfalar
- [7.3 REST ve JSON](../07-api/03-rest-json.md)
- [16.4 BSON](04-bson.md)
- [16.6 YAML](06-yaml.md)
