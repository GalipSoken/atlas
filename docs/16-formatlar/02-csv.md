# 16.2 CSV

## Nedir?
**CSV (Comma-Separated Values — virgülle ayrılmış değerler)**, tablo verisini düz metin olarak saklayan en basit formattır. Her satır bir kayıt, her alan bir ayırıcıyla (genellikle virgül) ayrılır; ilk satır çoğunlukla sütun başlıklarıdır.

```csv
donem,kalem,tutar
2026-09,kobi,1250000.50
2026-09,konut,980000.00
```

Resmî bir tanımı (RFC 4180) olsa da pratikte birçok "lehçesi" vardır: noktalı virgülle ayrılan (Türkçe Excel), sekmeyle ayrılan (**TSV**), farklı tırnak ve satır sonu kuralları.

## Hangi problemi çözer?
- Her araç okur ve yazar: Excel, veritabanları, Python, R, BI araçları, komut satırı.
- İnsan tarafından okunabilir; hızlıca açılıp kontrol edilebilir.
- Kurumlar ve sistemler arası en düşük ortak payda.

## Nasıl çalışır?

**Kurallar (RFC 4180)**
| Kural | Örnek |
|---|---|
| Alanlar ayırıcıyla ayrılır | `a,b,c` |
| Ayırıcı, tırnak veya satır sonu içeren alan çift tırnak içine alınır | `"Örnek, A.Ş.",kobi,100` |
| Alan içindeki çift tırnak ikiyle kaçışlanır | `"12"" monitör"` |
| Satır sonu CRLF (pratikte LF de yaygın) | |

**CSV'nin zayıf yanları ve tuzaklar**
| Sorun | Örnek | Çözüm |
|---|---|---|
| **Tip bilgisi yok** | Her şey metin; `00123` sayıya çevrilince `123` olur | Okurken tipleri açıkça ver (`dtype`) |
| **Ayırıcı belirsizliği** | Türkçe Excel `;` ile kaydeder (çünkü ondalık ayırıcı `,`) | `sep=";"` |
| **Sayı biçimi** | `1.250.000,50` (TR) vs `1,250,000.50` (EN) | `decimal=","`, `thousands="."` |
| **Karakter kodlaması** | Türkçe karakterler `Ä±` gibi bozulur | UTF-8 kullan; eski dosyalarda `cp1254` / `ISO-8859-9` |
| **Excel ve UTF-8** | Excel, BOM'suz UTF-8 CSV'yi yanlış kodlamayla açabilir | Yazarken `utf-8-sig` (BOM ekler) |
| **Tarih biçimi** | `09/10/2026` → 9 Ekim mi, 10 Eylül mü? | ISO 8601: `2026-10-09` |
| **Excel'in otomatik dönüştürmesi** | Uzun numaralar (TCKN, hesap no) bilimsel gösterime döner: `1,23457E+10`; bazı kodlar tarihe döner | Kimlik alanlarını metin olarak işaretle, mümkünse Excel'de elle açıp kaydetme |
| **Boş değer** | Boş, `NULL`, `NA`, `-` karışık | Okurken `na_values` ile tanımla |
| **Şema yok** | Sütun sırası/adı değişince süreç kırılır | Okuma sonrası şema kontrolü ([11.5](../11-veri-akisi/05-veri-kalitesi.md)) |
| **Büyük boyut** | GB'larca, yavaş | Parquet'e dönüştür ([16.5](05-parquet.md)) |

## Mini örnek
**pandas ile güvenli okuma**
```python
import pandas as pd

df = pd.read_csv(
    "bddk_kredi.csv",
    sep=";",                        # Türkçe Excel çıktısı
    decimal=",", thousands=".",     # 1.250.000,50
    encoding="utf-8-sig",           # BOM'lu UTF-8 (değilse "cp1254" dene)
    dtype={"musteri_no": str, "sube_kodu": str},     # baştaki sıfırlar korunur
    parse_dates=["islem_tarihi"], dayfirst=True,
    na_values=["", "NULL", "-"],
)
```

**Excel'in doğru açacağı CSV yazmak**
```python
df.to_csv("rapor.csv", sep=";", decimal=",", index=False, encoding="utf-8-sig")
```

**Python standart kütüphanesi (pandas'sız)**
```python
import csv
with open("kredi.csv", newline="", encoding="utf-8") as f:
    for satir in csv.DictReader(f):
        print(satir["kalem"], float(satir["tutar"]))
```

**Komut satırı**
```bash
head -3 kredi.csv
wc -l kredi.csv                                  # satır sayısı
cut -d';' -f2 kredi.csv | sort | uniq -c         # 2. sütundaki değer dağılımı
file -i kredi.csv                                # karakter kodlaması tahmini
iconv -f cp1254 -t utf-8 eski.csv > yeni.csv     # kodlama dönüştürme
```

**DuckDB ile doğrudan SQL ve otomatik tip tespiti**
```sql
SELECT kalem, SUM(tutar) FROM read_csv('kredi_*.csv', delim=';', decimal_separator=',') GROUP BY kalem;
COPY (SELECT * FROM read_csv('kredi.csv')) TO 'kredi.parquet' (FORMAT parquet);
```

**PostgreSQL'e hızlı yükleme**
```sql
\copy silver.bddk_kredi(donem, kalem, tutar) FROM 'kredi.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',')
```

## Nerede kullanılır?
- Excel kullanıcılarıyla ve dış kurumlarla veri alışverişi
- Veritabanı dışa/içe aktarım (`COPY`, SQL*Loader)
- Küçük referans tabloları (dbt `seeds`, [11.4](../11-veri-akisi/04-dbt.md))
- Test verisi ve örnek dosyalar

## Benzerleri ve farkları
- **TSV:** Sekmeyle ayrılmış; metin içinde virgül çok olduğunda daha az tırnak sorunu.
- **Sabit genişlikli (fixed-width) dosyalar:** Ana bilgisayar (mainframe) ve eski bankacılık sistemlerinde yaygın; her alan belirli karakter aralığında (`pd.read_fwf`).
- **Parquet:** Büyük ve tekrar okunan veride CSV yerine tercih edilmelidir; tipleri korur, sıkıştırır, hızlıdır.
- **Excel:** Biçimlendirme ve çoklu sayfa sunar, ama makine okuması için daha kırılgandır ([16.9](09-digerleri.md)).

## İlgili sayfalar
- [16.1 Genel bakış](01-genel-bakis.md)
- [16.5 Parquet](05-parquet.md)
- [10.2 pandas](../10-veri-isleme/02-pandas.md)
- [16.11 Karakter kodlaması](11-karakter-kodlamasi.md)
