# Cron ve Tarih/Zaman Formatları Cheat Sheet

Zamanlanmış işler (cron, Kubernetes CronJob, Airflow) ve tarih/saat verisi, veri işlerinde en sık hata kaynaklarındandır: yanlış saat dilimi, ay sonu, yaz saati, `MM/DD` ile `DD/MM` karışması. Kavramsal anlatım için [1.8 cron](../01-linux/08-cron.md) ve [11.8 Orkestrasyon](../11-veri-akisi/08-orkestrasyon.md).

## Cron ifadesi
```
┌───────────── dakika      (0-59)
│ ┌─────────── saat        (0-23)
│ │ ┌───────── ayın günü   (1-31)
│ │ │ ┌─────── ay          (1-12 veya JAN-DEC)
│ │ │ │ ┌───── haftanın günü (0-7; 0 ve 7 = Pazar, 1 = Pazartesi, veya MON-SUN)
│ │ │ │ │
* * * * *  komut
```

| Sembol | Anlamı | Örnek |
|---|---|---|
| `*` | Her değer | `*` saat alanında: her saat |
| `,` | Liste | `1,15` → ayın 1'i ve 15'i |
| `-` | Aralık | `1-5` → Pazartesi–Cuma |
| `/` | Adım | `*/15` → her 15 dakikada |

## Hazır ifadeler
| İfade | Anlamı |
|---|---|
| `*/5 * * * *` | Her 5 dakikada |
| `0 * * * *` | Her saat başı |
| `30 6 * * *` | Her gün 06:30 |
| `0 7 * * 1-5` | Hafta içi her gün 07:00 |
| `0 8 * * 1` | Her Pazartesi 08:00 |
| `0 2 1 * *` | Her ayın 1'i 02:00 (aylık BDDK/MIS yüklemeleri) |
| `0 9 5 * *` | Her ayın 5'i 09:00 |
| `0 3 1 1,4,7,10 *` | Çeyrek başlarında 03:00 |
| `0 0 * * 0` | Her Pazar gece yarısı |
| `@reboot` | Sunucu açılışında (sadece crontab) |
| `@daily`, `@hourly`, `@weekly`, `@monthly` | Kısaltmalar |

**Tuzaklar**
- **Ayın son günü** standart cron'da ifade edilemez. Yaygın çözüm: her gün `28-31` arası çalıştırıp betik içinde "yarın ayın 1'i mi?" kontrolü: `[ "$(date -d tomorrow +\%d)" = "01" ] && ./ay_sonu.sh`
- crontab içinde `%` özel karakterdir; `\%` olarak kaçırılmalıdır.
- Ayın günü **ve** haftanın günü birlikte verilirse çoğu cron uygulamasında **VEYA** ile birleşir: `0 9 1 * 1` → ayın 1'i **veya** her Pazartesi.
- Cron, kullanıcının etkileşimli kabuk ortamını yüklemez: `PATH`, conda/venv etkin değildir; tam yol kullanın ([1.5](../01-linux/05-environment.md)).
- Sunucu saat dilimini kontrol edin: `timedatectl` veya `date`.
- Bazı sistemler (Quartz, Spring, Jenkins `H`) **6–7 alanlı** veya farklı söz dizimi kullanır; ifadeyi aracın belgesine göre yazın.

```bash
crontab -l                      # listele
crontab -e                      # düzenle
# Örnek satır: her iş günü 07:00, çıktıyı log'a yaz
0 7 * * 1-5 /opt/etl/.venv/bin/python /opt/etl/gunluk.py >> /var/log/etl/gunluk.log 2>&1
```

## Kubernetes / OpenShift CronJob
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: bddk-aylik-yukleme
spec:
  schedule: "0 2 1 * *"              # tırnak içinde yazın: * ile başlayan değer YAML'da alias sayılır
  timeZone: "Europe/Istanbul"        # yoksa cluster saat dilimi (genelde UTC) kullanılır
  concurrencyPolicy: Forbid          # önceki iş bitmeden yenisini başlatma
  startingDeadlineSeconds: 3600      # kaçırılırsa 1 saat içinde yine de başlat
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: yukleme
              image: registry.sirket.local/veri/bddk-yukleme:1.4.0
              args: ["--donem", "onceki-ay"]
