# 5.6 Job ve CronJob: Zamanlanmış İşler (ETL)

## Nedir?
- **Job:** Bir görevi **çalıştırıp bitene kadar** takip eden nesne. Pod başarıyla tamamlanınca Job da tamamlanır.
- **CronJob:** Belirli bir takvime göre (cron söz dizimiyle) düzenli olarak Job oluşturan nesne.

Deployment "sürekli çalış" der; Job "işini yap ve bitir" der.

## Hangi problemi çözer?
- ETL, rapor üretimi, model eğitimi, yedekleme gibi işler sürekli çalışmaz, bir süre çalışıp biter.
- Linux cron'u ([1.8](../01-linux/08-cron.md)) tek sunucuya bağlıdır; sunucu kapanırsa iş çalışmaz. CronJob ise cluster'daki uygun herhangi bir node'da çalışır.
- Başarısız olan iş otomatik olarak yeniden denenir.
- Her çalışma izole bir container'da, aynı image ile ve aynı ortamda koşar.

## Nasıl çalışır?

```mermaid
flowchart LR
    CJ[CronJob<br/>schedule: 0 7 * * 1-5] -->|her sabah| J1[Job<br/>bddk-etl-28812345]
    J1 --> P1[Pod<br/>çalış ve bitir]
    P1 -->|başarısız| P2[Pod<br/>yeniden dene]
    P2 -->|başarılı| OK[Completed]
```

**Önemli ayarlar**

| Ayar | Anlamı |
|---|---|
| `schedule` | Cron ifadesi (`0 7 * * 1-5`) |
| `timeZone` | Saat dilimi (ör. `Europe/Istanbul`); verilmezse cluster saati (çoğunlukla UTC) |
| `concurrencyPolicy` | Önceki çalışma sürerken yenisi gelirse: `Allow`, `Forbid` (atla), `Replace` |
| `backoffLimit` | Başarısız olursa kaç kez yeniden denensin |
| `activeDeadlineSeconds` | Maksimum çalışma süresi; aşılırsa iş durdurulur |
| `successfulJobsHistoryLimit` / `failedJobsHistoryLimit` | Kaç eski çalışma saklansın |
| `restartPolicy` | Job'larda `OnFailure` veya `Never` olmalı |
| `completions` / `parallelism` | Kaç kez ve aynı anda kaç pod (paralel işler için) |

## Mini örnek
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: bddk-kredi-etl
  namespace: veri-prod
spec:
  schedule: "0 7 * * 1-5"
  timeZone: "Europe/Istanbul"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 3600
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: etl
              image: registry.sirket.local/veri/bddk-etl:2.1.0
              args: ["python", "-m", "etl.kredi"]
              envFrom:
                - configMapRef: { name: etl-config }
                - secretRef:    { name: etl-db-secret }
              resources:
                requests: { cpu: "500m", memory: "1Gi" }
                limits:   { memory: "2Gi" }
```

```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs                                      # geçmiş çalışmalar
kubectl create job --from=cronjob/bddk-kredi-etl elle-calistir   # takvimi beklemeden çalıştır
kubectl logs job/elle-calistir                         # çıktı
kubectl patch cronjob bddk-kredi-etl -p '{"spec":{"suspend":true}}'   # geçici durdur
```

## Nerede kullanılır?
- Günlük/aylık ETL'ler (BDDK, TÜİK, iç sistem verileri)
- Rapor üretimi ve e-posta gönderimi
- Model yeniden eğitimi ve batch skorlama
- Veritabanı yedekleri, eski veri temizliği

## Benzerleri ve farkları
| Araç | Güçlü yanı | Zayıf yanı |
|---|---|---|
| **Linux cron** | Basit | Tek sunucu, izleme yok |
| **Kubernetes CronJob** | İzole, yeniden deneme, cluster genelinde | Adımlar arası bağımlılık yok |
| **Airflow / Argo Workflows** | Bağımlı adımlar (DAG), geçmiş, arayüz, geri doldurma (backfill) | Kurulum ve bakım yükü |

Kural: Tek adımlı, bağımsız işler → CronJob. "A bitince B ve C, ikisi bitince D" gibi bağımlılıklar → iş akışı orkestratörü (Bölüm 11).

**Dikkat:**
- İşin **idempotent** olması (iki kez çalışsa da sonucun aynı olması) önemlidir; yeniden denemeler aynı veriyi iki kez yükleyebilir.
- `timeZone` vermezseniz 07:00, Türkiye saatiyle 10:00 olabilir.

## İlgili sayfalar
- [1.8 cron](../01-linux/08-cron.md)
- [5.9 ConfigMap ve Secret](09-configmap-secret.md)
