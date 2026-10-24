# 9.18 Index Derinlemesine: Doğru Index, Kullanılmayan Index

## Nedir?
Index'in temel tanımı ve türleri [9.4](04-performans.md)'te anlatıldı. Bu sayfa pratikteki soruları ele alır: **Hangi sütunlara, hangi sırayla index konur? Index varken veritabanı neden onu kullanmaz? Index ne zaman zarar verir?**

## Hangi problemi çözer?
- "Index ekledim ama sorgu hâlâ yavaş."
- "Tabloya 15 index eklenmiş, yükleme saatler sürüyor."
- "Aynı sorgu dün hızlıydı, bugün yavaş."
- Dashboard'ların arkasındaki sorguların milisaniyelere inmesi.

## Nasıl çalışır?

**Seçicilik (selectivity) ve kardinalite**
- **Kardinalite:** Sütundaki farklı değer sayısı. `musteri_id` yüksek, `cinsiyet` çok düşük kardinaliteli.
- **Seçicilik:** Filtrenin tablonun ne kadarını döndürdüğü. `WHERE musteri_id = 123` → 1 satır (yüksek seçicilik); `WHERE aktif = true` → %90 (düşük).
- Kaba kural: Sorgu tablonun **küçük bir kısmını** (çoğu durumda yüzde birkaçından azını) döndürüyorsa index faydalıdır. Büyük kısmını döndürüyorsa tabloyu baştan sona okumak (full scan) daha hızlıdır ve optimizer bunu seçer.

**Bileşik (composite) index'te sütun sırası**
Index `(donem, segment, sube_kodu)` bir telefon rehberi gibi sıralıdır: önce dönem, sonra segment, sonra şube.
| Sorgu koşulu | Index kullanılır mı? |
|---|---|
| `donem = ?` | Evet |
| `donem = ? AND segment = ?` | Evet |
| `donem = ? AND sube_kodu = ?` | Kısmen (sadece dönem kısmı daraltır) |
| `segment = ?` | Genellikle hayır (en soldaki sütun yok) |
| `donem BETWEEN ? AND ? AND segment = ?` | Kısmen: aralık koşulundan sonraki sütunlar daha az yardımcı olur |

**Sıralama önerisi:** Önce eşitlikle (`=`) filtrelenen sütunlar, sonra aralık (`>`, `BETWEEN`) veya sıralama (`ORDER BY`) sütunları.

**Kapsayan index (covering index) ve index-only scan**
Sorgunun ihtiyaç duyduğu bütün sütunlar index'te varsa veritabanı tabloya hiç gitmeden cevap verebilir.
```sql
CREATE INDEX ix_islem_musteri_tarih ON islem (musteri_id, tarih) INCLUDE (tutar);   -- PostgreSQL 11+
SELECT tarih, tutar FROM islem WHERE musteri_id = 123 AND tarih >= DATE '2026-01-01';
```

**Index'in kullanılmadığı yaygın durumlar**
| Durum | Kötü | İyi |
|---|---|---|
| Sütuna fonksiyon uygulamak | `WHERE UPPER(unvan) = 'ABC'` | Fonksiyon tabanlı index veya veriyi normalize saklamak |
| Tarih sütununu dönüştürmek | `WHERE TO_CHAR(tarih,'YYYY-MM') = '2026-09'` | `WHERE tarih >= DATE '2026-09-01' AND tarih < DATE '2026-10-01'` |
| Örtük tip dönüşümü | `WHERE musteri_no = 12345` (sütun metin) | `WHERE musteri_no = '12345'` |
| Başı joker `LIKE` | `WHERE unvan LIKE '%bank%'` | Tam metin arama, trigram (`pg_trgm`) index |
| Sütun üzerinde aritmetik | `WHERE tutar * 1.2 > 1000` | `WHERE tutar > 1000 / 1.2` |
| `OR` ile farklı sütunlar | `WHERE a = 1 OR b = 2` | Ayrı index'ler + `UNION ALL` (veya optimizer'ın bitmap birleştirmesi) |
| `NOT IN`, `<>` | Çoğu satırı döndürür | Genellikle full scan doğrudur |
| İstatistikler eski | Optimizer satır sayısını yanlış tahmin eder | `ANALYZE` / `DBMS_STATS.GATHER_TABLE_STATS` |

