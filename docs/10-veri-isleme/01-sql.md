# 10.1 SQL: Veri Veritabanında İşlenir

## Nedir?
SQL ile veri işlemek, veriyi uygulamaya çekmeden **veritabanı motorunun içinde** filtrelemek, birleştirmek, toplamak ve dönüştürmektir. SQL **deklaratif** (bildirimsel) bir dildir: **ne** istediğinizi yazarsınız, **nasıl** yapılacağına veritabanının sorgu iyileştiricisi karar verir.

## Hangi problemi çözer?
- **Veri hareketi azalır:** 500 milyon satırı Python'a çekip toplamak yerine, veritabanı toplar ve 12 satırlık sonucu gönderir.
- **Motorun gücü kullanılır:** Index, bölümleme, paralel çalışma, sorgu iyileştirme ([9.4](../09-veri-depolama/04-performans.md)).
- **Ortak dil:** Analist, veri mühendisi, BI aracı ve veritabanı yöneticisi aynı dili konuşur.
- **Yetki ve güvenlik:** Veri veritabanından çıkmadan işlenir.

## Nasıl çalışır?

**Mantıksal çalışma sırası** (yazılış sırasından farklıdır):
```mermaid
flowchart LR
    A[FROM / JOIN] --> B[WHERE] --> C[GROUP BY] --> D[HAVING] --> E[SELECT] --> F[ORDER BY] --> G[LIMIT]
```
Bu yüzden `SELECT`'te verilen takma ad `WHERE`'de kullanılamaz, ama `ORDER BY`'da kullanılabilir.

**Analitikte en çok kullanılan yapılar**

| Yapı | Ne işe yarar? |
|---|---|
| `JOIN` (INNER, LEFT, FULL) | Tabloları birleştirir |
| `GROUP BY` + toplama (`SUM`, `COUNT`, `AVG`) | Özetler |
| `CASE WHEN` | Koşullu değer, segmentleme |
| **CTE** (`WITH ... AS`) | Sorguyu okunaklı adımlara böler |
| **Pencere fonksiyonları** (`OVER`) | Satırları kaybetmeden grup içi hesap: sıra, önceki dönem, kümülatif toplam |
| `COALESCE` | NULL yerine varsayılan değer |
| `UNION ALL` | Sonuçları alt alta ekler |

**Pencere fonksiyonları** analitik SQL'in en güçlü aracıdır:
| Fonksiyon | Örnek |
|---|---|
| `LAG(x, 1)` / `LEAD` | Önceki/sonraki dönemin değeri (MoM, YoY) |
| `ROW_NUMBER()`, `RANK()` | Sıralama, grup içinde ilk N |
| `SUM(x) OVER (... ROWS UNBOUNDED PRECEDING)` | Kümülatif toplam (YTD) |
| `AVG(x) OVER (... ROWS 2 PRECEDING)` | Hareketli ortalama |

## Mini örnek
Aylık kredi bakiyesinden MoM, YoY ve YTD metrikleri üretmek:
```sql
WITH aylik AS (                                  -- 1. adım: aylık toplam
    SELECT donem, segment, SUM(bakiye) AS bakiye
    FROM kredi_bakiye
    WHERE donem >= DATE '2024-01-01'
    GROUP BY donem, segment
),
metrik AS (                                      -- 2. adım: pencere fonksiyonları
    SELECT
        donem,
        segment,
        bakiye,
        LAG(bakiye, 1)  OVER (PARTITION BY segment ORDER BY donem) AS onceki_ay,
        LAG(bakiye, 12) OVER (PARTITION BY segment ORDER BY donem) AS gecen_yil,
        SUM(bakiye) OVER (
            PARTITION BY segment, EXTRACT(YEAR FROM donem)
            ORDER BY donem ROWS UNBOUNDED PRECEDING
        ) AS ytd_toplam
    FROM aylik
)
SELECT
    donem,
    segment,
    bakiye,
    ROUND(100.0 * (bakiye / NULLIF(onceki_ay, 0) - 1), 2) AS mom_yuzde,
    ROUND(100.0 * (bakiye / NULLIF(gecen_yil, 0) - 1), 2) AS yoy_yuzde,
    ytd_toplam
FROM metrik
ORDER BY segment, donem;
```

Her segmentte en büyük 3 krediyi bulmak:
```sql
SELECT *
FROM (
    SELECT k.*, ROW_NUMBER() OVER (PARTITION BY segment ORDER BY tutar DESC) AS sira
    FROM kredi k
) t
WHERE sira <= 3;
```

Python'dan SQL çalıştırıp sadece sonucu almak:
```python
import pandas as pd
from sqlalchemy import create_engine, text

engine = create_engine("postgresql+psycopg2://...")
sorgu = text("SELECT segment, SUM(bakiye) AS bakiye FROM kredi_bakiye WHERE donem = :d GROUP BY segment")
df = pd.read_sql(sorgu, engine, params={"d": "2026-09-01"})     # parametreli: güvenli
```

## Nerede kullanılır?
- Veri ambarı ve lakehouse üzerinde raporlama ve metrik hesaplama
- ETL/ELT dönüşümleri (özellikle ELT'de dönüşüm tamamen SQL'dir; dbt [11.4](../11-veri-akisi/04-dbt.md))
- BI araçlarının arka planda ürettiği sorgular
- Spark SQL, DuckDB, Trino gibi motorlarda dosyalar üzerinde de aynı SQL

## Benzerleri ve farkları
- **Pandas/Polars:** Veri belleğe alınır, Python ile işlenir; karmaşık mantık, ML hazırlığı ve görselleştirme için daha esnek ([10.2](02-pandas.md)).
- **Spark:** SQL'i (Spark SQL) ve DataFrame API'sini dağıtık olarak çalıştırır ([10.3](03-spark.md)).
- Karşılaştırma ve "hangisini ne zaman" için: [10.4](04-karsilastirma.md).

**İpucu:** "Önce SQL'de küçült, sonra Python'a al." Filtre ve toplamayı veritabanında yapıp sadece gereken satırları çekmek, en sık yapılan performans iyileştirmesidir.

## İlgili sayfalar
- [9.2 İlişkisel veritabanları ve SQL](../09-veri-depolama/02-iliskisel-sql-acid.md)
- [10.4 SQL, Pandas, Spark karşılaştırması](04-karsilastirma.md)
