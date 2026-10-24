# Atlas

Yazılım geliştirme, DevOps, veri depolama, veri işleme, BI, MLOps, güvenlik ve dosya formatı kavramlarını sade bir dille anlatan Türkçe bilgi kaynağı.

Bölümler, yazılan kodun **saklanmasından canlıda çalışmasına** ve verinin **kaynaktan rapora** ulaşmasına kadar olan yolu sırasıyla takip eder. Her bölüm bir öncekinin üzerine kurulur.

---

## Sayfa şablonu

Her konu sayfası aynı yapıyı izler:

1. **Nedir?** — 1–2 cümlelik tanım
2. **Hangi problemi çözer?**
3. **Nasıl çalışır?** — temel kavramlar + diyagram
4. **Mini örnek** — komut veya kod
5. **Nerede kullanılır?** — bankacılık / veri işleri örnekleri
6. **Benzerleri ve farkları**
7. **İlgili sayfalar**

---

## İçindekiler

### Kısım I — Temeller

#### 0. Büyük Resim
- [0.1 Koddan canlıya: uçtan uca akış](docs/00-buyuk-resim/01-koddan-canliya.md)
- [0.2 Veriden rapora: uçtan uca akış](docs/00-buyuk-resim/02-veriden-rapora.md)
- [0.3 Sözlük](docs/00-buyuk-resim/03-sozluk.md)

