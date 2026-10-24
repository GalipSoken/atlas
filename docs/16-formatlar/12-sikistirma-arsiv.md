# 16.12 Sıkıştırma ve Arşiv Formatları: zip, gzip, tar, zstd, snappy

## Nedir?
İki ayrı işlev çoğu zaman birbirine karıştırılır:
- **Sıkıştırma (compression):** Verideki tekrarları kullanarak dosyayı küçültmek. `gzip`, `bzip2`, `xz`, `zstd`, `snappy`, `lz4`.
- **Arşivleme (archiving):** Birden çok dosyayı ve klasör yapısını tek dosyada toplamak. `tar`.
- **İkisi birden:** `zip`, `7z`, `rar` hem arşivler hem sıkıştırır. `tar.gz` (`.tgz`) ise önce `tar` ile toplayıp sonra `gzip` ile sıkıştırmaktır.

Sıkıştırma bir dosya formatının **üzerinde** ayrı bir katmandır: `kredi.csv.gz` hâlâ bir CSV'dir. Parquet ve ORC gibi formatlar ise sıkıştırmayı **içeride**, sütun veya blok bazında yapar ([16.5](05-parquet.md)).

## Hangi problemi çözer?
- **Depolama maliyeti:** Metin tabanlı veriler (CSV, JSON, log) genellikle 5–10 kat küçülür.
- **Aktarım süresi:** Ağ üzerinden, SFTP ile veya e-postayla gönderilen dosyalar küçülür.
- **Toplu teslim:** Yüzlerce dosyayı tek paket olarak iletmek (aylık rapor seti, yedek, model ve ekleri).
- **Okuma hızı:** Disk ve ağ çoğu zaman CPU'dan yavaştır; sıkıştırılmış veriyi okuyup açmak, sıkıştırılmamış veriyi okumaktan hızlı olabilir.

## Nasıl çalışır?

**Format karşılaştırması**
| Format | Uzantı | Hız | Oran | Bölünebilir mi? (Spark) | Tipik kullanım |
|---|---|---|---|---|---|
| **gzip** | `.gz` | Orta | İyi | **Hayır** | Log dosyaları, HTTP sıkıştırma, CSV aktarımı |
| **zip** | `.zip` | Orta | İyi | Hayır | Windows'ta her yerde açılan paket; iş birimleri, dış kurumlar |
| **bzip2** | `.bz2` | Yavaş | Daha iyi | Evet | Eski Hadoop ortamları |
| **xz (LZMA)** | `.xz` | Çok yavaş sıkıştırma | Çok iyi | Hayır | Yazılım dağıtımı, uzun süreli arşiv |
| **zstd (Zstandard)** | `.zst` | Hızlı | Çok iyi (seviye ayarlanabilir) | Formata bağlı | Modern varsayılan: Parquet, Kafka, yedekler, veritabanı dökümleri |
| **snappy** | `.snappy` | Çok hızlı | Orta | (format içinde) | Parquet/ORC varsayılanı, Kafka |
| **lz4** | `.lz4` | En hızlı | Orta | (format içinde) | Gerçek zamanlı sistemler, Kafka |
| **7z** | `.7z` | Yavaş | Çok iyi | Hayır | Güçlü AES-256 parolalı arşiv |
| **tar** | `.tar` | — | Sıkıştırmaz | — | Linux'ta klasör paketleme, Docker image katmanları |

**Bölünebilirlik (splittable) neden önemli?** Spark veya Hadoop büyük bir dosyayı parçalara bölüp paralel okur. 20 GB'lık tek bir `islem.csv.gz` dosyası bölünemediği için **tek bir çekirdekte** okunur; cluster'ın geri kalanı bekler. Çözüm: dosyayı birçok küçük `.gz` parçasına bölmek veya Parquet'e çevirmek.

**Sıkıştırma seviyesi:** Çoğu araçta 1 (hızlı, büyük) ile 9 (yavaş, küçük) arası; zstd'de 1–19 (ve "ultra" modda 22). Günlük işler için varsayılan seviye (gzip 6, zstd 3) genellikle en iyi dengedir.

