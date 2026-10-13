# 1.7 Pipe, Yönlendirme ve Bash Script

## Nedir?
- **Pipe (`|`):** Bir komutun çıktısını, bir sonraki komutun girdisi yapar.
- **Yönlendirme (`>`, `>>`, `2>`):** Çıktıyı ekran yerine bir dosyaya yazar.
- **Bash script:** Komutların bir dosyaya yazılıp tek seferde çalıştırılmasıdır.

## Hangi problemi çözer?
Linux felsefesi: "Her araç tek işi iyi yapsın, araçlar birleştirilsin." Küçük komutlar pipe ile birleştirilince güçlü analizler yapılır. Tekrarlanan işler script'e dönüştürülünce elle yapılan hatalar azalır.

## Nasıl çalışır?

Her sürecin üç standart akışı vardır:

| Akış | Numara | Anlamı |
|---|---|---|
| stdin | 0 | Girdi |
| stdout | 1 | Normal çıktı |
| stderr | 2 | Hata çıktısı |

| İfade | Anlamı |
|---|---|
| `komut > dosya` | Çıktıyı dosyaya yaz (üzerine yazar) |
| `komut >> dosya` | Dosyanın sonuna ekle |
| `komut 2> hata.log` | Sadece hataları dosyaya yaz |
| `komut > log 2>&1` | Çıktı ve hataları aynı dosyaya yaz |
| `komut1 \| komut2` | 1'in çıktısı 2'nin girdisi |
| `komut1 && komut2` | 1 başarılıysa 2'yi çalıştır |
| `komut1 \|\| komut2` | 1 başarısızsa 2'yi çalıştır |

## Mini örnek

**Pipe ile log analizi**
```bash
# En çok hata veren 5 modül
grep "ERROR" app.log | awk '{print $4}' | sort | uniq -c | sort -rn | head -5

# CSV'de 3. sütundaki benzersiz değerler
cut -d',' -f3 veri.csv | sort -u

# Bugünkü dosya sayısı
ls /data/inbox | grep "$(date +%Y%m%d)" | wc -l
```

**Basit bir ETL script'i: `yukle.sh`**
```bash
#!/usr/bin/env bash
set -euo pipefail        # hata olursa dur, tanımsız değişkene izin verme

TARIH=$(date +%Y-%m-%d)
KAYNAK_DIR="/data/inbox"
LOG="/var/log/etl/yukle_${TARIH}.log"

echo "[$(date)] Yükleme başladı" >> "$LOG"

if [ ! -d "$KAYNAK_DIR" ]; then
    echo "Klasör yok: $KAYNAK_DIR" >&2
    exit 1
fi

for dosya in "$KAYNAK_DIR"/*.csv; do
    echo "İşleniyor: $dosya" >> "$LOG"
    python3 /opt/etl/yukle.py --dosya "$dosya" >> "$LOG" 2>&1
done

echo "[$(date)] Yükleme bitti" >> "$LOG"
```

```bash
chmod +x yukle.sh
./yukle.sh
echo $?        # son komutun çıkış kodu: 0 = başarılı, diğerleri = hata
```

`set -euo pipefail` satırı önemlidir: `-e` hata olunca durdurur, `-u` tanımsız değişkende hata verir, `-o pipefail` pipe içindeki hatayı da yakalar.

## Nerede kullanılır?
- Dosya kontrolü, taşıma, arşivleme gibi küçük operasyon işleri
- Python ETL'lerini sırayla çağıran sarmalayıcı (wrapper) script'ler
- Docker container'larının başlangıç (entrypoint) script'leri
- CI/CD pipeline adımları (pipeline'lar aslında shell komutları çalıştırır)

## Benzerleri ve farkları
- **Python script:** Mantık karmaşıklaşınca (veri işleme, API, hata yönetimi) Python daha okunaklıdır. Kural: 30–50 satırı geçen bash script Python'a taşınmayı düşünmelidir.
- **Makefile:** Proje görevlerini (`make test`, `make build`) tanımlamak için.
- **Orkestrasyon araçları (Airflow vb.):** Çok adımlı, bağımlılıkları olan işler için (Bölüm 11).

## İlgili sayfalar
- [1.8 cron](08-cron.md)
- [1.2 Temel komutlar](02-temel-komutlar.md)
