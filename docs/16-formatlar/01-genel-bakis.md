# 16.1 Formatlara Genel Bakış: Metin / İkili, Satır / Sütun, Şemalı / Şemasız

## Nedir?
**Dosya formatı**, verinin bir dosyada (veya ağ üzerinden gönderilirken) hangi kurallarla yazıldığıdır. Bu kitapta karşılaşılan formatlar iki ana amaca hizmet eder:
- **Veri formatları:** Tablo veya kayıt verisini saklamak ve taşımak: CSV, JSON, BSON, Parquet, Avro, ORC, Excel.
- **Yapılandırma (config) formatları:** Ayarları ve tanımları yazmak: YAML, TOML, `.env`, JSON, INI.

## Hangi problemi çözer?
Yanlış format seçimi; yavaş sorgular, dev dosyalar, kaybolan veri tipleri (tarih metne dönüşür, baştaki sıfırlar silinir), Türkçe karakter bozulmaları ve okunamaz konfigürasyonlar demektir. Formatların özelliklerini bilmek doğru yerde doğru seçimi yapmayı sağlar.

## Nasıl çalışır?
Formatları dört soruyla sınıflandırmak mümkündür:

**1. Metin mi, ikili (binary) mi?**
| | Metin | İkili |
|---|---|---|
| Örnek | CSV, JSON, YAML, TOML, XML | Parquet, BSON, Avro, ORC, Excel (xlsx, sıkıştırılmış XML) |
| İnsan okuyabilir mi? | Evet (`cat`, editör) | Hayır, özel araç gerekir |
| Boyut ve hız | Büyük, yavaş ayrıştırma | Küçük, hızlı |
| Git'te fark (diff) | Anlamlı | Anlamsız |

**2. Satır mı, sütun mu? (veri formatları için)**
```
Satır bazlı (CSV, JSON Lines, Avro):     [1,Ali,KOBİ,100] [2,Ayşe,Bireysel,250] ...
Sütun bazlı (Parquet, ORC):              [1,2,...] [Ali,Ayşe,...] [KOBİ,Bireysel,...] [100,250,...]
```
| | Satır bazlı | Sütun bazlı |
|---|---|---|
| Kayıt ekleme / tek kayıt okuma | Hızlı | Yavaş |
| Analitik ("toplam bakiye") | Tüm satırlar okunur | Sadece gereken sütun okunur |
| Sıkıştırma | Orta | Çok iyi (aynı tip yan yana) |
| Tipik kullanım | Akış, mesajlaşma, işlem kaydı | Analitik, lakehouse ([9.1](../09-veri-depolama/01-oltp-olap.md)) |

**3. Şemalı mı, şemasız mı?**
| | Şema dosyada / zorunlu | Şema yok, yorumlanır |
|---|---|---|
| Örnek | Parquet, Avro, ORC (şema dosyanın içinde), Protobuf (ayrı tanım) | CSV, JSON, YAML |
| Veri tipleri | Korunur (tarih, ondalık, tamsayı) | Tahmin edilir veya metindir |
| Esneklik | Daha az | Daha çok |
| Bağlantı | Schema-on-write'a yakın | Schema-on-read ([9.11](../09-veri-depolama/11-schema-on-write-read.md)) |

**4. Hiyerarşik mi, düz mü?**
- **Düz (tablo):** CSV, Excel sayfası.
- **Hiyerarşik (iç içe):** JSON, BSON, YAML, TOML, XML, Avro, Parquet (iç içe tipleri destekler).

