# SQL / psql Cheat Sheet

Ayrıntılar: [9.2 SQL ve ACID](../09-veri-depolama/02-iliskisel-sql-acid.md), [10.1 SQL ile veri işleme](../10-veri-isleme/01-sql.md). Örnekler PostgreSQL söz dizimiyledir; Oracle farkları [9.3](../09-veri-depolama/03-postgres-oracle.md)'te.

## Sorgu iskeleti ve çalışma sırası
```sql
SELECT   kolonlar, toplama()       -- 5
FROM     tablo t                   -- 1
JOIN     diger d ON ...            -- 1
WHERE    satır filtresi            -- 2
GROUP BY gruplama                  -- 3
HAVING   grup filtresi             -- 4
ORDER BY sıralama                  -- 6
LIMIT    n;                        -- 7
```

## Temel işlemler
```sql
SELECT DISTINCT segment FROM musteri;
SELECT * FROM kredi WHERE tutar BETWEEN 1e5 AND 1e6 AND segment IN ('KOBİ', 'Ticari');
SELECT * FROM musteri WHERE unvan ILIKE '%bank%';            -- büyük/küçük harf duyarsız
SELECT * FROM kredi WHERE kapanis_tarihi IS NULL;
SELECT COALESCE(segment, 'Bilinmiyor') FROM musteri;
SELECT CASE WHEN tutar > 1e6 THEN 'büyük' ELSE 'küçük' END AS boyut FROM kredi;
```

## JOIN türleri
| JOIN | Sonuç |
|---|---|
| `INNER JOIN` | İki tarafta da eşleşenler |
| `LEFT JOIN` | Soldakilerin hepsi + eşleşen sağ (yoksa NULL) |
| `FULL JOIN` | İki taraf da tümüyle |
| `CROSS JOIN` | Kartezyen çarpım |
| `LEFT JOIN ... WHERE sag.id IS NULL` | Solda olup sağda olmayanlar (anti join) |

## Gruplama
```sql
SELECT segment, COUNT(*) adet, SUM(tutar) toplam, AVG(tutar) ort
FROM kredi
GROUP BY segment
HAVING SUM(tutar) > 1e7
ORDER BY toplam DESC;
```

## CTE ve pencere fonksiyonları
```sql
WITH aylik AS (
    SELECT date_trunc('month', tarih) AS ay, segment, SUM(tutar) AS tutar
    FROM islem GROUP BY 1, 2
)
SELECT ay, segment, tutar,
       LAG(tutar)  OVER w                                   AS onceki_ay,
       tutar / NULLIF(LAG(tutar, 12) OVER w, 0) - 1         AS yoy,
       SUM(tutar)  OVER (PARTITION BY segment, date_trunc('year', ay) ORDER BY ay) AS ytd,
       AVG(tutar)  OVER (w ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)              AS hareketli_ort_3,
       ROW_NUMBER() OVER (PARTITION BY ay ORDER BY tutar DESC)                    AS sira
FROM aylik
WINDOW w AS (PARTITION BY segment ORDER BY ay);
```

## Tarih işlemleri (PostgreSQL)
```sql
CURRENT_DATE, NOW()
date_trunc('month', tarih)                 -- ay başı
tarih + INTERVAL '3 months'
EXTRACT(YEAR FROM tarih)
to_char(tarih, 'YYYY-MM')
age(bitis, baslangic)
```

## Veri değiştirme
```sql
INSERT INTO t (a, b) VALUES (1, 'x');
INSERT INTO t SELECT ... ;
UPDATE t SET b = 'y' WHERE a = 1;
DELETE FROM t WHERE a = 1;
INSERT INTO t (k, v) VALUES (1, 'x') ON CONFLICT (k) DO UPDATE SET v = EXCLUDED.v;   -- upsert
BEGIN; ... COMMIT;   -- veya ROLLBACK;
```

## Tablo, index, yetki
```sql
CREATE TABLE t (id BIGSERIAL PRIMARY KEY, donem DATE NOT NULL, tutar NUMERIC(20,2) CHECK (tutar >= 0));
ALTER TABLE t ADD COLUMN segment TEXT;
CREATE INDEX idx_t_donem ON t (donem);
CREATE VIEW v AS SELECT ...;
CREATE MATERIALIZED VIEW mv AS SELECT ...;  REFRESH MATERIALIZED VIEW mv;
GRANT SELECT ON ALL TABLES IN SCHEMA gold TO rol_okuma;
EXPLAIN ANALYZE SELECT ...;                 -- sorgu planı ve gerçek süre
```

## psql komutları
| Komut | Açıklama |
|---|---|
| `psql -h host -U kullanici -d db` | Bağlan |
| `psql "postgresql://u@host:5432/db?sslmode=verify-full"` | URL ile bağlan |
| `\l` | Veritabanları |
| `\c db` | Veritabanı değiştir |
| `\dn` | Şemalar |
| `\dt gold.*` | Tablolar |
| `\d+ tablo` | Tablo yapısı |
| `\di`, `\dv`, `\df` | Index'ler, view'lar, fonksiyonlar |
| `\du` | Roller |
| `\x` | Geniş satırları dikey göster |
| `\timing` | Sorgu sürelerini göster |
| `\i dosya.sql` | SQL dosyası çalıştır |
| `\copy t FROM 'f.csv' CSV HEADER` | İstemci tarafında CSV yükle |
| `\copy (SELECT ...) TO 'f.csv' CSV HEADER` | Sorgu sonucunu CSV'ye |
| `\q` | Çık |

## Yönetim sorguları
```sql
SELECT pid, usename, state, now() - query_start AS sure, query
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY sure DESC;   -- çalışan sorgular

SELECT pg_cancel_backend(12345);                                     -- sorguyu iptal et

SELECT relname, pg_size_pretty(pg_total_relation_size(relid))
FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC LIMIT 10;   -- en büyük tablolar
```

## Oracle karşılıkları (kısa)
| PostgreSQL | Oracle |
|---|---|
| `LIMIT 10` | `FETCH FIRST 10 ROWS ONLY` |
| `COALESCE` | `NVL` / `COALESCE` |
| `NOW()`, `CURRENT_DATE` | `SYSDATE`, `TRUNC(SYSDATE)` |
| `ON CONFLICT ... DO UPDATE` | `MERGE INTO ...` |
| `date_trunc('month', d)` | `TRUNC(d, 'MM')` |
| `psql` | `sqlplus` / SQLcl |
