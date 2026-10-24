# 9.21 Hint'ler: Optimizer'a Yön Vermek

## Nedir?
**Hint**, SQL sorgusunun içine yazılan ve optimizer'a "bu sorguyu şu şekilde çalıştır" diye öneride bulunan özel bir yorumdur. Optimizer normalde planı istatistiklere göre kendisi seçer ([9.19](19-execution-plan.md)); hint bu kararı belirli bir yöne zorlar.

```sql
SELECT /*+ INDEX(i ix_islem_tarih) */ ...      -- Oracle: "bu index'i kullan"
```

Hint desteği veritabanına göre çok farklıdır:
| Veritabanı | Hint desteği |
|---|---|
| **Oracle** | Çok zengin ve yaygın; `/*+ ... */` yorumu |
| **SQL Server** | `WITH (INDEX(...))`, `OPTION (HASH JOIN, MAXDOP 4)` |
| **MySQL** | `/*+ ... */` ve `USE INDEX`, `FORCE INDEX` |
| **PostgreSQL** | **Yerleşik hint yok** (bilinçli tasarım tercihi). `pg_hint_plan` eklentisi veya oturum ayarları (`enable_nestloop` vb.) kullanılır |
| **Spark SQL** | `/*+ BROADCAST(t) */`, `REPARTITION`, `COALESCE` |

## Hangi problemi çözer?
- Optimizer'ın yanlış satır tahmini nedeniyle kötü plan seçtiği ve kısa vadede başka çözümün olmadığı acil durumlar
- Toplu yükleme ve paralellik gibi, optimizer'ın kendiliğinden seçmediği **çalışma biçimlerini** açmak (Oracle `APPEND`, `PARALLEL`)
- Spark'ta küçük tabloyu yayınlamak (broadcast) gibi dağıtık stratejileri belirlemek

## Nasıl çalışır?
- Hint bir **yorum** biçimindedir; yazım hatası yapılırsa veritabanı **hata vermeden yok sayar**. Hint'in uygulanıp uygulanmadığı plana bakılarak doğrulanmalıdır.
- Hint'te tabloya sorgudaki **takma adıyla (alias)** başvurulur: `FROM islem i` → `INDEX(i ...)`.
- Oracle'da hint, `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` anahtar kelimesinin hemen ardından gelir.

**Sık kullanılan Oracle hint'leri**
| Hint | Etkisi |
|---|---|
| `INDEX(t ix_adi)` / `NO_INDEX(t ix_adi)` | Index'i kullan / kullanma |
| `FULL(t)` | Tabloyu tamamen tara |
| `LEADING(t1 t2)` | Join sırası: önce t1, sonra t2 |
| `USE_NL(t)` / `USE_HASH(t)` / `USE_MERGE(t)` | Join stratejisi ([9.20](20-join-stratejileri.md)) |
| `PARALLEL(t 8)` | 8 paralel süreçle çalış |
| `APPEND` | `INSERT ... SELECT`'te doğrudan yol (direct-path) yükleme: tablonun sonuna toplu yazar, çok hızlıdır |
| `MATERIALIZE` / `INLINE` | `WITH` (CTE) bloğunu geçici sonuç olarak sakla / sorguya göm |
| `NO_MERGE`, `NO_UNNEST` | Alt sorguların birleştirilmesini engelle |
| `CARDINALITY(t 1000)` | Optimizer'a satır sayısı tahmini ver (belgelenmemiş, dikkatli) |
| `GATHER_PLAN_STATISTICS` | Gerçek satır sayılarını plan çıktısına ekle (teşhis için) |
| `QB_NAME(ad)` | Sorgu bloğuna isim ver; karmaşık sorgularda hint hedeflemek için |

