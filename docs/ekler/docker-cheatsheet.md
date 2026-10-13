# Docker Cheat Sheet

Ayrıntılar: [Bölüm 4](../04-docker/01-docker-nedir.md)

## Image
| Komut | Açıklama |
|---|---|
| `docker build -t ad:etiket .` | Dockerfile'dan build |
| `docker build -f Dockerfile.prod -t ad:1.0 .` | Farklı Dockerfile |
| `docker images` | Yerel image'lar |
| `docker pull postgres:16` | İndir |
| `docker tag ad:1.0 registry.sirket.local/ekip/ad:1.0` | Yeniden etiketle |
| `docker push registry.sirket.local/ekip/ad:1.0` | Gönder |
| `docker history ad:1.0` | Katmanlar ve boyutları |
| `docker rmi ad:1.0` | Sil |
| `docker login registry.sirket.local` | Registry girişi |

## Container yaşam döngüsü
| Komut | Açıklama |
|---|---|
| `docker run -d --name web -p 8080:80 nginx` | Arka planda, isimli, port yönlendirmeli (host:container) |
| `docker run -it --rm python:3.12 bash` | Etkileşimli, çıkınca sil |
| `docker run -e KEY=val --env-file .env image` | Ortam değişkenleri |
| `docker run -v veri:/data image` | Named volume |
| `docker run -v "$(pwd)":/app image` | Bind mount |
| `docker run --network agim image` | Ağa bağla |
| `docker ps` / `docker ps -a` | Çalışanlar / hepsi |
| `docker stop\|start\|restart web` | Durdur / başlat / yeniden başlat |
| `docker rm web` / `docker rm -f web` | Sil / zorla sil |

## Hata ayıklama
| Komut | Açıklama |
|---|---|
| `docker logs -f --tail 100 web` | Loglar |
| `docker exec -it web bash` (veya `sh`) | İçine gir |
| `docker inspect web` | Ayrıntılı bilgi (JSON) |
| `docker stats` | CPU/bellek kullanımı |
| `docker cp web:/app/log.txt .` | Dosya kopyala |
| `docker run -it --rm --entrypoint sh image` | Image'ı elle incele |

## Volume ve network
| Komut | Açıklama |
|---|---|
| `docker volume ls\|create\|inspect\|rm` | Volume yönetimi |
| `docker network ls\|create\|inspect` | Ağ yönetimi |
| `docker network connect agim web` | Çalışan container'ı ağa ekle |

## Temizlik
| Komut | Açıklama |
|---|---|
| `docker system df` | Disk kullanımı |
| `docker system prune` | Kullanılmayanları temizle |
| `docker image prune -a` | Kullanılmayan tüm image'lar |
| `docker system prune -a --volumes` | Her şey (**volume'ler dahil, dikkat**) |

## Docker Compose
| Komut | Açıklama |
|---|---|
| `docker compose up -d` | Başlat |
| `docker compose up -d --build servis` | Yeniden build edip başlat |
| `docker compose ps` / `logs -f servis` | Durum / log |
| `docker compose exec servis bash` | Servise gir |
| `docker compose run --rm servis komut` | Tek seferlik çalıştır |
| `docker compose down` / `down -v` | Durdur / volume'lerle sil |

## Dockerfile şablonu (Python)
```dockerfile
FROM python:3.12-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN chgrp -R 0 /app && chmod -R g=u /app      # OpenShift uyumu
USER 1001
CMD ["python", "-m", "uygulama"]
```

## .dockerignore başlangıç
```gitignore
.git
.venv
__pycache__/
.env
data/
tests/
```

## Sık hatalar
| Hata | Neden |
|---|---|
| `port is already allocated` | Host portu dolu → başka port |
| Container hemen `Exited (0)` | Ana süreç bitti |
| `localhost`'a bağlanamıyor | Container'daki localhost container'ın kendisi → servis adını kullan |
| `permission denied` (volume) | Container kullanıcısının host klasörüne izni yok |