**Neyi sıkıştırmanın faydası yok?** Zaten sıkıştırılmış formatlar: JPEG, PNG, MP4, Parquet, xlsx (içi zaten zip'tir), şifrelenmiş dosyalar. Bunları tekrar sıkıştırmak boyutu neredeyse değiştirmez, sadece CPU harcar.

**Güvenlik notları**
| Risk | Açıklama | Önlem |
|---|---|---|
| **Zip bomb** | KB'larca arşiv açılınca GB'larca/TB'larca veri üretir | Açılmış boyut ve dosya sayısı sınırı koyun |
| **Zip slip (yol gezinme)** | Arşivdeki `../../etc/cron.d/x` gibi yollar hedef klasörün dışına yazar | Dış kaynaklı arşivleri ayrı ve boş bir klasöre açın, yolları kontrol edin |
| **Zayıf zip parolası** | Klasik ZipCrypto şifrelemesi kolay kırılır | Hassas veri için AES-256 (7z veya AES destekli zip) ya da GPG; en iyisi kurumsal güvenli aktarım kanalı ([15.8](../15-guvenlik/08-sifreleme.md)) |
| **Parolayı aynı e-postada göndermek** | Arşivi korumanın anlamı kalmaz | Parolayı farklı kanaldan iletin |

## Mini örnek
**Komut satırı**
```bash
gzip -k islem.csv                  # islem.csv.gz oluşturur, orijinali korur (-k)
gunzip islem.csv.gz                # açar
zcat islem.csv.gz | head           # açmadan ilk satırlar
zgrep "ERROR" app.log.gz           # sıkıştırılmış logda arama

tar -czf rapor_2026_09.tar.gz rapor/          # klasörü paketle + gzip
tar -tzf rapor_2026_09.tar.gz                 # içeriği listele
tar -xzf rapor_2026_09.tar.gz -C /tmp/acik    # belirli klasöre aç
tar --zstd -cf yedek.tar.zst veri/            # zstd ile (GNU tar 1.31+)

zip -r rapor.zip rapor/            # zip arşivi
unzip -l rapor.zip                 # açmadan listele
unzip rapor.zip -d hedef/

zstd -19 -T0 dump.sql              # yüksek seviye, tüm çekirdekler
zstd -d dump.sql.zst

split -l 5000000 islem.csv parca_ && gzip parca_*   # büyük dosyayı bölünebilir parçalara ayır
```

**pandas ve PyArrow**
```python
import pandas as pd

df = pd.read_csv("islem.csv.gz")                    # uzantıdan sıkıştırma otomatik anlaşılır
df = pd.read_csv("rapor.zip")                       # zip içinde tek dosya varsa doğrudan okunur
df.to_csv("cikti.csv.gz", index=False)              # gzip ile yaz
df.to_csv("cikti.zip", index=False,
          compression={"method": "zip", "archive_name": "cikti.csv"})
df.to_parquet("cikti.parquet", compression="zstd")  # Parquet içi sıkıştırma
df.to_json("olaylar.jsonl.gz", orient="records", lines=True, compression="gzip")
```

**Zip içinde birden çok dosya ve güvenli açma**
```python
import zipfile
from pathlib import Path

hedef = Path("/veri/gelen/2026-09").resolve()
with zipfile.ZipFile("bddk_paket.zip") as z:
    toplam = sum(i.file_size for i in z.infolist())
    assert toplam < 5 * 1024**3, "Açılmış boyut 5 GB sınırını aşıyor"
    for bilgi in z.infolist():
        yol = (hedef / bilgi.filename).resolve()
        if not yol.is_relative_to(hedef):            # zip slip kontrolü (Python 3.9+)
            raise ValueError(f"Şüpheli yol: {bilgi.filename}")
    z.extractall(hedef)

    with z.open("krediler.xlsx") as f:               # açmadan tek dosyayı okumak
        df = pd.read_excel(f)
```

**Spark**
```python
df = spark.read.option("header", True).csv("s3a://veri-lake/bronze/islem/*.csv.gz")   # her .gz bir görev
df.write.option("compression", "zstd").parquet("s3a://veri-lake/silver/islem/")
```

**HTTP ve nginx tarafı:** Web sunucuları cevapları `gzip` veya `br` (Brotli) ile sıkıştırarak gönderir; tarayıcı `Accept-Encoding` başlığıyla desteklediklerini bildirir ([8.7](../08-nginx/07-cache-rate-limit.md)).

## Nerede kullanılır?
- Dış kurumlardan gelen veri paketleri (zip) ve SFTP aktarımları
- Log arşivleri ve logrotate (`.gz`)
- Veritabanı yedekleri ve dökümleri (`pg_dump | zstd`)
- Data lake'te ham dosyalar ve Parquet içi sıkıştırma
- Kafka mesaj sıkıştırması (lz4, zstd, snappy)
- Docker image katmanları (tar + gzip/zstd)

## Benzerleri ve farkları
- **Sütun bazlı formatların iç sıkıştırması:** Parquet'te sözlük kodlama + snappy/zstd, dışarıdan gzip'lemekten çok daha verimlidir ve bölünebilirliği korur ([16.5](05-parquet.md)).
- **Veritabanı sıkıştırması:** Oracle Advanced/Hybrid Columnar Compression, PostgreSQL TOAST ve tablo erişim yöntemleri; sıkıştırmayı depolama motoru yapar.
- **Deduplication:** Yedekleme sistemlerinde aynı blokları bir kez saklamak; sıkıştırmadan farklı ama tamamlayıcı bir teknik.

## İlgili sayfalar
- [16.1 Formatlara genel bakış](01-genel-bakis.md)
- [16.5 Parquet](05-parquet.md)
- [9.15 Nesne depolama](../09-veri-depolama/15-nesne-depolama.md)
- [Linux / bash cheat sheet](../ekler/linux-bash-cheatsheet.md)
