# 17.6 YAML

## Nedir?
**YAML ("YAML Ain't Markup Language")**, insan tarafından kolay okunup yazılmak üzere tasarlanmış, **girintiye (indentation) dayalı** hiyerarşik veri formatıdır. Dosya uzantıları `.yaml` veya `.yml`.

Kitapta en çok karşılaşılan konfigürasyon formatıdır: Kubernetes ve OpenShift kaynakları ([5](../05-kubernetes/01-kubernetes-nedir.md)), Docker Compose ([4.8](../04-docker/08-docker-compose.md)), GitHub Actions ve GitLab CI ([6](../06-ci-cd/01-ci-cd-nedir.md)), Ansible, Helm, dbt, Prometheus kuralları.

## Hangi problemi çözer?
- JSON'a göre **daha okunaklı**: süslü parantez ve tırnak kalabalığı yok.
- **Yorum satırı** destekler (`#`); konfigürasyonu belgelemek mümkün.
- Çok satırlı metinleri (SQL, script) doğal biçimde yazmaya izin verir.
- JSON'un üst kümesidir: Geçerli JSON genellikle geçerli YAML'dır.

## Nasıl çalışır?

**Temel yapılar**
```yaml
# Yorum satırı
uygulama: kredi-api            # anahtar: değer (map / sözlük)
surum: 1.4                     # sayı
aktif: true                    # mantıksal
aciklama: null                 # boş (~ da kullanılabilir)

veritabani:                    # iç içe sözlük (2 boşluk girinti)
  host: pg.sirket.local
  port: 5432

segmentler:                    # liste
  - Bireysel
  - KOBİ
  - Ticari

kisa_liste: [Bireysel, KOBİ]   # akış (flow) stili, JSON benzeri
kisa_sozluk: {host: pg, port: 5432}

kaynaklar:                     # sözlüklerden oluşan liste
  - ad: kredi
    zamanlama: "0 7 * * 1-5"
  - ad: mevduat
    zamanlama: "30 7 * * 1-5"
```

**Çok satırlı metin**
```yaml
sorgu_korunur: |               # | : satır sonları KORUNUR (SQL, script için)
  SELECT donem, SUM(tutar)
  FROM gold.kredi
  GROUP BY donem;

aciklama_katlanir: >           # > : satırlar tek satırda BİRLEŞTİRİLİR (uzun düz metin)
  Bu açıklama editörde birden çok satıra
  bölünmüş olsa da tek satır olarak okunur.
```

**Birden çok belge (`---`)**: Tek dosyada birden çok Kubernetes kaynağı:
```yaml
apiVersion: v1
kind: Service
metadata: {name: kredi-api}
---
apiVersion: apps/v1
kind: Deployment
metadata: {name: kredi-api}
```

**Çapa ve takma ad (anchor & alias)**: Tekrarı önlemek için
```yaml
varsayilan: &varsayilan
  retries: 3
  timeout: 60

kredi_etl:
  <<: *varsayilan              # varsayılanları birleştir
  timeout: 120                 # birini ez
```

**YAML tuzakları**
| Tuzak | Örnek | Sonuç / çözüm |
|---|---|---|
| **Sekme (tab) karakteri** | Girintide tab | Hata; sadece boşluk kullanın |
| **Girinti hatası** | Bir boşluk kaydırma | Yapı tamamen değişir; editörde YAML desteği ve linter kullanın |
| **Norveç problemi (YAML 1.1)** | `ulke: NO`, `aktif: yes`, `on`, `off` | `false`/`true` olarak okunabilir; metinse tırnak içine alın: `"NO"` |
| **Sayıya dönüşen metinler** | `surum: 1.10` → `1.1`; `kod: 0123` → sekizlik/123 | Tırnak: `"1.10"`, `"0123"` |
| **Saat/süre ifadeleri** | `zaman: 22:30` bazı ayrıştırıcılarda sayıya dönüşebilir | Tırnak: `"22:30"` |
| **İki nokta ve özel karakterler** | `mesaj: Hata: bağlantı yok` | Tırnak: `"Hata: bağlantı yok"` |
| **Cron ifadeleri** | `schedule: */15 * * * *` | `*` ile başlayan değer alias sanılır ve hata verir; cron ifadelerini her zaman tırnak içine alın: `"*/15 * * * *"` |
| **Güvenlik** | Python'da `yaml.load` keyfi nesne üretebilir | Her zaman `yaml.safe_load` |

## Mini örnek
**Python (PyYAML)**
```python
import yaml

with open("configs/egitim.yaml", encoding="utf-8") as f:
    ayar = yaml.safe_load(f)                 # ASLA yaml.load(f) (güvensiz)

print(ayar["veritabani"]["host"])

with open("cikti.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(ayar, f, allow_unicode=True, sort_keys=False)
```

Pydantic ile doğrulanmış konfigürasyon:
```python
from pydantic import BaseModel

class Veritabani(BaseModel):
    host: str
    port: int = 5432

class Ayar(BaseModel):
    uygulama: str
    veritabani: Veritabani
    segmentler: list[str]

ayar = Ayar(**yaml.safe_load(open("config.yaml", encoding="utf-8")))   # tip hatası varsa açık hata
```

**Komut satırı araçları**
```bash
yq '.veritabani.host' config.yaml                 # alan oku (mikefarah/yq)
yq -i '.spec.replicas = 3' deployment.yaml        # yerinde değiştir
yq -o=json config.yaml                            # YAML → JSON
yamllint .                                        # biçim ve hata kontrolü
kubectl apply --dry-run=client -f deployment.yaml # Kubernetes YAML doğrulama
```

## Nerede kullanılır?
- Kubernetes/OpenShift manifestleri, Helm values dosyaları
- CI/CD pipeline tanımları (GitHub Actions, GitLab CI, Azure Pipelines)
- Docker Compose, Ansible playbook'ları, Prometheus/Alertmanager kuralları
- dbt proje ve model tanımları, uygulama konfigürasyonları, ML eğitim parametreleri
- OpenAPI şemaları, MkDocs gibi doküman araçları

## Benzerleri ve farkları
| Format | Fark |
|---|---|
| **JSON** | Daha katı ve makine dostu; yorum yok; API'ler için ([17.3](03-json.md)) |
| **TOML** | Daha az belirsizlik, düz/orta derinlikte konfigürasyon için; girintiye bağımlı değil ([17.7](07-toml.md)) |
| **INI** | Çok basit bölüm + anahtar=değer; iç içe yapı yok |
| **.env** | Sadece düz anahtar=değer; ortam değişkenleri ([17.8](08-env.md)) |

Kural: **İnsanların yazdığı, iç içe ve uzun konfigürasyonlar → YAML; programlar arası veri alışverişi → JSON; Python proje/araç ayarları → TOML.**

## İlgili sayfalar
- [5.14 kubectl](../05-kubernetes/14-kubectl-komutlari.md)
- [6.2 Pipeline aşamaları](../06-ci-cd/02-pipeline-asamalari.md)
- [17.7 TOML](07-toml.md)
