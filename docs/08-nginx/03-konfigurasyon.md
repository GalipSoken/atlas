# 8.3 Konfigürasyon Yapısı: server, location, upstream

## Nedir?
nginx'in davranışı metin tabanlı konfigürasyon dosyalarıyla tanımlanır. Dosya, iç içe **bloklar** (context) ve bu bloklar içindeki **direktiflerden** (directive) oluşur.

## Hangi problemi çözer?
Tek bir nginx, birden çok alan adını, farklı URL yollarını ve arkadaki birçok uygulamayı tek dosyada, okunabilir kurallarla yönetir.

## Nasıl çalışır?

**Blok hiyerarşisi**
```nginx
# ana (main) bağlam
worker_processes auto;

events {
    worker_connections 1024;
}

http {                                   # HTTP ayarları
    upstream api_sunuculari {            # arka uç grubu
        server 10.0.0.11:8000;
        server 10.0.0.12:8000;
    }

    server {                             # bir sanal sunucu (site)
        listen 80;
        server_name api.sirket.local;

        location / { ... }               # URL yoluna göre kural
        location /static/ { ... }
    }
}
```

| Blok | Görevi |
|---|---|
| **main** | Genel ayarlar: worker sayısı, kullanıcı, log |
| **events** | Bağlantı işleme ayarları |
| **http** | Tüm HTTP siteleri için ortak ayarlar |
| **server** | Bir site: hangi port (`listen`) ve hangi alan adı (`server_name`) |
| **location** | URL yoluna göre ne yapılacağı |
| **upstream** | İsteklerin iletileceği arka uç sunucu grubu |

**`server` seçimi:** İstek gelince nginx önce `listen` portuna, sonra `Host` header'ını `server_name` ile eşleştirir. Eşleşme yoksa `default_server` kullanılır.

**`location` eşleşme kuralları**

| Söz dizimi | Anlamı | Öncelik |
|---|---|---|
| `location = /health` | Tam eşleşme | 1 (en yüksek) |
| `location ^~ /static/` | Önek; eşleşirse regex'lere bakılmaz | 2 |
| `location ~ \.php$` | Regex (büyük/küçük harf duyarlı) | 3 |
| `location ~* \.(jpg\|png)$` | Regex (duyarsız) | 3 |
| `location /api/` | Önek (en uzun eşleşen kazanır) | 4 |

**Sık kullanılan direktifler**

| Direktif | Görevi |
|---|---|
| `root` / `alias` | Statik dosyaların disk yolu |
| `index` | Klasör istenince gösterilecek dosya |
| `try_files` | Sırayla dosya dene, yoksa son seçenek (tek sayfa uygulamalar için) |
| `proxy_pass` | İsteği arka uca ilet ([8.4](04-reverse-proxy.md)) |
| `return` | Sabit cevap veya yönlendirme (`return 301 https://...`) |
| `rewrite` | URL'yi değiştir |
| `include` | Başka dosyayı dahil et |
| `client_max_body_size` | En büyük istek gövdesi (dosya yükleme) |
| `access_log` / `error_log` | Log ayarları |

**Değişkenler:** `$host`, `$remote_addr`, `$request_uri`, `$scheme` gibi değişkenler istek bilgilerini taşır.

## Mini örnek
Statik rapor sitesi + API + sağlık kontrolü (`/etc/nginx/conf.d/veri.conf`):
```nginx
upstream kredi_api {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name veri.sirket.local;

    access_log /var/log/nginx/veri_access.log;
    error_log  /var/log/nginx/veri_error.log warn;

    client_max_body_size 20m;

    location = /health {
        return 200 "ok\n";
    }

    # Statik HTML raporlar
    location /raporlar/ {
        alias /srv/raporlar/;
        autoindex on;                    # klasör listesini göster
    }

    # Tek sayfa ön yüz uygulaması (React/Vue)
    location / {
        root /srv/frontend;
        try_files $uri $uri/ /index.html;
    }

    # API
    location /api/ {
        proxy_pass http://kredi_api/;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
curl http://veri.sirket.local/health
```

**`root` ile `alias` farkı**
- `location /raporlar/ { root /srv; }` → `/raporlar/a.html` isteği `/srv/raporlar/a.html` dosyasını arar.
- `location /raporlar/ { alias /srv/raporlar/; }` → aynı istek `/srv/raporlar/a.html` (location kısmı alias ile değiştirilir).

## Nerede kullanılır?
- Sunucuya doğrudan kurulan nginx'te `/etc/nginx/conf.d/`
- Docker image'ında `COPY default.conf /etc/nginx/conf.d/`
- Kubernetes'te ConfigMap olarak bağlanarak
- Ingress Controller'da konfigürasyon, Ingress nesnelerinden otomatik üretilir; ince ayarlar annotation ile yapılır

## Benzerleri ve farkları
- **Apache `.htaccess`:** Klasör bazında dağınık ayar; nginx'te bu yoktur, tüm ayar merkezidir (daha hızlı ve öngörülebilir).
- **Sık hata:** `proxy_pass` sonunda `/` olup olmaması yolu değiştirir ([8.4](04-reverse-proxy.md)).
- **Sık hata:** Değişiklikten sonra `nginx -t` yapmadan reload → hatalı konfigürasyonla servis kesilebilir.

## İlgili sayfalar
- [8.4 Reverse proxy](04-reverse-proxy.md)
- [nginx cheat sheet](../ekler/nginx-cheatsheet.md)
