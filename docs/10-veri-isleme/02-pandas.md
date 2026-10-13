# 10.2 Pandas: Tek Makine, Bellekte İşlem

## Nedir?
**pandas**, Python'un en yaygın veri analizi kütüphanesidir. Veriyi **DataFrame** adı verilen, Excel tablosuna benzeyen bir yapıda **bilgisayarın belleğinde (RAM)** tutar ve işler.

## Hangi problemi çözer?
- Excel'in sınırlarını (1 milyon satır, elle tekrar eden işler, hata yapmaya açık formüller) aşar.
- Farklı kaynaklardan (CSV, Excel, SQL, JSON, Parquet, API) veriyi okuyup birleştirmeyi tek dilde yapar.
- Analizi **tekrarlanabilir kod** hâline getirir.
- Görselleştirme (matplotlib, plotly), istatistik ve makine öğrenmesi (scikit-learn) kütüphaneleriyle doğrudan çalışır.

## Nasıl çalışır?
- **Series:** Tek sütun (etiketli dizi).
- **DataFrame:** Sütunlardan oluşan tablo; her sütunun bir tipi (`int64`, `float64`, `object`, `datetime64`, `category`) vardır.
- **Index:** Satır etiketleri.
- **Vektörel işlem:** İşlemler satır satır döngüyle değil, tüm sütun üzerinde birden yapılır (altında NumPy veya Arrow). `for` döngüsü pandas'ta genellikle yanlış yoldur.
- **Eager (anında) çalışma:** Her komut hemen çalışır ve sonucu bellekte üretir.

**Bellek sınırı:** Bütün veri RAM'e sığmalıdır. Pratik kural: Dosya boyutunun 5–10 katı bellek gerekebilir (ara kopyalar nedeniyle). 16 GB RAM'li bir makinede birkaç GB'lık veri rahatça işlenir; onlarca GB zorlanır.

## Mini örnek
```python
import pandas as pd

# Okuma
df = pd.read_csv("kredi_bakiye.csv", parse_dates=["donem"], dtype={"segment": "category"})
# pd.read_excel("bddk.xlsx", sheet_name="Krediler", skiprows=3)
# pd.read_sql("SELECT ...", engine)
# pd.read_parquet("s3://lake/silver/kredi/")

# Keşif
df.head()
df.info()                 # sütun tipleri, boş değerler, bellek
df.describe()             # özet istatistikler
df["segment"].value_counts()

# Temizleme
df = df.dropna(subset=["bakiye"])
df["segment"] = df["segment"].str.strip().str.upper()
df = df.drop_duplicates(subset=["donem", "segment", "musteri_id"])

# Filtre, yeni sütun
buyuk = df[(df["bakiye"] > 1_000_000) & (df["segment"] == "KOBİ")]
df["bakiye_milyon"] = df["bakiye"] / 1e6

# Gruplama: aylık segment toplamı
aylik = (df.groupby(["donem", "segment"], as_index=False, observed=True)["bakiye"].sum()
           .sort_values(["segment", "donem"]))

# Zaman serisi metrikleri (SQL'deki LAG karşılığı: shift)
aylik["mom_yuzde"] = aylik.groupby("segment", observed=True)["bakiye"].pct_change(1) * 100
aylik["yoy_yuzde"] = aylik.groupby("segment", observed=True)["bakiye"].pct_change(12) * 100
aylik["yil"] = aylik["donem"].dt.year
aylik["ytd"] = aylik.groupby(["segment", "yil"], observed=True)["bakiye"].cumsum()

# Birleştirme (SQL JOIN karşılığı)
musteri = pd.read_csv("musteri.csv")
birlesik = df.merge(musteri, on="musteri_id", how="left", validate="many_to_one")

# Pivot (Excel pivot tablosu)
pivot = aylik.pivot_table(index="donem", columns="segment", values="bakiye", aggfunc="sum")

# Yazma
aylik.to_parquet("kredi_aylik.parquet", index=False)
aylik.to_excel("kredi_aylik.xlsx", index=False)
# aylik.to_sql("kredi_aylik", engine, if_exists="replace", index=False)
```

Bellek tasarrufu ipuçları:
```python
df = pd.read_csv("buyuk.csv", usecols=["donem", "segment", "bakiye"])   # sadece gereken sütunlar
df["segment"] = df["segment"].astype("category")                        # tekrar eden metin
df["bakiye"] = pd.to_numeric(df["bakiye"], downcast="float")
for parca in pd.read_csv("cok_buyuk.csv", chunksize=1_000_000):          # parça parça işle
    ...
df = pd.read_parquet("veri.parquet", dtype_backend="pyarrow")           # Arrow tabanlı tipler
```

## Nerede kullanılır?
- Keşifsel veri analizi (Jupyter notebook)
- Küçük–orta hacimli ETL'ler (BDDK, TÜİK, HMB Excel dosyalarını okuyup veritabanına yazmak)
- ML modelleri için veri hazırlama ve feature engineering
- Streamlit dashboard'larının veri katmanı

## Benzerleri ve farkları
| | pandas | Polars | Spark |
|---|---|---|---|
| Çalışma | Tek çekirdek ağırlıklı, eager | Çok çekirdekli, lazy seçeneği | Dağıtık, lazy |
| Ölçek | RAM'e sığan veri | RAM'e sığan (ve streaming ile biraz fazlası) | Cluster |
| Ekosistem | En geniş | Hızla büyüyor | Büyük veri |

- **Excel:** Elle analiz ve sunum için; tekrar eden ve büyük işlerde pandas.
- **SQL:** Veri veritabanındaysa önce SQL'de küçültüp sonra pandas'a almak en verimlisi ([10.1](01-sql.md)).
- Ayrıntılı karşılaştırma: [10.4](04-karsilastirma.md), daha hızlı alternatifler: [10.5](05-polars-duckdb.md).

**Sık hatalar:**
- `SettingWithCopyWarning`: Filtrelenmiş DataFrame'i değiştirirken `.copy()` veya `.loc` kullanın.
- `merge` sonrası satır sayısının beklenmedik artması: anahtar tekrarlıdır; `validate=` parametresiyle yakalayın.
- Türkçe Excel'lerde sayıların metin gelmesi (`"1.250.000,50"`): `decimal=","`, `thousands="."` parametreleri.
- `iterrows()` ile döngü: Çok yavaştır; vektörel işlem kullanın.

## İlgili sayfalar
- [10.3 Spark](03-spark.md)
- [10.4 Karşılaştırma](04-karsilastirma.md)
- [10.5 Polars ve DuckDB](05-polars-duckdb.md)
- [16.2 CSV](../16-formatlar/02-csv.md)
