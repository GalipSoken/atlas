# Linux / bash Cheat Sheet

Ayrıntılar: [Bölüm 1](../01-linux/01-linux-nedir.md)

## Gezinme ve dosyalar
| Komut | Açıklama |
|---|---|
| `pwd` | Bulunduğun dizin |
| `ls -lah` | Gizliler dahil, okunur boyutlu liste |
| `cd -` | Önceki dizine dön |
| `mkdir -p a/b/c` | İç içe klasör |
| `cp -r kaynak/ hedef/` | Klasör kopyala |
| `mv eski yeni` | Taşı / yeniden adlandır |
| `rm -r klasor/` | Sil (geri alınamaz) |
| `ln -s hedef baglanti` | Sembolik bağlantı |
| `tree -L 2` | Klasör ağacı (kuruluysa) |

## Okuma ve arama
| Komut | Açıklama |
|---|---|
| `cat`, `less`, `head -n 20`, `tail -n 50` | Dosya okuma |
| `tail -f app.log` | Canlı izle |
| `grep -rin "hata" klasor/` | Özyinelemeli, büyük/küçük harf duyarsız, satır numaralı ara |
| `grep -v DEBUG app.log` | Eşleşmeyenleri göster |
| `find . -name "*.csv" -mtime -1` | Son 1 günde değişen csv'ler |
| `find /tmp -type f -size +100M` | 100 MB'tan büyük dosyalar |
| `wc -l dosya` | Satır sayısı |
| `diff a.txt b.txt` | Fark |

## Metin işleme
| Komut | Açıklama |
|---|---|
| `cut -d',' -f2,3 veri.csv` | Sütun seç |
| `sort \| uniq -c \| sort -rn` | Frekans tablosu |
| `awk -F',' '{s+=$3} END {print s}' veri.csv` | 3. sütun toplamı |
| `sed -i 's/eski/yeni/g' dosya` | Yerinde bul-değiştir |
| `tr ';' ','` | Karakter dönüştür |
| `jq '.alan' dosya.json` | JSON alanı |
| `xargs` | Çıktıyı argümana çevir: `find . -name "*.log" \| xargs gzip` |

## Yönlendirme
| İfade | Anlamı |
|---|---|
| `> dosya` / `>> dosya` | Yaz / ekle |
| `2> hata.log` | Hataları yaz |
| `> log 2>&1` | Çıktı + hata aynı dosyaya |
| `a \| b` | a'nın çıktısı b'ye |
| `a && b` / `a \|\| b` | Başarılıysa / başarısızsa |
| `$(komut)` | Komut çıktısını değer olarak kullan |

## İzinler ve kullanıcılar
| Komut | Açıklama |
|---|---|
| `chmod 600 .env` | Sadece sahip okur/yazar |
| `chmod +x script.sh` | Çalıştırılabilir yap |
| `chown kullanici:grup dosya` | Sahip değiştir |
| `sudo komut` | Yönetici yetkisiyle |
| `id`, `whoami`, `groups` | Kimlik bilgisi |

## Süreçler ve servisler
| Komut | Açıklama |
|---|---|
| `ps aux \| grep python` | Süreç ara |
| `top` / `htop` | Canlı kaynak kullanımı |
| `kill PID` / `kill -9 PID` | Durdur / zorla durdur |
| `nohup komut > log 2>&1 &` | Arka planda, oturum kapansa da |
| `systemctl status\|start\|stop\|restart\|enable svc` | Servis yönetimi |
| `journalctl -u svc -f` | Servis logu |
| `ss -tulpn` | Dinlenen portlar |

## Disk, bellek, sistem
| Komut | Açıklama |
|---|---|
| `df -h` | Disk doluluğu |
| `du -sh * \| sort -h` | Klasör boyutları |
| `free -h` | Bellek |
| `uname -a`, `cat /etc/os-release` | Sistem bilgisi |
| `uptime` | Açık kalma süresi, yük |

## Ağ
| Komut | Açıklama |
|---|---|
| `curl -v https://adres` | HTTP isteği, ayrıntılı |
| `nc -vz host 5432` | Port erişilebilir mi |
| `nslookup host` / `dig host +short` | DNS |
| `ssh kullanici@host` | Uzak bağlantı |
| `scp dosya host:/yol/` / `rsync -avz` | Dosya kopyala |

## Arşiv
| Komut | Açıklama |
|---|---|
| `tar czf arsiv.tar.gz klasor/` | Sıkıştır |
| `tar xzf arsiv.tar.gz` | Aç |
| `zip -r a.zip klasor/` / `unzip a.zip` | Zip |
| `gzip dosya` / `gunzip dosya.gz` | Tek dosya |

## Ortam değişkenleri
| Komut | Açıklama |
|---|---|
| `export DEGISKEN=deger` | Tanımla |
| `echo $DEGISKEN`, `env` | Oku / hepsini listele |
| `set -a; source .env; set +a` | .env yükle |
| `which python` | Komutun yolu |

## Script şablonu
```bash
#!/usr/bin/env bash
set -euo pipefail
LOG="/var/log/is_$(date +%F).log"
echo "[$(date '+%F %T')] başladı" >> "$LOG"
# ... komutlar ...
echo "[$(date '+%F %T')] bitti" >> "$LOG"
```

## cron
```
dk saat gün ay haftagünü  komut
0  7    *   *  1-5        /opt/etl/.venv/bin/python /opt/etl/main.py >> /var/log/etl.log 2>&1
```
`crontab -e` düzenle, `crontab -l` listele. Test: [crontab.guru](https://crontab.guru)
