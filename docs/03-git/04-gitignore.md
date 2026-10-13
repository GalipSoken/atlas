# 3.4 .gitignore

## Nedir?
`.gitignore`, Git'in **takip etmemesi** gereken dosya ve klasörlerin listelendiği metin dosyasıdır. Reponun kök dizinine konur.

## Hangi problemi çözer?
Her dosya Git'e girmemelidir:
- **Şifreler ve anahtarlar** (`.env`, `credentials.json`): Bir kez push edilirse geçmişte kalır, silmek zordur ve sızmış sayılmalıdır.
- **Üretilen dosyalar** (`__pycache__/`, `*.pyc`, `dist/`): Koddan yeniden oluşturulabilir.
- **Sanal ortamlar** (`.venv/`): Büyük ve makineye özeldir.
- **Veri dosyaları** (`*.csv`, `*.parquet`): Büyük ve çoğu zaman gizlilik içerir (müşteri verisi asla repoya girmemelidir).
- **Kişisel editör ayarları** (`.idea/`, `.DS_Store`).

## Nasıl çalışır?
- Her satır bir desendir (pattern). Git, `git status` ve `git add` sırasında bu desenlere uyan dosyaları görmezden gelir.
- **Zaten takip edilen** bir dosya `.gitignore`'a eklenince takipten çıkmaz; önce `git rm --cached` ile çıkarılması gerekir.

| Desen | Anlamı |
|---|---|
| `.env` | Her yerdeki `.env` dosyası |
| `/config.local.yaml` | Sadece kök dizindeki |
| `logs/` | `logs` adlı tüm klasörler |
| `*.csv` | Tüm csv dosyaları |
| `data/**/*.parquet` | `data` altındaki tüm parquet'ler |
| `!ornek.csv` | İstisna: bunu takip et |
| `# yorum` | Açıklama satırı |

## Mini örnek
Python veri projesi için örnek `.gitignore`:
```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/

# Ortamlar
.venv/
venv/
.conda/

# Gizli bilgiler
.env
*.pem
*.key
secrets/

# Veri ve çıktılar
data/
*.csv
*.parquet
*.xlsx
!tests/fixtures/*.csv
outputs/
mlruns/

# Jupyter
.ipynb_checkpoints/

# Editör ve işletim sistemi
.vscode/
.idea/
.DS_Store
Thumbs.db

# Loglar
*.log
logs/
```

Yanlışlıkla eklenen dosyayı takipten çıkarmak:
```bash
echo ".env" >> .gitignore
git rm --cached .env          # dosya diskte kalır, Git takibi bırakır
git commit -m ".env takipten çıkarıldı"
```

Bir dosyanın neden görmezden gelindiğini bulmak:
```bash
git check-ignore -v data/kredi.csv
```

## Nerede kullanılır?
- Her reponun ilk commit'inde oluşturulmalı
- Hazır şablonlar: [github.com/github/gitignore](https://github.com/github/gitignore) (Python, Node, Java...)
- `.dockerignore` aynı mantıkla Docker build'ine girmeyecek dosyaları belirler ([4.3](../04-docker/03-multi-stage-build.md))

## Benzerleri ve farkları
- **`.git/info/exclude`:** Sadece sizin makinenizde geçerli, paylaşılmayan ignore listesi.
- **Global ignore** (`git config --global core.excludesfile ~/.gitignore_global`): Tüm repolarınız için kişisel desenler (ör. `.DS_Store`).

**Şifre yanlışlıkla push edildiyse:** `.gitignore`'a eklemek yetmez, geçmişte durur. Şifreyi **hemen değiştirin** (rotate). Geçmişten temizlemek için `git filter-repo` gibi araçlar vardır ama şifre sızmış kabul edilmelidir.

## İlgili sayfalar
- [1.5 Environment variable'lar](../01-linux/05-environment.md)
- [3.3 Git komutları](03-git-komutlari.md)