**Özel index türleri**
| Tür | Kullanım |
|---|---|
| **Fonksiyon / ifade index'i** | `CREATE INDEX ON musteri (UPPER(unvan));` |
| **Partial (kısmi) index** (PostgreSQL) | Sadece ilgili satırlar: `CREATE INDEX ON basvuru (tarih) WHERE durum = 'BEKLIYOR';` |
| **Unique index** | İş kuralı: aynı dönem + kalem iki kez yüklenemez |
| **GIN + pg_trgm** | `LIKE '%...%'` ve benzer metin araması |
| **BRIN** | Zamanla sıralı büyüyen log/işlem tabloları; çok küçük |
| **Bitmap** (Oracle) | Ambar tablolarında düşük kardinaliteli sütunlar; OLTP'de kilit sorunu yaratır |

**Index'in bedeli**
- Her `INSERT`/`UPDATE`/`DELETE` ilgili index'leri de günceller.
- Büyük toplu yüklemelerde: index'leri yükleme öncesi kaldırıp sonra yeniden oluşturmak çoğu zaman daha hızlıdır.
- Kullanılmayan index'ler düzenli tespit edilip kaldırılmalıdır.
- Zamanla şişen (bloat) index'ler `REINDEX` ile yeniden oluşturulabilir.

## Mini örnek
**Kullanılmayan ve en çok kullanılan index'ler (PostgreSQL)**
```sql
SELECT schemaname, relname AS tablo, indexrelname AS index_adi,
       idx_scan AS kullanim_sayisi,
       pg_size_pretty(pg_relation_size(indexrelid)) AS boyut
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC
LIMIT 20;
```

**İstatistik güncelleme**
```sql
ANALYZE dwh.bddk_kredi;                                           -- PostgreSQL
EXEC DBMS_STATS.GATHER_TABLE_STATS('DWH', 'BDDK_KREDI');          -- Oracle (SQL*Plus)
```
Büyük yüklemelerden sonra istatistik toplamak, plan hatalarının en ucuz çözümüdür.

**Dashboard sorgusu için index tasarımı**
```sql
-- Sorgu: belirli dönemde, segment bazında toplam
SELECT segment, SUM(tutar)
FROM kredi_bakiye
WHERE donem = DATE '2026-09-30' AND urun_grubu = 'KMH'
GROUP BY segment;

-- Eşitlik sütunları önde, toplanan sütunlar INCLUDE ile
CREATE INDEX ix_kredi_bakiye_donem_urun
    ON kredi_bakiye (donem, urun_grubu) INCLUDE (segment, tutar);
```

**Canlı tabloda index oluşturmak**
```sql
CREATE INDEX CONCURRENTLY ix_islem_musteri ON islem (musteri_id);     -- PostgreSQL
CREATE INDEX ix_islem_musteri ON islem (musteri_id) ONLINE;           -- Oracle (Enterprise)
```

## Nerede kullanılır?
- Dashboard ve API'lerin arkasındaki tekrar eden sorgular
- ETL'lerde join ve MERGE anahtarları
- Benzersizlik kuralları (unique index)
- Arama ekranları (ünvan, TCKN/VKN, müşteri no)

## Benzerleri ve farkları
- **Partitioning:** Index satırı bulur; partitioning okunacak veri miktarını parçalara göre azaltır. Büyük tablolarda ikisi birlikte kullanılır ([9.23](23-partitioning-derinlemesine.md)).
- **Materialized view / özet tablo:** Index yetmediğinde sonucu önceden hesaplayıp saklamak.
- **Kolon bazlı depolama:** Analitik veritabanları (ClickHouse, Snowflake, DuckDB, Parquet) klasik index yerine min/max istatistikleri ve sıralamayla veri atlar ([17.5](../17-formatlar/05-parquet.md)).

## İlgili sayfalar
- [9.4 Veritabanı performansı](04-performans.md)
- [9.19 Execution plan](19-execution-plan.md)
- [9.21 Hint'ler](21-hint.md)
