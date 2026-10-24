# 17.7 TOML

## Nedir?
**TOML (Tom's Obvious, Minimal Language)**, okunması ve yazılması kolay, **belirsizliği az** olacak şekilde tasarlanmış bir konfigürasyon formatıdır. Görünüşü INI dosyalarına benzer ama gerçek veri tipleri (tamsayı, ondalık, tarih-saat, dizi, tablo) ve iç içe yapı destekler.

Python dünyasında **`pyproject.toml`** dosyası sayesinde standart hâline gelmiştir ([2.5](../02-python-ortam/05-bagimlilik-dosyalari.md)). Rust (`Cargo.toml`), Hugo, Streamlit (`.streamlit/config.toml`, `secrets.toml`) ve birçok araç TOML kullanır.

## Hangi problemi çözer?
| Format | Sorun | TOML'un yaklaşımı |
|---|---|---|
| INI | Tip yok (her şey metin), iç içe yapı yok, standart yok | Açık tipler ve tablolar, resmî spesifikasyon |
| YAML | Girinti hataları, `NO` → `false` gibi örtük dönüşümler, karmaşık spesifikasyon | Girintiye bağlı değil; metinler her zaman tırnaklı, örtük tip dönüşümü yok |
| JSON | Yorum yok, insan için kalabalık | Yorum (`#`), sade yazım |

## Nasıl çalışır?

**Temel yapı**
```toml
# Yorum
baslik = "Kredi İzleme"          # metin (her zaman tırnaklı)
surum = 3                         # tamsayı
oran = 0.65                       # ondalık
aktif = true                      # mantıksal
baslangic = 2026-01-01            # tarih (yerel)
son_calisma = 2026-10-13T07:00:00+03:00   # tarih-saat (zaman dilimli)
segmentler = ["Bireysel", "KOBİ", "Ticari"]   # dizi
satir_ici = { host = "redis", port = 6379 }    # satır içi tablo

aciklama = """
Çok satırlı
metin
"""
yol = 'C:\veri\ham'               # tek tırnak: kaçış karakteri işlenmez (literal)
# Not: tablo başlıklarından ([...]) SONRA yazılan anahtarlar o tabloya ait olur;
# kök düzeydeki anahtarlar ilk tablo başlığından önce yazılmalıdır.

[veritabani]                      # tablo (bölüm) = iç içe sözlük
host = "pg.sirket.local"
port = 5432

[veritabani.havuz]                # iç içe tablo: veritabani.havuz
min = 2
max = 10

[[kaynak]]                        # tablolar dizisi (liste içindeki sözlükler)
ad = "kredi"
zamanlama = "0 7 * * 1-5"

[[kaynak]]
ad = "mevduat"
zamanlama = "30 7 * * 1-5"

```

Yukarıdakinin JSON karşılığı (özet):
```json
{
  "baslik": "Kredi İzleme",
  "veritabani": {"host": "pg.sirket.local", "port": 5432, "havuz": {"min": 2, "max": 10}},
  "kaynak": [{"ad": "kredi", "zamanlama": "0 7 * * 1-5"}, {"ad": "mevduat", "zamanlama": "30 7 * * 1-5"}]
}
```

**Kurallar**
- Anahtar tekrarı yasaktır (aynı tabloda iki kez `port` tanımlanamaz).
- Bir tablo bir kez tanımlanır.
- Dosya UTF-8 olmalıdır.
- Tarih-saat yerleşik bir tiptir (JSON ve YAML'dan farklı olarak belirsizlik yok).

## Mini örnek
**`pyproject.toml`** (Python projesinin kimlik kartı)
```toml
[project]
name = "kredi-izleme"
version = "0.3.0"
description = "BDDK kredi verisi ETL ve metrikleri"
requires-python = ">=3.12"
dependencies = [
    "pandas>=2.2,<3",
    "sqlalchemy>=2.0,<3",
    "psycopg[binary]>=3.2",
]

[project.scripts]
kredi-izleme = "kredi_izleme.main:calistir"      # komut satırı giriş noktası

[dependency-groups]
dev = ["pytest>=8", "ruff>=0.6"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# Araç ayarları da aynı dosyada
[tool.ruff]
line-length = 110

[tool.ruff.lint]
select = ["E", "F", "I", "B"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```

**Streamlit gizli bilgileri** (`.streamlit/secrets.toml`, Git'e girmez)
```toml
db_url = "postgresql+psycopg://..."

[ldap]
sunucu = "ldaps://dc01.sirket.local"
bind_dn = "CN=svc-app,OU=ServisHesaplari,DC=sirket,DC=local"
```
```python
import streamlit as st
st.secrets["db_url"]; st.secrets["ldap"]["sunucu"]
```

**Python'da okuma ve yazma**
```python
import tomllib                       # Python 3.11+ standart kütüphane (sadece okuma)

with open("pyproject.toml", "rb") as f:          # ikili modda açılır
    proje = tomllib.load(f)
print(proje["project"]["version"])

# Yazmak için üçüncü parti: tomli-w (basit) veya tomlkit (yorum ve biçimi korur)
import tomli_w
with open("ayar.toml", "wb") as f:
    tomli_w.dump({"veritabani": {"host": "pg", "port": 5432}}, f)
```

## Nerede kullanılır?
- Python projeleri: `pyproject.toml` (bağımlılıklar, paketleme, ruff/pytest/mypy ayarları), `uv.lock` da TOML tabanlıdır
- Streamlit yapılandırma ve secrets dosyaları
- Uygulama konfigürasyonları (orta derinlikte, insan tarafından düzenlenen ayarlar)
- Grafana'nın LDAP ayar dosyası `ldap.toml` ([15.4](../15-guvenlik/04-ldap-entegrasyonu.md))

## Benzerleri ve farkları
| | TOML | YAML | JSON | INI |
|---|---|---|---|---|
| Okunabilirlik | Yüksek | Yüksek | Orta | Yüksek |
| Yorum | Var | Var | Yok | Var |
| Tipler | Açık, tarih dahil | Örtük (tuzaklı) | Sınırlı | Yok |
| Derin iç içe yapı | Zahmetli (uzun bölüm adları) | Çok uygun | Uygun | Yok |
| Tipik kullanım | Proje ve araç ayarları | Kubernetes, CI/CD, Ansible | API, veri | Eski uygulama ayarları |

Kural: Derin ve uzun hiyerarşiler (Kubernetes manifestleri) için YAML; düz–orta derinlikteki proje ve araç ayarları için TOML.

## İlgili sayfalar
- [2.5 pyproject.toml ve bağımlılıklar](../02-python-ortam/05-bagimlilik-dosyalari.md)
- [17.6 YAML](06-yaml.md)
- [17.8 .env](08-env.md)
