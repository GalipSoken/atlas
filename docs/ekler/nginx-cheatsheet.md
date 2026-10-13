# nginx Cheat Sheet

Ayrıntılar: [Bölüm 8](../08-nginx/02-nginx-nedir.md)

## Komutlar
| Komut | Açıklama |
|---|---|
| `sudo nginx -t` | Konfigürasyonu test et (**her değişiklikten sonra**) |
| `sudo systemctl reload nginx` | Kesintisiz yeniden yükle |
| `sudo systemctl restart nginx` | Yeniden başlat |
| `sudo nginx -T` | Tüm etkin konfigürasyonu yazdır |
| `nginx -V` | Sürüm ve modüller |
| `sudo tail -f /var/log/nginx/access.log /var/log/nginx/error.log` | Loglar |

## Dosya yerleri
| Yol | İçerik |
|---|---|
| `/etc/nginx/nginx.conf` | Ana konfigürasyon |
| `/etc/nginx/conf.d/*.conf` | Site konfigürasyonları |
| `/usr/share/nginx/html` | Varsayılan statik içerik |
| `/var/log/nginx/` | Loglar |

## Statik site
```nginx
server {
    listen 80;
    server_name rapor.sirket.local;
    root /srv/rapor;
    index index.html;
    location / { try_files $uri $uri/ /index.html; }
}
```

## Reverse proxy (API + Streamlit)
```nginx
server {
    listen 80;
    server_name veri.sirket.local;

    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    location /api/ {
        proxy_pass http://127.0.0.1:8000/;      # sondaki / → /api/ öneki atılır
    }

    location /dashboard/ {                      # WebSocket gerekli
        proxy_pass http://127.0.0.1:8501/dashboard/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
}
```

## HTTPS
```nginx
server { listen 80; server_name api.sirket.local; return 301 https://$host$request_uri; }

server {
    listen 443 ssl;
    server_name api.sirket.local;
    ssl_certificate     /etc/nginx/tls/fullchain.pem;
    ssl_certificate_key /etc/nginx/tls/api.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    location / { proxy_pass http://127.0.0.1:8000; }
}
```

## Load balancing
```nginx
upstream api {
    least_conn;
    server 10.0.0.11:8000 max_fails=3 fail_timeout=30s;
    server 10.0.0.12:8000 max_fails=3 fail_timeout=30s;
    server 10.0.0.13:8000 backup;
}
server { listen 80; location / { proxy_pass http://api; } }
```

## Rate limit, boyut, zaman aşımı, gzip
```nginx
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    gzip on;
    gzip_types application/json text/css application/javascript;

    server {
        client_max_body_size 50m;               # 413 hatası için
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            limit_req_status 429;
            proxy_read_timeout 120s;             # 504 için
            proxy_pass http://127.0.0.1:8000/;
        }
    }
}
```

## location öncelikleri
| Söz dizimi | Anlamı | Öncelik |
|---|---|---|
| `location = /yol` | Tam eşleşme | 1 |
| `location ^~ /yol/` | Önek, regex'i atla | 2 |
| `location ~ regex` / `~* regex` | Regex (duyarlı / duyarsız) | 3 |
| `location /yol/` | Önek (en uzun kazanır) | 4 |

## Hata kodları
| Kod | Bak |
|---|---|
| 502 | Uygulama ayakta mı, port doğru mu: `curl 127.0.0.1:8000` |
| 504 | Uygulama yavaş → `proxy_read_timeout` veya işi asenkron yap |
| 413 | `client_max_body_size` |
| 404 | `root`/`alias`, `proxy_pass` sonundaki `/` |
| 403 | Dosya izinleri, index yok |
| 429 / 503 | Rate limit |

## Log analizi
```bash
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn     # durum kodu dağılımı
grep -E '" 5[0-9]{2} ' /var/log/nginx/access.log | tail                      # 5xx'ler
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head # en çok istek yapan IP'ler
```

## Container / OpenShift
- Root olmayan image: `nginxinc/nginx-unprivileged` (port **8080**)
- Konfigürasyon: `COPY default.conf /etc/nginx/conf.d/default.conf` veya ConfigMap ile bağla
- Kubernetes'te ingress-nginx ayarları annotation ile: `nginx.ingress.kubernetes.io/proxy-body-size: "50m"` ([8.9](../08-nginx/09-ingress-controller.md))
