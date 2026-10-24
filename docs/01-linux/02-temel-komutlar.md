# 1.2 Dosya Sistemi ve Temel Komutlar

## Nedir?
Linux'ta tüm dosyalar tek bir kök dizinden (`/`) başlayan ağaç yapısında durur. Terminal komutlarıyla bu ağaçta gezinir, dosya oluşturur, okur, arar ve sileriz.

## Hangi problemi çözer?
Sunucularda grafik arayüz yoktur. Log okumak, konfigürasyon değiştirmek, veri dosyası taşımak gibi her iş terminalden yapılır.

## Nasıl çalışır?

Önemli dizinler:

| Dizin | İçerik |
|---|---|
| `/` | Kök dizin |
| `/home/kullanici` | Kullanıcının kişisel klasörü (`~` ile kısaltılır) |
| `/etc` | Konfigürasyon dosyaları |
| `/var/log` | Log dosyaları |
| `/tmp` | Geçici dosyalar (yeniden başlatınca silinebilir) |
| `/opt` | Ek kurulan uygulamalar |
| `/usr/bin` | Komutların çalıştırılabilir dosyaları |

**Yol (path) türleri:**
- **Mutlak yol:** `/` ile başlar → `/var/log/nginx/access.log`
- **Göreli yol:** Bulunulan dizine göre → `logs/app.log`, `../config`
- `.` bulunulan dizin, `..` bir üst dizin, `~` ev dizini.

## Mini örnek

**Gezinme**
```bash
pwd                     # neredeyim?
ls                      # içerik
ls -la                  # gizli dosyalar dahil, detaylı liste
cd /var/log             # dizine git
cd ..                   # bir üst dizin
cd ~                    # ev dizinine dön
```

**Dosya ve klasör işlemleri**
```bash
mkdir -p proje/data/raw       # iç içe klasör oluştur
touch notlar.txt              # boş dosya oluştur
cp kaynak.csv yedek.csv       # kopyala
cp -r klasor/ klasor_yedek/   # klasörü kopyala
mv eski.csv yeni.csv          # taşı / yeniden adlandır
rm dosya.txt                  # sil (çöp kutusu yok!)
rm -r klasor/                 # klasörü içiyle sil
```

**Dosya okuma**
```bash
cat config.yaml               # tamamını yaz
less buyuk_log.log            # sayfa sayfa oku (q ile çık)
head -n 20 veri.csv           # ilk 20 satır
tail -n 50 app.log            # son 50 satır
tail -f app.log               # dosyaya eklenenleri canlı izle
wc -l veri.csv                # satır sayısı
```

**Arama**
```bash
grep "ERROR" app.log                  # içinde ERROR geçen satırlar
grep -i "timeout" -r /var/log/app/    # büyük/küçük harf duyarsız, klasörde
grep -c "ERROR" app.log               # kaç satırda geçiyor
find . -name "*.csv"                  # bu dizinin altındaki csv'ler
find /data -mtime -1                  # son 1 günde değişen dosyalar
```

**Disk ve boyut**
```bash
df -h          # disk bölümlerinin doluluğu
du -sh *       # bu dizindeki öğelerin boyutu
```

## Nerede kullanılır?
- ETL sunucusunda gelen dosyaların kontrolü (`ls -la`, `wc -l`)
- Hata ayıklarken log takibi (`tail -f`, `grep ERROR`)
- Disk dolduğunda büyük klasörleri bulmak (`du -sh`)

## Benzerleri ve farkları
| Linux | Windows (cmd) | Windows (PowerShell) |
|---|---|---|
| `ls` | `dir` | `Get-ChildItem` / `ls` |
| `cat` | `type` | `Get-Content` |
| `grep` | `findstr` | `Select-String` |
| `rm` | `del` | `Remove-Item` |

**Dikkat:** `rm -rf` geri alınamaz. Özellikle değişkenli yollarda (`rm -rf $KLASOR/`) değişken boşsa kök dizini silebilir.

## İlgili sayfalar
- [1.3 İzinler](03-izinler.md)
- [1.7 Pipe ve bash script](07-bash-script.md)
- [Linux cheat sheet](../ekler/linux-bash-cheatsheet.md)
- [vim / nano cheat sheet](../ekler/vim-nano-cheatsheet.md)