#### 1. Linux ve Terminal
- [1.1 Linux nedir, neden sunucuların çoğu Linux](docs/01-linux/01-linux-nedir.md)
- [1.2 Dosya sistemi ve temel komutlar](docs/01-linux/02-temel-komutlar.md) — `ls`, `cd`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `cat`, `less`, `head`, `tail`, `grep`, `find`
- [1.3 Kullanıcılar, gruplar ve dosya izinleri](docs/01-linux/03-izinler.md) — `chmod`, `chown`, `sudo`
- [1.4 Süreçler ve servisler](docs/01-linux/04-surecler.md) — `ps`, `top`, `kill`, `systemctl`
- [1.5 Environment variable'lar ve PATH](docs/01-linux/05-environment.md)
- [1.6 SSH ve uzak sunucuya bağlanma](docs/01-linux/06-ssh.md) — anahtar çifti, `scp`
- [1.7 Pipe, yönlendirme ve bash script](docs/01-linux/07-bash-script.md)
- [1.8 cron ile zamanlama](docs/01-linux/08-cron.md)
- [1.9 Ağ temelleri: DNS, port, firewall, proxy, load balancer](docs/01-linux/09-ag-temelleri.md)
- [1.10 localhost, 127.0.0.1, 0.0.0.0 ve port dinleme](docs/01-linux/10-localhost.md)
- [1.11 Sanal makine (VM) ve hypervisor](docs/01-linux/11-sanal-makine.md)

#### 2. Python Ortam Yönetimi
- [2.1 Neden sanal ortam gerekir](docs/02-python-ortam/01-neden-sanal-ortam.md)
- [2.2 venv ve pip](docs/02-python-ortam/02-venv-pip.md)
- [2.3 conda](docs/02-python-ortam/03-conda.md)
- [2.4 uv ve poetry](docs/02-python-ortam/04-uv-poetry.md)
- [2.5 requirements.txt, pyproject.toml ve sürüm sabitleme](docs/02-python-ortam/05-bagimlilik-dosyalari.md)
- [2.6 Kurum içi paket deposu ve proxy](docs/02-python-ortam/06-kurum-ici-paket.md)

### Kısım II — Koddan Canlıya

#### 3. Versiyon Kontrol — Git
- [3.1 Git nedir, neden kullanılır](docs/03-git/01-git-nedir.md)
- [3.2 Temel kavramlar: repo, commit, branch, merge](docs/03-git/02-temel-kavramlar.md)
- [3.3 Git komutları](docs/03-git/03-git-komutlari.md) — `clone`, `status`, `add`, `commit`, `push`, `pull`, `fetch`, `branch`, `switch`/`checkout`, `merge`, `rebase`, `stash`, `log`, `diff`, `reset`, `revert`, `tag`
- [3.4 .gitignore](docs/03-git/04-gitignore.md)
- [3.5 Merge conflict çözümü](docs/03-git/05-merge-conflict.md)
- [3.6 GitHub / GitLab, Pull Request ve code review](docs/03-git/06-github-pull-request.md)
- [3.7 Branch stratejileri: Git Flow ve trunk-based](docs/03-git/07-branch-stratejileri.md)

#### 4. Container'lar — Docker
- [4.1 Docker nedir: image ve container](docs/04-docker/01-docker-nedir.md)
- [4.2 Dockerfile yazmak, image katmanları](docs/04-docker/02-dockerfile.md)
- [4.3 Multi-stage build ve .dockerignore](docs/04-docker/03-multi-stage-build.md)
- [4.4 Docker komutları](docs/04-docker/04-docker-komutlari.md) — `build`, `run`, `ps`, `logs`, `exec`, `stop`, `rm`, `images`, `pull`, `push`
- [4.5 Volume ve bind mount: kalıcı veri](docs/04-docker/05-volume.md)
- [4.6 Docker network](docs/04-docker/06-network.md)
- [4.7 Docker Hub ve registry kavramı](docs/04-docker/07-registry-docker-hub.md)
- [4.8 Docker Compose: çoklu container](docs/04-docker/08-docker-compose.md)

#### 5. Orkestrasyon — Kubernetes ve OpenShift
- [5.1 Kubernetes nedir, neden gerekir](docs/05-kubernetes/01-kubernetes-nedir.md)
- [5.2 Mimari: cluster, node, control plane](docs/05-kubernetes/02-mimari.md)
- [5.3 Namespace ve Pod](docs/05-kubernetes/03-namespace-pod.md)
- [5.4 Deployment ve ReplicaSet](docs/05-kubernetes/04-deployment.md)
- [5.5 StatefulSet ve DaemonSet](docs/05-kubernetes/05-statefulset-daemonset.md)
- [5.6 Job ve CronJob: zamanlanmış işler (ETL)](docs/05-kubernetes/06-job-cronjob.md)
- [5.7 Service türleri: ClusterIP, NodePort, LoadBalancer](docs/05-kubernetes/07-service.md)
- [5.8 Ingress](docs/05-kubernetes/08-ingress.md)
- [5.9 ConfigMap ve Secret](docs/05-kubernetes/09-configmap-secret.md)
- [5.10 PersistentVolume, PVC ve StorageClass](docs/05-kubernetes/10-persistent-volume.md)
- [5.11 Liveness / readiness probe ve resource request/limit](docs/05-kubernetes/11-probe-resources.md)
- [5.12 HPA: otomatik ölçekleme](docs/05-kubernetes/12-hpa.md)
- [5.13 RBAC: yetkilendirme](docs/05-kubernetes/13-rbac.md)
- [5.14 kubectl komutları](docs/05-kubernetes/14-kubectl-komutlari.md)
- [5.15 Helm](docs/05-kubernetes/15-helm.md)
- [5.16 OpenShift: Kubernetes'ten farkları](docs/05-kubernetes/16-openshift-nedir.md)
- [5.17 OpenShift kavramları: Project, Route, BuildConfig, ImageStream, SCC](docs/05-kubernetes/17-openshift-kavramlari.md)
- [5.18 oc komutları](docs/05-kubernetes/18-oc-komutlari.md)

#### 6. CI/CD
- [6.1 CI ve CD nedir](docs/06-ci-cd/01-ci-cd-nedir.md)
- [6.2 Pipeline aşamaları: test, build, deploy](docs/06-ci-cd/02-pipeline-asamalari.md)
- [6.3 Otomatik testler: pytest, unit ve entegrasyon testi](docs/06-ci-cd/03-otomatik-test.md)
- [6.4 Araçlar: GitHub Actions, GitLab CI, Jenkins, OpenShift Pipelines](docs/06-ci-cd/04-araclar.md)
- [6.5 GitOps ve ArgoCD](docs/06-ci-cd/05-gitops-argocd.md)

#### 7. API'ler
- [7.1 API nedir, client–server modeli](docs/07-api/01-api-nedir.md)
- [7.2 HTTP: metotlar, durum kodları, header'lar](docs/07-api/02-http.md)
- [7.3 REST ve JSON](docs/07-api/03-rest-json.md)
- [7.4 FastAPI ile API yazmak](docs/07-api/04-fastapi.md)
- [7.5 API güvenliği: API key, token, rate limit](docs/07-api/05-api-guvenligi.md)
- [7.6 REST dışı yaklaşımlar: GraphQL, gRPC, webhook](docs/07-api/06-diger-yaklasimlar.md)

#### 8. Web Sunucuları, Reverse Proxy ve Kenar Ağı — nginx, Cloudflare
- [8.1 Web sunucusu nedir: statik içerik ve uygulama sunucusu farkı](docs/08-nginx/01-web-sunucusu-nedir.md)
- [8.2 nginx nedir, nerede durur](docs/08-nginx/02-nginx-nedir.md)
- [8.3 Konfigürasyon yapısı: server, location, upstream](docs/08-nginx/03-konfigurasyon.md)
- [8.4 Reverse proxy: uygulamanın önüne nginx koymak](docs/08-nginx/04-reverse-proxy.md)
- [8.5 Load balancing](docs/08-nginx/05-load-balancing.md)
- [8.6 HTTPS ve TLS sertifikaları](docs/08-nginx/06-https-tls.md)
- [8.7 Cache, gzip ve rate limit](docs/08-nginx/07-cache-rate-limit.md)
- [8.8 Loglar ve sık hatalar: 502, 504, 413](docs/08-nginx/08-log-hatalar.md)
- [8.9 Kubernetes'te nginx: Ingress Controller](docs/08-nginx/09-ingress-controller.md)

**Kenar ağı: Cloudflare**
- [8.10 CDN ve kenar ağı: Cloudflare nedir](docs/08-nginx/10-cdn-edge-cloudflare.md) — proxied/DNS only, önbellek, 52x hataları
- [8.11 Cloudflare DNS: kayıtlar, SPF, DKIM, DMARC](docs/08-nginx/11-cloudflare-dns.md)
- [8.12 Cloudflare'de SSL/TLS: Flexible, Full, Full (strict)](docs/08-nginx/12-cloudflare-ssl-tls.md)
- [8.13 WAF, DDoS ve bot koruması](docs/08-nginx/13-waf-ddos-bot.md) — custom rules, rate limiting, Turnstile
- [8.14 Cloudflare Tunnel ve Zero Trust erişim](docs/08-nginx/14-tunnel-zero-trust.md)

### Kısım III — Veri

#### 9. Veri Depolama
- [9.1 OLTP ve OLAP farkı](docs/09-veri-depolama/01-oltp-olap.md)
- [9.2 İlişkisel veritabanları, SQL ve ACID](docs/09-veri-depolama/02-iliskisel-sql-acid.md)
- [9.3 PostgreSQL ve Oracle karşılaştırması](docs/09-veri-depolama/03-postgres-oracle.md)
- [9.4 Veritabanı performansı: index, partitioning, EXPLAIN](docs/09-veri-depolama/04-performans.md)
- [9.5 NoSQL türleri: document, key-value, columnar, graph](docs/09-veri-depolama/05-nosql-turleri.md)
- [9.6 MongoDB](docs/09-veri-depolama/06-mongodb.md)
- [9.7 Redis ve cache kavramı](docs/09-veri-depolama/07-redis-cache.md)
- [9.8 Graph veritabanları ve Neo4j](docs/09-veri-depolama/08-graph-db.md)
- [9.9 Vector veritabanları ve pgvector](docs/09-veri-depolama/09-vector-db.md)
- [9.10 CAP teoremi](docs/09-veri-depolama/10-cap-teoremi.md)
- [9.11 Schema-on-write ve schema-on-read](docs/09-veri-depolama/11-schema-on-write-read.md)
- [9.12 Data warehouse, data lake ve lakehouse](docs/09-veri-depolama/12-warehouse-lake-lakehouse.md)
- [9.13 Medallion mimarisi: bronze, silver, gold](docs/09-veri-depolama/13-medallion.md)
- [9.14 Delta Lake ve Apache Iceberg](docs/09-veri-depolama/14-delta-iceberg.md)
- [9.15 Nesne depolama: S3 ve MinIO](docs/09-veri-depolama/15-nesne-depolama.md)

**SQL derinlemesine**
- [9.16 Transaction ve isolation seviyeleri](docs/09-veri-depolama/16-transaction-isolation.md) — MVCC, dirty/phantom read, savepoint
- [9.17 Lock ve deadlock](docs/09-veri-depolama/17-lock-deadlock.md) — `FOR UPDATE`, `SKIP LOCKED`, `lock_timeout`
- [9.18 Index derinlemesine: doğru index, kullanılmayan index](docs/09-veri-depolama/18-index-derinlemesine.md)
- [9.19 Execution plan (sorgu planı) okumak](docs/09-veri-depolama/19-execution-plan.md) — `EXPLAIN ANALYZE`, `DBMS_XPLAN`
- [9.20 Join türleri ve join stratejileri](docs/09-veri-depolama/20-join-stratejileri.md) — nested loop, hash, merge
- [9.21 Hint'ler: optimizer'a yön vermek](docs/09-veri-depolama/21-hint.md)
- [9.22 CTE ve window functions](docs/09-veri-depolama/22-cte-window.md) — `ROW_NUMBER`, `LAG`, recursive CTE
- [9.23 Partitioning derinlemesine](docs/09-veri-depolama/23-partitioning-derinlemesine.md) — pruning, local/global index, exchange
- [9.24 Connection pooling](docs/09-veri-depolama/24-connection-pooling.md) — PgBouncer, SQLAlchemy pool
- [9.25 Stored procedure, function, trigger ve view](docs/09-veri-depolama/25-procedure-function-trigger.md)
- [9.26 Query timeout ve kaynak sınırları](docs/09-veri-depolama/26-query-timeout.md)

#### 10. Veri İşleme Araçları
- [10.1 SQL: veri veritabanında işlenir](docs/10-veri-isleme/01-sql.md)
- [10.2 Pandas: tek makine, bellekte işlem](docs/10-veri-isleme/02-pandas.md)
- [10.3 Spark: dağıtık işlem ve lazy evaluation](docs/10-veri-isleme/03-spark.md)
- [10.4 SQL, Pandas ve Spark karşılaştırması](docs/10-veri-isleme/04-karsilastirma.md) — aynı sorgunun üç araçta yazılışı
- [10.5 Ara seçenekler: Polars ve DuckDB](docs/10-veri-isleme/05-polars-duckdb.md)
- [10.6 Dağıtık işleme: Apache Spark ve Dask](docs/10-veri-isleme/06-spark-dask.md)

#### 11. Veri Akışı
- [11.1 ETL ve ELT: diyagramla akış](docs/11-veri-akisi/01-etl-elt.md) — kaynak (transaction DB) → warehouse / lakehouse → rapor / HQ
- [11.2 Schema-on-write ve schema-on-read'in akıştaki yeri](docs/11-veri-akisi/02-akista-schema.md)
- [11.3 Yükleme türleri: full, incremental ve CDC](docs/11-veri-akisi/03-yukleme-turleri.md)
- [11.4 dbt: SQL ile dönüşüm katmanı](docs/11-veri-akisi/04-dbt.md)
- [11.5 Veri kalitesi ve testleri: Great Expectations, dbt test](docs/11-veri-akisi/05-veri-kalitesi.md)
- [11.6 Batch ve streaming farkı](docs/11-veri-akisi/06-batch-streaming.md)
- [11.7 Mesaj kuyrukları ve Kafka](docs/11-veri-akisi/07-kafka.md)
- [11.8 Zamanlama ve orkestrasyon: cron, Airflow, n8n](docs/11-veri-akisi/08-orkestrasyon.md)
- [11.9 ETL/ELT konnektörleri: Airbyte, Meltano, dlt](docs/11-veri-akisi/09-etl-konnektorleri.md)
- [11.10 Workflow orkestrasyonu: Airflow, Dagster, Prefect](docs/11-veri-akisi/10-orkestrator-karsilastirma.md) — aynı iş üç araçta
- [11.11 Streaming platformları: Kafka ve Redpanda, teslim garantileri, akış işleme](docs/11-veri-akisi/11-kafka-redpanda.md)

#### 12. BI ve Görselleştirme
- [12.1 BI nedir](docs/12-bi/01-bi-nedir.md)
- [12.2 BI araçları: Power BI, Tableau, Qlik, Superset](docs/12-bi/02-bi-araclari.md)
- [12.3 Grafana: BI'dan farkı](docs/12-bi/03-grafana.md)
- [12.4 Dashboard tasarım prensipleri](docs/12-bi/04-dashboard-tasarimi.md)
- [12.5 Streamlit ve hazır BI araçları karşılaştırması](docs/12-bi/05-streamlit.md)

### Kısım IV — Model ve Operasyon

#### 13. MLOps
- [13.1 ML yaşam döngüsü](docs/13-mlops/01-ml-yasam-dongusu.md)
- [13.2 Experiment tracking ve model registry (MLflow)](docs/13-mlops/02-mlflow.md)
- [13.3 Model serving: batch ve online](docs/13-mlops/03-model-serving.md)
- [13.4 Model monitoring ve drift](docs/13-mlops/04-monitoring-drift.md)
- [13.5 Feature ve feature engineering kavramı](docs/13-mlops/05-feature-kavrami.md)
- [13.6 Feature store nedir: hangi problemi çözer (training–serving tutarlılığı, tekrar kullanım)](docs/13-mlops/06-feature-store-nedir.md)
- [13.7 Feature store bileşenleri: registry, dönüşüm, materialization, serving](docs/13-mlops/07-feature-store-bilesenleri.md)
- [13.8 Offline ve online store](docs/13-mlops/08-offline-online-store.md)
- [13.9 Point-in-time doğruluk ve veri sızıntısı (leakage)](docs/13-mlops/09-point-in-time.md)
- [13.10 Plugin / eklenti mimarisi: platforma yeni yetenek eklemek](docs/13-mlops/10-plugin-mimarisi.md) — feature store'un bir ML platformuna plugin olarak eklenmesi örneğiyle
- [13.11 Uçtan uca ML platformları: bileşenler nasıl bir araya gelir](docs/13-mlops/11-ml-platformlari.md)
- [13.12 GenAI altyapısı: embedding, RAG ve LLMOps](docs/13-mlops/12-genai-rag.md)

#### 14. Gözlemlenebilirlik
- [14.1 Log, metrik ve trace](docs/14-gozlemlenebilirlik/01-log-metrik-trace.md)
- [14.2 Prometheus ve Grafana](docs/14-gozlemlenebilirlik/02-prometheus-grafana.md)
- [14.3 Merkezi loglama: ELK / OpenSearch](docs/14-gozlemlenebilirlik/03-merkezi-loglama.md)
- [14.4 Alarm ve uyarı kurgusu](docs/14-gozlemlenebilirlik/04-alarm.md)

### Kısım V — Kurumsal Katman

#### 15. Kimlik ve Güvenlik
- [15.1 Kimlik doğrulama ve yetkilendirme farkı (authN / authZ)](docs/15-guvenlik/01-authn-authz.md)
- [15.2 LDAP nedir: dizin yapısı, DN, OU, bind](docs/15-guvenlik/02-ldap.md)
- [15.3 Active Directory ve LDAP ilişkisi](docs/15-guvenlik/03-active-directory.md)
- [15.4 LDAP entegrasyonu: uygulamalarda, OpenShift'te, BI araçlarında](docs/15-guvenlik/04-ldap-entegrasyonu.md)
- [15.5 SSO, Kerberos, OAuth 2.0 ve OpenID Connect](docs/15-guvenlik/05-sso-oauth-oidc.md)
- [15.6 Rol bazlı erişim (RBAC) ve en az yetki prensibi](docs/15-guvenlik/06-rbac-en-az-yetki.md)
- [15.7 Secret yönetimi: Vault, şifrelerin koddan ayrılması](docs/15-guvenlik/07-secret-yonetimi.md)
- [15.8 Şifreleme: TLS, at-rest ve in-transit](docs/15-guvenlik/08-sifreleme.md)
- [15.9 Veri maskeleme, anonimleştirme ve KVKK](docs/15-guvenlik/09-maskeleme-kvkk.md)

#### 16. Veri Yönetişimi ve Kalite
- [16.1 Veri yönetişimi nedir: çerçeve, roller, politikalar, BCBS 239](docs/16-veri-yonetisimi/01-veri-yonetisimi-nedir.md)
- [16.2 Veri kataloğu ve metadata yönetimi, iş sözlüğü](docs/16-veri-yonetisimi/02-katalog-metadata.md)
- [16.3 Data lineage ve etki analizi](docs/16-veri-yonetisimi/03-lineage.md)
- [16.4 OpenLineage ve DataHub: lineage ve metadata'yı otomatik toplamak](docs/16-veri-yonetisimi/04-openlineage-datahub.md)
- [16.5 Veri sahipliği, data contract ve data mesh](docs/16-veri-yonetisimi/05-sahiplik-data-contract.md)
- [16.6 Veri sınıflandırma ve etiketleme](docs/16-veri-yonetisimi/06-siniflandirma.md)
- [16.7 Veriye erişim politikaları: RBAC, ABAC, satır ve sütun güvenliği](docs/16-veri-yonetisimi/07-erisim-politikalari.md)
- [16.8 Veri kalitesi yönetimi: boyutlar, kurallar, eşikler, sorun süreci](docs/16-veri-yonetisimi/08-kalite-yonetimi.md)
- [16.9 Veri kalitesi araçları: Great Expectations ve Soda](docs/16-veri-yonetisimi/09-kalite-araclari.md)

### Kısım VI — Başvuru

#### 17. Dosya ve Veri Formatları
- [17.1 Formatlara genel bakış: metin / ikili, satır / sütun, şemalı / şemasız](docs/17-formatlar/01-genel-bakis.md)
- [17.2 CSV](docs/17-formatlar/02-csv.md)
- [17.3 JSON ve JSON Lines](docs/17-formatlar/03-json.md)
- [17.4 BSON](docs/17-formatlar/04-bson.md)
- [17.5 Parquet](docs/17-formatlar/05-parquet.md)
- [17.6 YAML](docs/17-formatlar/06-yaml.md)
- [17.7 TOML](docs/17-formatlar/07-toml.md)
- [17.8 .env dosyaları](docs/17-formatlar/08-env.md)
- [17.9 Diğerleri: Avro, ORC, Arrow, XML, Excel](docs/17-formatlar/09-digerleri.md)
- [17.10 Markdown (.md)](docs/17-formatlar/10-markdown.md) — README, GFM, Mermaid, Pandoc, MkDocs
- [17.11 Karakter kodlaması: UTF-8, Windows-1254, Türkçe İ/ı](docs/17-formatlar/11-karakter-kodlamasi.md)
- [17.12 Sıkıştırma ve arşiv: zip, gzip, tar, zstd, snappy](docs/17-formatlar/12-sikistirma-arsiv.md)
- [17.13 Model ve notebook formatları: pickle, joblib, ONNX, PMML, .ipynb](docs/17-formatlar/13-model-notebook-formatlari.md)
- [17.14 Bankacılık ve finans formatları: sabit uzunluklu dosya, EBCDIC, ISO 20022, SWIFT MT, XBRL](docs/17-formatlar/14-finans-formatlari.md)

#### Ekler — Cheat Sheet'ler
- [Linux / bash](docs/ekler/linux-bash-cheatsheet.md)
- [Git](docs/ekler/git-cheatsheet.md)
- [Docker](docs/ekler/docker-cheatsheet.md)
- [kubectl / oc](docs/ekler/kubectl-oc-cheatsheet.md)
- [nginx](docs/ekler/nginx-cheatsheet.md)
- [SQL / psql](docs/ekler/sql-psql-cheatsheet.md)
- [Pandas / PySpark](docs/ekler/pandas-pyspark-cheatsheet.md)
- [LDAP (ldapsearch)](docs/ekler/ldap-cheatsheet.md)
- [Regex (düzenli ifadeler)](docs/ekler/regex-cheatsheet.md)
- [vim / nano](docs/ekler/vim-nano-cheatsheet.md)
- [cron ve tarih/zaman formatları](docs/ekler/cron-zaman-cheatsheet.md)

---

## Repo yapısı

```
atlas/
├── README.md               # bu index
├── docs/
│   ├── 00-buyuk-resim/
│   ├── 01-linux/
│   ├── 02-python-ortam/
│   ├── 03-git/
│   ├── 04-docker/
│   ├── 05-kubernetes/
│   ├── 06-ci-cd/
│   ├── 07-api/
│   ├── 08-nginx/
│   ├── 09-veri-depolama/
│   ├── 10-veri-isleme/
│   ├── 11-veri-akisi/
│   ├── 12-bi/
│   ├── 13-mlops/
│   ├── 14-gozlemlenebilirlik/
│   ├── 15-guvenlik/
│   ├── 16-veri-yonetisimi/
│   ├── 17-formatlar/
│   └── ekler/
└── assets/                 # diyagramlar ve görseller
```
