# 1.8 cron ile Zamanlama

## Nedir?
**cron**, Linux'ta komutları belirli zamanlarda otomatik çalıştıran servistir. Zamanlanmış görevlerin listesine **crontab** denir.

## Hangi problemi çözer?
"Her sabah 07:00'de ETL'i çalıştır", "her pazartesi raporu üret", "her saat başı eski logları temizle" gibi tekrarlanan işleri insan müdahalesi olmadan yapar.

## Nasıl çalışır?
Her satır bir görevdir: 5 zaman alanı + çalıştırılacak komut.

```
┌───────── dakika (0-59)
│ ┌─────── saat (0-23)
│ │ ┌───── ayın günü (1-31)
│ │ │ ┌─── ay (1-12)
│ │ │ │ ┌─ haftanın günü (0-6, 0 = pazar)
│ │ │ │ │
* * * * *  komut
```

| İfade | Anlamı |
|---|---|
| `0 7 * * *` | Her gün 07:00 |
| `30 6 * * 1-5` | Hafta içi her gün 06:30 |
| `0 * * * *` | Her saat başı |
| `*/15 * * * *` | Her 15 dakikada bir |
| `0 9 1 * *` | Her ayın 1'i, 09:00 |
| `0 22 * * 0` | Her pazar 22:00 |

İfadeleri test etmek için: [crontab.guru](https://crontab.guru)

## Mini örnek
```bash
crontab -l        # mevcut görevleri listele
crontab -e        # düzenle
```

Örnek crontab:
```cron
# Ortam değişkenleri (cron kısıtlı bir ortamda çalışır!)
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin

# Hafta içi her sabah 07:00'de BDDK ETL'i
0 7 * * 1-5  cd /opt/etl && /opt/etl/.venv/bin/python bddk_etl.py >> /var/log/etl/bddk.log 2>&1

# Her gece 02:00'de 30 günden eski logları sil
0 2 * * *  find /var/log/etl -name "*.log" -mtime +30 -delete
```

## Nerede kullanılır?
- Tek sunucuda çalışan basit ETL ve rapor işleri
- Yedekleme, log temizliği
- Veri dosyası kontrolü ve bildirim

## Benzerleri ve farkları
| Araç | Ne zaman? |
|---|---|
| **cron** | Tek sunucu, basit ve bağımsız işler |
| **Kubernetes CronJob** | Aynı mantık ama container olarak, cluster üzerinde ([5.6](../05-kubernetes/06-job-cronjob.md)) |
| **Airflow / n8n** | Adımları birbirine bağımlı, yeniden deneme ve izleme gereken iş akışları (Bölüm 11) |
| **systemd timer** | cron'un modern alternatifi, loglar journalctl'de |

**Sık hatalar:**
- **cron'un ortamı kısıtlıdır:** Terminalde çalışan komut cron'da "command not found" verebilir. Tam yol kullanın (`/opt/etl/.venv/bin/python`) ve PATH tanımlayın.
- **Çıktıyı yönlendirmemek:** `>> log 2>&1` olmadan hata mesajları kaybolur.
- **Saat dilimi:** Sunucu UTC'de çalışıyorsa 07:00 Türkiye saatiyle 10:00 olur. `timedatectl` ile kontrol edin.
- **Üst üste binme:** Önceki çalışma bitmeden yenisi başlayabilir; `flock` ile engellenebilir.

## İlgili sayfalar
- [1.7 Bash script](07-bash-script.md)
- [1.4 Süreçler ve servisler](04-surecler.md)
- [cron ve tarih/zaman formatları cheat sheet](../ekler/cron-zaman-cheatsheet.md)