**Karşılaştırma tablosu**
| Format | Tür | Yapı | Şema | Tipik kullanım | Bölüm |
|---|---|---|---|---|---|
| CSV | Metin | Satır, düz | Yok | Basit veri alışverişi, Excel dışa aktarım | [16.2](02-csv.md) |
| JSON | Metin | Hiyerarşik | Yok (JSON Schema ile isteğe bağlı) | API'ler, konfigürasyon, log | [16.3](03-json.md) |
| JSON Lines | Metin | Satır, her satır bir JSON | Yok | Log, akış, büyük JSON veri | [16.3](03-json.md) |
| BSON | İkili | Hiyerarşik | Yok | MongoDB'nin iç formatı | [16.4](04-bson.md) |
| Parquet | İkili | Sütun | Var | Analitik, lakehouse | [16.5](05-parquet.md) |
| YAML | Metin | Hiyerarşik | Yok | Kubernetes, CI/CD, Ansible, uygulama ayarları | [16.6](06-yaml.md) |
| TOML | Metin | Hiyerarşik (tablolar) | Yok | `pyproject.toml`, araç ayarları | [16.7](07-toml.md) |
| .env | Metin | Düz anahtar=değer | Yok | Ortam değişkenleri, gizli olmayan + gizli ayarlar | [16.8](08-env.md) |
| Avro | İkili | Satır | Var | Kafka mesajları, veri aktarımı | [16.9](09-digerleri.md) |
| ORC | İkili | Sütun | Var | Hive / Hadoop dünyası | [16.9](09-digerleri.md) |
| Arrow | İkili (bellek) | Sütun | Var | Bellekte araçlar arası kopyasız veri paylaşımı | [16.9](09-digerleri.md) |
| XML | Metin | Hiyerarşik | XSD ile | SOAP, eski entegrasyonlar, düzenleyici bildirimler | [16.9](09-digerleri.md) |
| Excel (xlsx) | İkili (zip+XML) | Sayfa/tablo | Yok | İş birimleriyle paylaşım, dış veri kaynakları | [16.9](09-digerleri.md) |

## Mini örnek
Aynı 3 satırlık veri, farklı formatlarda:
```csv
donem,kalem,tutar
2026-09,kobi,1250000.50
2026-09,konut,980000.00
```
```json
[{"donem": "2026-09", "kalem": "kobi", "tutar": 1250000.50},
 {"donem": "2026-09", "kalem": "konut", "tutar": 980000.00}]
```
```yaml
- donem: "2026-09"
  kalem: kobi
  tutar: 1250000.50
- donem: "2026-09"
  kalem: konut
  tutar: 980000.00
```

Python'da format dönüşümleri:
```python
import pandas as pd

df = pd.read_csv("kredi.csv", dtype={"donem": str})
df.to_json("kredi.json", orient="records", force_ascii=False, indent=2)
df.to_json("kredi.jsonl", orient="records", lines=True, force_ascii=False)
df.to_parquet("kredi.parquet", index=False)
df.to_excel("kredi.xlsx", index=False)
```

Hızlı inceleme araçları:
```bash
file veri.bin                       # dosya türü tahmini
head -5 kredi.csv                   # metin formatlar
jq . kredi.json | head              # JSON
yq . config.yaml                    # YAML (yq kuruluysa)
duckdb -c "DESCRIBE SELECT * FROM 'kredi.parquet'"    # Parquet şeması
```

## Nerede kullanılır?
| Durum | Önerilen format |
|---|---|
| Dış kurumla veya Excel kullanıcısıyla basit tablo paylaşımı | CSV (UTF-8) veya Excel |
| API isteği/cevabı | JSON |
| Büyük analitik veri, lakehouse | Parquet (+ Delta/Iceberg) |
| Log ve olay akışı dosyaları | JSON Lines |
| Kafka mesajları (şemalı) | Avro veya Protobuf (Schema Registry ile) |
| Kubernetes, CI/CD, Ansible | YAML |
| Python projesi tanımı | TOML (`pyproject.toml`) |
| Ortama özgü ayar ve şifre (geliştirmede) | `.env` |

## Benzerleri ve farkları
- **Sıkıştırma ayrı bir katmandır:** `kredi.csv.gz`, Parquet içinde Snappy/Zstd. Sıkıştırma boyutu küçültür; formatın yapısını değiştirmez.
- **Kodlama (encoding):** Metin formatlarında karakter kodlaması önemlidir. Standart **UTF-8**; eski Windows/Excel dosyalarında `cp1254` (Türkçe Windows) veya `ISO-8859-9` ile karşılaşılabilir.

## İlgili sayfalar
- [16.2 CSV](02-csv.md)
- [16.5 Parquet](05-parquet.md)
- [16.6 YAML](06-yaml.md)