**Neden son çare olmalı?**
| Risk | Açıklama |
|---|---|
| Veri değişir | Bugün doğru olan plan, tablo 10 kat büyüyünce yanlış olur; hint optimizer'ın uyum sağlamasını engeller |
| Sessiz başarısızlık | Index adı değişirse veya silinirse hint sessizce yok sayılır |
| Sürüm yükseltmeleri | Yeni optimizer daha iyi plan bulabilir; hint bunu engeller |
| Bakım | Kodun içine gömülü performans kararlarını kimse hatırlamaz |

**Hint'ten önce denenecekler**
1. İstatistikleri güncellemek (`ANALYZE`, `DBMS_STATS`)
2. Sorguyu index'i kullanabilecek şekilde yeniden yazmak ([9.18](18-index-derinlemesine.md))
3. Eksik index veya partition eklemek
4. Sorguyu sadeleştirmek, ara sonucu geçici tabloya yazmak
5. Oracle'da **SQL Plan Baseline / SQL Profile**: Kodu değiştirmeden, DBA'nın iyi bir planı sabitlemesi

## Mini örnek
**Oracle: hızlı toplu yükleme**
```sql
INSERT /*+ APPEND PARALLEL(h 4) */ INTO dwh.islem_gunluk h
SELECT /*+ PARALLEL(s 4) */ * FROM staging.islem_gunluk s
WHERE islem_tarihi = DATE '2026-10-23';
COMMIT;      -- APPEND sonrası aynı transaction'da tabloyu okumak için commit gerekir
```

**Oracle: join sırası ve stratejisi**
```sql
SELECT /*+ LEADING(m) USE_HASH(i) FULL(i) */
       m.segment, SUM(i.tutar)
FROM musteri m
JOIN islem i ON i.musteri_id = m.musteri_id
WHERE m.segment = 'KOBI'
GROUP BY m.segment;
```

**Oracle: CTE'yi bir kez hesapla**
```sql
WITH aktif AS (
    SELECT /*+ MATERIALIZE */ musteri_id FROM kredi WHERE durum = 'AKTIF'
)
SELECT ... FROM aktif a JOIN ... ;
```

**PostgreSQL: hint yerine oturum ayarı (teşhis için)**
```sql
BEGIN;
SET LOCAL enable_nestloop = off;     -- sadece bu transaction'da nested loop'u kapat
EXPLAIN ANALYZE SELECT ... ;         -- plan ve süre iyileşiyor mu?
ROLLBACK;
```
Bu yöntem kalıcı çözüm değil, sorunun kaynağını gösteren bir **deneydir**. Kalıcı çözüm genellikle istatistik, index veya sorgu yapısındadır.

**pg_hint_plan eklentisi (kuruluysa)**
```sql
/*+ HashJoin(i m) SeqScan(i) */
SELECT ... FROM islem i JOIN musteri m ON ... ;
```

**Spark SQL**
```sql
SELECT /*+ BROADCAST(s) */ i.*, s.sube_adi
FROM islem i JOIN sube s ON s.sube_kodu = i.sube_kodu;
```

## Nerede kullanılır?
- Oracle tabanlı veri ambarlarında toplu yükleme (`APPEND`, `PARALLEL`)
- Kritik bir rapor sorgusunun ani plan değişikliğiyle yavaşladığı acil durumlar
- Spark işlerinde join ve dağılım kontrolü

## Benzerleri ve farkları
- **SQL Plan Management (Oracle):** Planı kod dışında, veritabanında sabitler; hint'ten daha yönetilebilir.
- **Query Store (SQL Server):** Plan geçmişini izler ve iyi planı zorlamayı sağlar.
- **Oturum parametreleri:** `work_mem`, `parallel_degree_policy` gibi ayarlar bütün sorguları etkiler; hint sadece tek sorguyu.

## İlgili sayfalar
- [9.19 Execution plan](19-execution-plan.md)
- [9.20 Join stratejileri](20-join-stratejileri.md)
- [9.3 PostgreSQL ve Oracle](03-postgres-oracle.md)
