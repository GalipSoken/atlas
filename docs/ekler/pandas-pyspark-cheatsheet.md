# pandas / PySpark Cheat Sheet

Ayrıntılar: [10.2 pandas](../10-veri-isleme/02-pandas.md), [10.3 Spark](../10-veri-isleme/03-spark.md), [10.4 karşılaştırma](../10-veri-isleme/04-karsilastirma.md)

```python
import pandas as pd
from pyspark.sql import SparkSession, functions as F, Window
spark = SparkSession.builder.appName("ornek").getOrCreate()
```

## Okuma ve yazma
| İş | pandas | PySpark |
|---|---|---|
| CSV | `pd.read_csv("f.csv", sep=";", decimal=",")` | `spark.read.option("header", True).option("sep", ";").csv("f.csv")` |
| Parquet | `pd.read_parquet("klasor/")` | `spark.read.parquet("s3a://.../")` |
| Excel | `pd.read_excel("f.xlsx", sheet_name="S", skiprows=3)` | (pandas ile okuyup `spark.createDataFrame`) |
| JSON Lines | `pd.read_json("f.jsonl", lines=True)` | `spark.read.json("f.jsonl")` |
| SQL | `pd.read_sql(sorgu, engine)` | `spark.read.jdbc(url, "tablo", properties=...)` |
| Tablo (katalog) | — | `spark.table("silver.kredi")` |
| Parquet yaz | `df.to_parquet("f.parquet", index=False)` | `df.write.mode("overwrite").parquet(yol)` |
| Bölümlü yaz | `df.to_parquet("k/", partition_cols=["donem"])` | `df.write.partitionBy("donem").parquet(yol)` |
| Delta yaz | — | `df.write.format("delta").mode("append").saveAsTable("gold.t")` |

## Keşif
| İş | pandas | PySpark |
|---|---|---|
| İlk satırlar | `df.head()` | `df.show(5)` / `df.limit(5).toPandas()` |
| Şema | `df.info()`, `df.dtypes` | `df.printSchema()` |
| Satır sayısı | `len(df)` | `df.count()` |
| Özet | `df.describe()` | `df.describe().show()` |
| Değer sayımı | `df["s"].value_counts()` | `df.groupBy("s").count().orderBy(F.desc("count"))` |

## Seçme ve filtreleme
| İş | pandas | PySpark |
|---|---|---|
| Sütun seç | `df[["a", "b"]]` | `df.select("a", "b")` |
| Filtre | `df[(df.a > 5) & (df.b == "x")]` / `df.query("a > 5")` | `df.filter((F.col("a") > 5) & (F.col("b") == "x"))` |
| İçinde | `df[df.s.isin(["KOBİ", "Ticari"])]` | `df.filter(F.col("s").isin("KOBİ", "Ticari"))` |
| Boş olmayan | `df[df.a.notna()]` | `df.filter(F.col("a").isNotNull())` |
| Metin içerir | `df[df.ad.str.contains("bank", case=False)]` | `df.filter(F.lower("ad").contains("bank"))` |

## Sütun işlemleri
| İş | pandas | PySpark |
|---|---|---|
| Yeni sütun | `df["c"] = df.a * 2` | `df = df.withColumn("c", F.col("a") * 2)` |
| Koşullu | `np.where(df.a > 0, "+", "-")` | `F.when(F.col("a") > 0, "+").otherwise("-")` |
| Yeniden adlandır | `df.rename(columns={"a": "x"})` | `df.withColumnRenamed("a", "x")` |
| Tip dönüştür | `df.a.astype("float64")`, `pd.to_numeric` | `F.col("a").cast("double")` |
| Tarih | `pd.to_datetime(df.d)`, `df.d.dt.to_period("M")` | `F.to_date("d")`, `F.date_trunc("month", "d")` |
| Boş doldur | `df.fillna({"a": 0})` | `df.fillna({"a": 0})` |
| Sil | `df.drop(columns=["a"])` | `df.drop("a")` |
| Tekrarları kaldır | `df.drop_duplicates(["k"])` | `df.dropDuplicates(["k"])` |

## Gruplama ve birleştirme
| İş | pandas | PySpark |
|---|---|---|
| Gruplama | `df.groupby("s", as_index=False).agg(top=("t", "sum"), adet=("t", "size"))` | `df.groupBy("s").agg(F.sum("t").alias("top"), F.count("*").alias("adet"))` |
| Join | `a.merge(b, on="id", how="left")` | `a.join(b, on="id", how="left")` |
| Küçük tabloyla join | — | `a.join(F.broadcast(b), "id")` |
| Alt alta ekle | `pd.concat([a, b])` | `a.unionByName(b)` |
| Pivot | `df.pivot_table(index="d", columns="s", values="t", aggfunc="sum")` | `df.groupBy("d").pivot("s").sum("t")` |
| Sıralama | `df.sort_values("t", ascending=False)` | `df.orderBy(F.desc("t"))` |

## Pencere (zaman serisi) işlemleri
```python
# pandas
df = df.sort_values(["segment", "donem"])
df["onceki"] = df.groupby("segment")["tutar"].shift(1)
df["yoy"]    = df.groupby("segment")["tutar"].pct_change(12)
df["kum"]    = df.groupby("segment")["tutar"].cumsum()
df["ort3"]   = df.groupby("segment")["tutar"].transform(lambda s: s.rolling(3).mean())

# PySpark
w = Window.partitionBy("segment").orderBy("donem")
df = (df.withColumn("onceki", F.lag("tutar", 1).over(w))
        .withColumn("yoy", F.col("tutar") / F.lag("tutar", 12).over(w) - 1)
        .withColumn("kum", F.sum("tutar").over(w.rowsBetween(Window.unboundedPreceding, 0)))
        .withColumn("ort3", F.avg("tutar").over(w.rowsBetween(-2, 0)))
        .withColumn("sira", F.row_number().over(Window.partitionBy("donem").orderBy(F.desc("tutar")))))
```

## SQL köprüsü
```python
df.createOrReplaceTempView("kredi")              # Spark: DataFrame → SQL
spark.sql("SELECT segment, SUM(tutar) FROM kredi GROUP BY segment").show()

import duckdb                                     # pandas DataFrame'ine SQL
duckdb.sql("SELECT segment, SUM(tutar) FROM df GROUP BY segment").df()
```

## pandas ↔ Spark
```python
pdf = sdf.limit(100_000).toPandas()               # sadece küçük sonuçlar!
sdf = spark.createDataFrame(pdf)
```

## Performans ipuçları
| pandas | PySpark |
|---|---|
| Sadece gereken sütunları oku (`usecols`, `columns`) | Erken `select` ve `filter` (iyileştirici aşağı iter) |
| Tekrarlı metinlere `category` | Python UDF yerine `F.*` fonksiyonları |
| `iterrows` yerine vektörel işlem | `collect()` / `toPandas()` sadece küçük veride |
| Büyük CSV yerine Parquet | Küçük dosya sorunu: yazmadan önce `coalesce(n)` |
| RAM'e sığmıyorsa Polars/DuckDB | `df.explain()` ve Spark UI ile plan/aşama incele |