```

## Airflow
```python
DAG(
    dag_id="bddk_aylik",
    schedule="0 2 1 * *",                                        # cron ifadesi veya "@daily"
    start_date=pendulum.datetime(2026, 1, 1, tz="Europe/Istanbul"),
    catchup=False,                                               # geçmiş aralıkları doldurma
)
```
Airflow'da bir çalışma, **veri aralığının sonunda** başlar: `@daily` için 13 Ekim'deki çalışma 12 Ekim verisini işler (`data_interval_start` / `data_interval_end`).

## Saat dilimleri
| Kavram | Açıklama |
|---|---|
| **UTC** | Evrensel referans saat. Sunucular, cluster'lar ve loglar genellikle UTC'dir |
| **Europe/Istanbul** | Türkiye saati: UTC+3, 2016'dan beri yaz saati uygulanmıyor |
| **IANA adı** | `Europe/Istanbul` gibi bölge adı kullanın; `+03:00` sabit fark tarih kuralı değişikliklerini bilmez |
| **Naive / aware** | Saat dilimi bilgisi olmayan (`naive`) ve olan (`aware`) zaman değerleri |

**Altın kural:** Saklarken **UTC** (veya saat dilimli tip), kullanıcıya gösterirken **yerel saat**. Tarih (gün) bazlı iş anlamı olan alanlarda (`islem_tarihi`, `donem`) yerel tarihi `DATE` olarak saklayın.

Türkiye'de UTC+3 nedeniyle **gece 00:00–03:00 arası** işlemler UTC'de bir **önceki güne** düşer; günlük toplamlar hatalı çıkar.

## Tarih formatları
| Format | Örnek | Not |
|---|---|---|
| ISO 8601 tarih | `2026-10-13` | Tercih edilen; metin olarak sıralanabilir |
| ISO 8601 zaman | `2026-10-13T09:30:00+03:00` | JSON/API standardı ([7.3](../07-api/03-rest-json.md)) |
| UTC (`Z`) | `2026-10-13T06:30:00Z` | `Z` = UTC |
| Dönem | `2026-09` veya `202609` | Aylık raporlar |
| Türkiye gösterimi | `13.10.2026` | Sadece arayüz/rapor için |
| ABD gösterimi | `10/13/2026` | `DD/MM` ile karıştırılmaya çok açık |
| Unix epoch (saniye) | `1791873000` | 1970-01-01 UTC'den beri geçen saniye |
| Epoch (milisaniye) | `1791873000000` | JavaScript, Kafka, MongoDB |

## strftime / strptime kodları
| Kod | Anlamı | Örnek |
|---|---|---|
| `%Y` / `%y` | Yıl (4 / 2 hane) | `2026` / `26` |
| `%m` | Ay (01–12) | `10` |
| `%d` | Gün (01–31) | `13` |
| `%H` / `%I` | Saat (24 / 12) | `09` |
| `%M` / `%S` | Dakika / saniye | `30` / `00` |
| `%f` | Mikrosaniye | `000123` |
| `%z` / `%Z` | UTC farkı / dilim adı | `+0300` / `+03` |
| `%j` | Yılın günü | `286` |
| `%A` / `%B` | Gün / ay adı (yerele göre) | `Tuesday` / `October` |
| `%V` | ISO hafta numarası | `42` |

## Python
```python
from datetime import datetime, date, timedelta, timezone
from zoneinfo import ZoneInfo

IST = ZoneInfo("Europe/Istanbul")

simdi_utc = datetime.now(timezone.utc)                     # aware UTC
simdi_tr = simdi_utc.astimezone(IST)                       # yerel saate çevir
datetime.strptime("13.10.2026", "%d.%m.%Y").date()          # metin → tarih
simdi_tr.strftime("%Y-%m-%d %H:%M")                         # tarih → metin
datetime.fromisoformat("2026-10-13T09:30:00+03:00")
datetime.fromtimestamp(1791873000, tz=timezone.utc)         # epoch → zaman

# Önceki ayın dönemi ve son günü
bugun = date.today()
onceki_ay_son = bugun.replace(day=1) - timedelta(days=1)
donem = onceki_ay_son.strftime("%Y-%m")
```

## pandas
```python
df["tarih"] = pd.to_datetime(df["tarih_metin"], format="%d.%m.%Y")      # formatı açıkça verin
df["ts"] = pd.to_datetime(df["ts_metin"], utc=True).dt.tz_convert("Europe/Istanbul")
df["ts_epoch"] = pd.to_datetime(df["epoch_ms"], unit="ms", utc=True)
df["donem"] = df["tarih"].dt.to_period("M")                               # 2026-10
df["ay_sonu"] = df["tarih"] + pd.offsets.MonthEnd(0)
pd.date_range("2026-01-31", periods=12, freq="ME")                        # ay sonları
```
`dayfirst=True` yerine **`format=`** vermek daha güvenlidir; `03.04.2026` hem 3 Nisan hem 4 Mart olarak okunabilir.

## SQL
```sql
-- PostgreSQL
SELECT now(), now() AT TIME ZONE 'Europe/Istanbul';
SELECT date_trunc('month', islem_ts) AS ay, SUM(tutar) FROM islem GROUP BY 1;
SELECT to_char(islem_ts, 'YYYY-MM') AS donem FROM islem;
SELECT to_date('13.10.2026', 'DD.MM.YYYY');
SELECT (date_trunc('month', current_date) - interval '1 day')::date AS onceki_ay_sonu;
SELECT to_timestamp(1791873000);                        -- epoch → timestamptz

-- Oracle
SELECT SYSTIMESTAMP, TO_CHAR(SYSDATE, 'YYYY-MM') FROM dual;
SELECT LAST_DAY(ADD_MONTHS(TRUNC(SYSDATE), -1)) FROM dual;   -- önceki ay sonu
SELECT TO_DATE('13.10.2026', 'DD.MM.YYYY') FROM dual;
```
PostgreSQL'de zaman damgaları için `timestamptz` (saat dilimli) tipini tercih edin.

## Komut satırı
```bash
date                                   # yerel saat
date -u                                # UTC
date +%Y-%m-%d                         # 2026-10-13
date -d "yesterday" +%F                # dün (GNU date)
date -d "$(date +%Y-%m-01) -1 day" +%F # önceki ay sonu
date +%s                               # şimdiki epoch
date -d @1791873000                    # epoch → okunur tarih
TZ=Europe/Istanbul date                # başka dilimde göster
timedatectl                            # sunucu saat dilimi ve NTP durumu
```
