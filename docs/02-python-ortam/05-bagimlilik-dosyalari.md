# 2.5 requirements.txt, pyproject.toml ve Sürüm Sabitleme

## Nedir?
Bir projenin hangi paketlere ve hangi sürümlere ihtiyaç duyduğunu yazan dosyalardır. Sanal ortamın kendisi Git'e gönderilmez; bu dosyalar gönderilir ve ortam her yerde bunlardan yeniden kurulur.

## Hangi problemi çözer?
Altı ay önce çalışan ETL bugün yeniden kurulduğunda, yeni gelen bir paket sürümü kodu bozabilir. Sürüm sabitleme (pinning) "dün çalışan bugün de çalışsın" garantisini verir.

## Nasıl çalışır?

**Sürüm ifadeleri**

| İfade | Anlamı | Ne zaman? |
|---|---|---|
| `pandas` | Herhangi bir sürüm | Kaçının |
| `pandas==2.2.2` | Tam olarak bu sürüm | Uygulama, kilit dosyası |
| `pandas>=2.0` | 2.0 veya üstü | Alt sınır |
| `pandas>=2.0,<3` | 2.x serisi | Kütüphane, `pyproject.toml` |
| `pandas~=2.2.0` | 2.2.x (yama sürümleri) | Ara çözüm |

**Semantik versiyonlama (SemVer):** `MAJOR.MINOR.PATCH` → `2.2.1`
- PATCH (2.2.**1**): hata düzeltmesi, güvenli
- MINOR (2.**2**.0): yeni özellik, geriye uyumlu olması beklenir
- MAJOR (**2**.0.0): kırıcı değişiklik olabilir

**Uygulama ve kütüphane farkı:**
- **Uygulama** (ETL, dashboard): Tam sürümleri sabitleyin, tekrarlanabilirlik önemli.
- **Kütüphane** (başkalarının kuracağı paket): Aralık verin, aksi hâlde kullanıcının diğer paketleriyle çakışır.

## Mini örnek

**`requirements.txt` (klasik yaklaşım)**
```text
pandas==2.2.2
sqlalchemy==2.0.31
psycopg2-binary==2.9.9
requests==2.32.3
```

İki dosyalı yaklaşım (pip-tools ile):
```text
# requirements.in  -> elle yazılır, sadece doğrudan bağımlılıklar
pandas>=2.2
sqlalchemy>=2.0
```
```bash
pip install pip-tools
pip-compile requirements.in      # -> requirements.txt (tüm ağaç, tam sürümler)
pip-sync requirements.txt        # ortamı birebir bu listeye eşitle
```

**`pyproject.toml` (modern standart)**
```toml
[project]
name = "bddk-etl"
version = "0.3.0"
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.2,<3",
    "sqlalchemy>=2.0,<3",
    "psycopg2-binary>=2.9",
]

[dependency-groups]
dev = ["pytest>=8", "ruff>=0.5"]
```
Kilit dosyası (`uv.lock`) bu dosyadan üretilir ([2.4](04-uv-poetry.md)).

## Nerede kullanılır?
- Her projenin kök dizininde
- Dockerfile içinde (`COPY requirements.txt .` → `RUN pip install -r requirements.txt`)
- CI pipeline'ında test ortamını kurarken
- Güvenlik taramalarında (Dependabot vb. bu dosyaları okuyarak açık bulunan paketleri bildirir)

## Benzerleri ve farkları
| Dosya | Kim yazar? | İçerik |
|---|---|---|
| `requirements.txt` | İnsan veya `pip freeze` | Paket listesi |
| `pyproject.toml` | İnsan | Proje tanımı + bağımlılık aralıkları |
| `uv.lock` / `poetry.lock` | Araç | Tüm ağacın tam sürümleri ve hash'leri |
| `environment.yml` | İnsan veya conda | conda ortamı |

## İlgili sayfalar
- [2.4 uv ve poetry](04-uv-poetry.md)
- [4.2 Dockerfile](../04-docker/02-dockerfile.md)
- [16.7 TOML](../16-formatlar/07-toml.md)
