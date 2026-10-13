# 1.5 Environment Variable'lar ve PATH

## Nedir?
**Environment variable (ortam değişkeni)**, bir süreç başlarken ona işletim sistemi tarafından verilen anahtar–değer ayarıdır. Örneğin `DB_HOST=10.0.0.5`.

**PATH** ise özel bir ortam değişkenidir: Bir komut yazdığımızda shell'in o komutu hangi klasörlerde arayacağını söyler.

## Hangi problemi çözer?
- **Ayarı koddan ayırmak:** Aynı kod dev ortamında test veritabanına, prod ortamında canlı veritabanına bağlanır; sadece değişken değişir.
- **Şifreleri koda yazmamak:** Şifre koda değil ortam değişkenine konur; kod Git'e gönderilse bile şifre sızmaz.
- **"command not found" hatası:** Çoğu zaman programın klasörü PATH'te olmadığı için çıkar.

## Nasıl çalışır?
- Değişken `export` ile tanımlanırsa o shell'den başlatılan tüm programlar onu görür.
- Terminal kapanınca kaybolur. Kalıcı olması için `~/.bashrc` (veya `~/.zshrc`) dosyasına yazılır.
- PATH, `:` ile ayrılmış klasör listesidir. Shell soldan sağa arar ve ilk bulduğunu çalıştırır.

```
PATH=/home/galip/.local/bin:/usr/local/bin:/usr/bin:/bin
      ↑ önce burada aranır                  ↑ en son burada
```

## Mini örnek

**Terminalde**
```bash
echo $HOME                     # değişkeni oku
env                            # tüm değişkenler
export DB_HOST=10.0.0.5        # tanımla
export DB_PASSWORD='gizli'     # (gerçekte secret yönetimi kullanın)
echo $PATH                     # PATH'i gör
which python                   # 'python' hangi dosya çalışıyor?
export PATH="$HOME/.local/bin:$PATH"   # PATH'in başına klasör ekle
```

**Python'da okumak**
```python
import os
db_host = os.environ.get("DB_HOST", "localhost")  # yoksa varsayılan
db_password = os.environ["DB_PASSWORD"]           # yoksa hata verir
```

**`.env` dosyası (geliştirme ortamı için)**
```bash
# .env  -> Git'e GÖNDERİLMEZ, .gitignore'a eklenir
DB_HOST=localhost
DB_USER=etl
DB_PASSWORD=gizli
```
```python
from dotenv import load_dotenv   # pip install python-dotenv
load_dotenv()
```

## Nerede kullanılır?
- Veritabanı bağlantı bilgileri, API anahtarları
- Docker'da `-e` veya `--env-file` ile container'a ayar vermek ([4.4](../04-docker/04-docker-komutlari.md))
- Kubernetes'te ConfigMap ve Secret'ların container'a aktarılması ([5.9](../05-kubernetes/09-configmap-secret.md))
- CI/CD pipeline'larında gizli değişkenler

## Benzerleri ve farkları
- **Konfigürasyon dosyası (YAML/JSON):** Çok sayıda ve yapılandırılmış ayar için daha uygundur. Genellikle ikisi birlikte kullanılır: yapı dosyada, şifreler ortam değişkeninde.
- **Secret yönetim araçları (Vault vb.):** Prod ortamında şifreler için doğru yer (Bölüm 15).

Bu yaklaşım "12-Factor App" prensiplerinden **Config** maddesi olarak bilinir: ayarlar ortamdan okunur, koda gömülmez.

## İlgili sayfalar
- [2.1 Sanal ortam](../02-python-ortam/01-neden-sanal-ortam.md)
- [3.4 .gitignore](../03-git/04-gitignore.md)
- [16.8 .env dosyaları](../16-formatlar/08-env.md)
