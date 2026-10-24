# LDAP (ldapsearch) Cheat Sheet

Ayrıntılar: [15.2 LDAP](../15-guvenlik/02-ldap.md), [15.3 Active Directory](../15-guvenlik/03-active-directory.md), [15.4 Entegrasyon](../15-guvenlik/04-ldap-entegrasyonu.md)

## Kavramlar
| Terim | Örnek |
|---|---|
| DN (tam adres) | `CN=Mehmet Türk,OU=GenelMudurluk,OU=Kullanicilar,DC=sirket,DC=local` |
| Base DN | `DC=sirket,DC=local` |
| Bind DN (servis hesabı) | `CN=svc-ldap-okuma,OU=ServisHesaplari,DC=sirket,DC=local` veya `svc-ldap-okuma@sirket.local` (AD) |
| Kullanıcı adı özniteliği | OpenLDAP: `uid` · AD: `sAMAccountName`, `userPrincipalName` |
| Grup üyeliği | Kullanıcıda `memberOf` · grupta `member` |
| Portlar | 389 LDAP (+StartTLS) · **636 LDAPS** · 3268/3269 AD Global Catalog |

## Kurulum
```bash
sudo dnf install openldap-clients      # RHEL
sudo apt install ldap-utils            # Debian/Ubuntu
```

## ldapsearch iskeleti
```bash
ldapsearch -H ldaps://dc01.sirket.local:636 \
  -D "svc-ldap-okuma@sirket.local" -W \
  -b "OU=Kullanicilar,DC=sirket,DC=local" \
  -s sub \
  "(sAMAccountName=mturk)" \
  cn mail department memberOf
```
| Parça | Anlamı |
|---|---|
| `-H` | Sunucu adresi (LDAPS) |
| `-D` + `-W` | Bind kimliği; `-W` parolayı sorar (`-y dosya`: parolayı dosyadan okur) |
| `-b` | Arama başlangıcı (base DN) |
| `-s` | Kapsam: `base` \| `one` \| `sub` |
| `"(...)"` | Filtre |
| Sondaki adlar | İstenen öznitelikler |
Yararlı seçenekler: `-LLL` (sade LDIF çıktı), `-x` (basit bind; SASL yerine), `-E pr=1000/noprompt` (sayfalama, AD'nin 1000 sonuç sınırı için), `-ZZ` (StartTLS zorunlu).

## Sık kullanılan aramalar
```bash
# Kullanıcı bilgisi
... -b "DC=sirket,DC=local" "(sAMAccountName=mturk)" displayName mail title department memberOf

# Bir grubun doğrudan üyeleri
... -b "OU=Gruplar,DC=sirket,DC=local" "(cn=veri-ekibi)" member

# Bir grubun (iç içe gruplar dahil) tüm üyesi kullanıcıları (AD)
... "(&(objectClass=user)(memberOf:1.2.840.113556.1.4.1941:=CN=veri-ekibi,OU=Gruplar,DC=sirket,DC=local))" sAMAccountName

# Etkin (devre dışı olmayan) kullanıcılar (AD)
... "(&(objectCategory=person)(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))" sAMAccountName

# Adı "Gal" ile başlayanlar
... "(&(objectClass=user)(givenName=Gal*))" displayName

# E-postası olan, Veri departmanındaki kullanıcılar
... "(&(objectClass=user)(department=Veri*)(mail=*))" mail

# Servis hesapları OU'sundaki hesaplar (tek seviye)
... -b "OU=ServisHesaplari,DC=sirket,DC=local" -s one "(objectClass=user)" cn
```

## Filtre söz dizimi
| İfade | Anlamı |
|---|---|
| `(a=b)` | Eşitlik |
| `(a=*)` | Öznitelik var |
| `(a=ab*)` | Joker |
| `(a>=5)` / `(a<=5)` | Büyük-eşit / küçük-eşit |
| `(&(x)(y))` | VE |
| `(\|(x)(y))` | VEYA |
| `(!(x))` | DEĞİL |
| Özel karakterler | `*` → `\2a`, `(` → `\28`, `)` → `\29`, `\` → `\5c` (kullanıcı girdisini mutlaka kaçışla) |

## Kimlik doğrulama testleri
```bash
# Kullanıcının parolası doğru mu? (kendi DN'i veya UPN'i ile bind)
ldapwhoami -H ldaps://dc01.sirket.local -D "mturk@sirket.local" -W

# Sertifika / TLS sorunu var mı?
openssl s_client -connect dc01.sirket.local:636 -showcerts </dev/null
LDAPTLS_CACERT=/etc/ssl/sirket-ca.pem ldapsearch -H ldaps://dc01.sirket.local ...
# ~/.ldaprc veya /etc/openldap/ldap.conf içinde: TLS_CACERT /etc/ssl/sirket-ca.pem
```

## Python (ldap3)
```python
from ldap3 import Server, Connection, SUBTREE
from ldap3.utils.conv import escape_filter_chars

s = Server("ldaps://dc01.sirket.local", use_ssl=True)
c = Connection(s, user="svc-ldap-okuma@sirket.local", password=PAROLA, auto_bind=True)
c.search("DC=sirket,DC=local",
         f"(sAMAccountName={escape_filter_chars(kullanici)})",
         search_scope=SUBTREE, attributes=["displayName", "mail", "memberOf"])
for e in c.entries:
    print(e.entry_dn, e.mail, e.memberOf.values)
```

## OpenShift
```bash
oc adm groups sync --sync-config=ldap-sync.yaml --confirm                     # grupları senkronize et
oc adm groups sync --sync-config=ldap-sync.yaml --whitelist=gruplar.txt --confirm
oc get groups
oc adm policy add-role-to-group view veri-ekibi -n veri-prod
```

## Sık hatalar
| Hata | Neden / çözüm |
|---|---|
| `Can't contact LDAP server (-1)` | DNS, firewall, yanlış port; `nc -vz dc01 636` |
| `Invalid credentials (49)` | Bind DN/parola yanlış, hesap kilitli veya parolanın süresi dolmuş |
| `certificate verify failed` | Kurum CA'sı tanıtılmamış |
| `Size limit exceeded (4)` | Sonuç sınırı; sayfalama (`-E pr=1000/noprompt`) |
| `No such object (32)` | Base DN yanlış |
| Kullanıcı bulunuyor ama gruplar boş | İç içe gruplar; AD'de `1.2.840.113556.1.4.1941` eşleştirme kuralı |
