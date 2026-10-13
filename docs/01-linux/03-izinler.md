# 1.3 Kullanıcılar, Gruplar ve Dosya İzinleri

## Nedir?
Linux çok kullanıcılı bir sistemdir. Her dosyanın bir **sahibi** (user), bir **grubu** (group) ve bu ikisi ile **diğerleri** (others) için ayrı ayrı **okuma, yazma, çalıştırma** izinleri vardır.

## Hangi problemi çözer?
Aynı sunucuyu birçok kişi ve servis paylaşır. İzinler sayesinde:
- ETL kullanıcısı sadece kendi klasörüne yazabilir,
- Uygulama kullanıcısı şifre dosyasını okuyabilir ama başkaları okuyamaz,
- Yanlışlıkla sistem dosyaları değiştirilemez.

## Nasıl çalışır?

`ls -l` çıktısı:
```
-rwxr-x---  1  etl_user  veri_ekibi  2048  Oct 13 10:00  yukle.sh
│└┬┘└┬┘└┬┘     └───┬───┘ └────┬───┘
│ │  │  │          sahip      grup
│ │  │  └ diğerleri: izin yok
│ │  └ grup: okuma + çalıştırma
│ └ sahip: okuma + yazma + çalıştırma
└ tür: - dosya, d dizin, l link
```

| Harf | Dosyada | Dizinde | Sayı |
|---|---|---|---|
| `r` | okuyabilir | içeriği listeleyebilir | 4 |
| `w` | değiştirebilir | içine dosya ekleyip silebilir | 2 |
| `x` | çalıştırabilir | içine girebilir (`cd`) | 1 |

Sayısal gösterim her grup için toplamdır: `rwx`=7, `r-x`=5, `r--`=4. Örneğin `750` = sahip 7, grup 5, diğerleri 0.

**root:** Her şeye yetkisi olan süper kullanıcı. Günlük işler root ile yapılmaz; gerektiğinde `sudo` ile geçici yetki alınır.

## Mini örnek
```bash
whoami                         # aktif kullanıcı
id                             # kullanıcı ve grupları
ls -l yukle.sh                 # izinleri gör

chmod +x yukle.sh              # çalıştırma izni ekle
chmod 750 yukle.sh             # sahip: rwx, grup: r-x, diğer: ---
chmod 600 .env                 # sadece sahip okuyup yazsın (şifre dosyası)
chmod -R 755 public/           # klasör ve altına uygula

sudo chown etl_user:veri_ekibi /data/bddk   # sahip ve grubu değiştir
sudo systemctl restart nginx                # yönetici yetkisiyle komut
```

## Nerede kullanılır?
- Şifre içeren `.env` ve anahtar dosyalarını korumak (`600`)
- Script'leri çalıştırılabilir yapmak (`chmod +x`)
- Ortak veri klasörlerinde ekip grubuna yazma izni vermek
- SSH anahtarları: `~/.ssh` dizini `700`, özel anahtar `600` olmalı, yoksa SSH kabul etmez

## Benzerleri ve farkları
- **Windows ACL:** Daha ayrıntılı, kullanıcı bazlı izin listeleri.
- **Container'larda:** Güvenlik için container'lar root olmayan kullanıcıyla çalıştırılmalıdır. OpenShift bunu varsayılan olarak zorunlu kılar ([5.17](../05-kubernetes/17-openshift-kavramlari.md)).
- **Kurumsal kimlik:** Sunucu kullanıcıları çoğu zaman LDAP / Active Directory'den gelir (Bölüm 15).

**Sık hata:** "Permission denied" alınca `chmod 777` yapmak. Bu herkese her yetkiyi verir; asıl sorunu (yanlış sahip veya grup) bulmak gerekir.

## İlgili sayfalar
- [1.4 Süreçler ve servisler](04-surecler.md)
- [1.6 SSH](06-ssh.md)
