# 9.3 PostgreSQL ve Oracle Karşılaştırması

## Nedir?
İkisi de güçlü, olgun ilişkisel veritabanlarıdır:
- **Oracle Database:** Oracle şirketinin ticari veritabanı. Onlarca yıldır bankacılık ve büyük kurumların ana işlem ve veri ambarı sistemlerinde standarttır.
- **PostgreSQL ("Postgres"):** Açık kaynak, ücretsiz, standartlara çok bağlı ve eklentilerle genişletilebilir veritabanı. Son yıllarda en hızlı yaygınlaşan ilişkisel veritabanıdır.

## Hangi problemi çözer?
Bir projede "hangi veritabanını kullanalım?" sorusu maliyet, ölçek, mevcut altyapı, ekip yetkinliği ve kurum politikasına göre cevaplanır. İki aracın farklarını ve SQL lehçelerini bilmek, iki dünya arasında geçişi kolaylaştırır.

## Nasıl çalışır?

| Konu | Oracle | PostgreSQL |
|---|---|---|
| Lisans | Ticari, çekirdek/özellik bazlı ücretli | Açık kaynak (PostgreSQL lisansı), ücretsiz |
| Destek | Oracle | Topluluk + ticari firmalar (EDB, Crunchy, bulut sağlayıcılar) |
| Prosedürel dil | PL/SQL | PL/pgSQL (ayrıca Python, JS vb.) |
| Yüksek erişilebilirlik | RAC (aktif–aktif cluster), Data Guard | Streaming replication, Patroni, operator'lar |
| Bölümleme (partitioning) | Çok gelişmiş (ek lisansla) | Yerleşik (declarative partitioning) |
| Analitik | Exadata, In-Memory seçenekleri | Eklentiler: TimescaleDB, Citus; ayrıca DuckDB ile entegrasyon |
| Genişletilebilirlik | Sınırlı | Çok güçlü: PostGIS (coğrafi), pgvector (vektör), özel tipler |
| JSON | Destekli | `jsonb` ile çok güçlü |
| Bulut | Oracle Cloud, Autonomous DB, diğer bulutlarda | Her bulutta yönetilen hizmet (RDS, Cloud SQL, Azure, Supabase) |
| Bankalarda | Ana sistemler, veri ambarı | Yeni uygulamalar, analitik ve iç araçlar, maliyet düşürme projeleri |

**Kavram eşleştirmesi**

| Oracle | PostgreSQL | Not |
|---|---|---|
| Instance + Database | Cluster (sunucu) + Database | |
| Schema = kullanıcı | Schema ayrı nesne | Postgres'te kullanıcı ve şema bağımsızdır |
| `VARCHAR2`, `NUMBER` | `VARCHAR`/`TEXT`, `NUMERIC`/`INTEGER` | |
| `DATE` (saat dahil) | `DATE` (sadece gün), `TIMESTAMP` | Taşımada sık hata kaynağı |
| `SYSDATE` | `CURRENT_DATE`, `NOW()` | |
| `NVL(a, b)` | `COALESCE(a, b)` | `COALESCE` iki tarafta da var |
| `DECODE(...)` | `CASE WHEN ...` | |
| `ROWNUM <= 10` | `LIMIT 10` | Oracle 12c+: `FETCH FIRST 10 ROWS ONLY` (ikisinde de çalışır) |
| `SEQUENCE.NEXTVAL` | `nextval('seq')`, `SERIAL`, `IDENTITY` | |
| `MERGE` | `INSERT ... ON CONFLICT` (upsert), PG15+ `MERGE` | |
| `DUAL` tablosu | Gerekmez: `SELECT 1;` | |
| Boş metin `''` = `NULL` | `''` ≠ `NULL` | **Önemli fark** |
| `CONNECT BY` | `WITH RECURSIVE` | Hiyerarşik sorgular |

## Mini örnek
Aynı iş, iki lehçe:

```sql
-- Oracle: son 3 ayın en büyük 10 kredisi
SELECT musteri_id, tutar, NVL(segment, 'Bilinmiyor') AS segment
FROM kredi
WHERE acilis_tarihi >= ADD_MONTHS(TRUNC(SYSDATE), -3)
ORDER BY tutar DESC
FETCH FIRST 10 ROWS ONLY;

-- PostgreSQL
SELECT musteri_id, tutar, COALESCE(segment, 'Bilinmiyor') AS segment
FROM kredi
WHERE acilis_tarihi >= CURRENT_DATE - INTERVAL '3 months'
ORDER BY tutar DESC
LIMIT 10;
```

Upsert (varsa güncelle, yoksa ekle) — ETL'lerde çok kullanılır:
```sql
-- PostgreSQL
INSERT INTO bddk_kredi (donem, kalem, tutar)
VALUES ('2026-09', 'tuketici', 1250000)
ON CONFLICT (donem, kalem) DO UPDATE SET tutar = EXCLUDED.tutar;

-- Oracle
MERGE INTO bddk_kredi t
USING (SELECT '2026-09' donem, 'tuketici' kalem, 1250000 tutar FROM dual) s
ON (t.donem = s.donem AND t.kalem = s.kalem)
WHEN MATCHED THEN UPDATE SET t.tutar = s.tutar
WHEN NOT MATCHED THEN INSERT (donem, kalem, tutar) VALUES (s.donem, s.kalem, s.tutar);
```

Python'dan bağlanmak:
```python
from sqlalchemy import create_engine
import pandas as pd

pg = create_engine("postgresql+psycopg2://etl:sifre@pg.sirket.local:5432/bddk")
ora = create_engine("oracle+oracledb://rapor:sifre@ora.sirket.local:1521/?service_name=DWH")

df = pd.read_sql("SELECT * FROM bddk_kredi WHERE donem = '2026-09'", pg)
df.to_sql("bddk_kredi_kopya", pg, if_exists="append", index=False)
```

Terminal araçları:
```bash
psql -h pg.sirket.local -U etl -d bddk          # PostgreSQL
sqlplus rapor@//ora.sirket.local:1521/DWH       # Oracle
```

## Nerede kullanılır?
- **Oracle:** Core banking, kurumsal veri ambarı, kritik işlem sistemleri
- **PostgreSQL:** ETL hedef veritabanları, iç uygulamalar, dashboard arka uçları, Supabase gibi platformların temeli, vektör arama (pgvector)

## Benzerleri ve farkları
- **Microsoft SQL Server:** Windows/Microsoft ekosisteminde yaygın ticari veritabanı (T-SQL).
- **MySQL / MariaDB:** Web uygulamalarında yaygın açık kaynak veritabanları.
- **IBM Db2:** Ana bilgisayar (mainframe) dünyasında, bazı bankaların çekirdek sistemlerinde.
- **Göç (migration):** Oracle'dan PostgreSQL'e geçişte araçlar (ora2pg, AWS SCT) şemayı çevirir; asıl iş PL/SQL kodunun ve uygulama sorgularının uyarlanmasıdır.

## İlgili sayfalar
- [9.2 İlişkisel veritabanları](02-iliskisel-sql-acid.md)
- [9.4 Performans](04-performans.md)
