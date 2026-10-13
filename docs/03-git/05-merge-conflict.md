# 3.5 Merge Conflict Çözümü

## Nedir?
**Merge conflict (birleştirme çakışması)**, iki dalda **aynı dosyanın aynı satırları** farklı şekilde değiştirildiğinde ortaya çıkar. Git hangisinin doğru olduğuna karar veremez ve kararı size bırakır.

## Hangi problemi çözer?
Conflict bir hata değil, **güvenlik mekanizmasıdır**: Git birinin işini sessizce silmek yerine durur ve sorar. Çözmeyi bilmek, ekip çalışmasının temel becerisidir.

## Nasıl çalışır?
Conflict şu işlemlerde çıkabilir: `merge`, `pull`, `rebase`, `cherry-pick`, `stash pop`.

Git çakışan bölgeyi dosyanın içine işaretlerle yazar:

```python
def faiz_hesapla(anapara, oran):
<<<<<<< HEAD
    return anapara * oran / 100
=======
    return round(anapara * oran / 100, 2)
>>>>>>> ozellik/yuvarlama
```

| İşaret | Anlamı |
|---|---|
| `<<<<<<< HEAD` | Sizin bulunduğunuz dalın hâli |
| `=======` | Ayırıcı |
| `>>>>>>> ozellik/yuvarlama` | Birleştirilmeye çalışılan dalın hâli |

Çözüm: İşaretleri silin, doğru son hâli bırakın (birini, diğerini veya ikisinin birleşimini).

## Mini örnek
```bash
git switch main
git merge ozellik/yuvarlama
# CONFLICT (content): Merge conflict in hesap.py

git status                     # çakışan dosyalar "both modified" olarak listelenir

# hesap.py'yi editörde aç, işaretleri temizle:
#     return round(anapara * oran / 100, 2)

git add hesap.py               # "çözüldü" olarak işaretle
git commit                     # merge commit'ini tamamla
```

Vazgeçmek isterseniz:
```bash
git merge --abort              # merge öncesine dön
git rebase --abort             # rebase öncesine dön
```

Tüm dosyada bir tarafı seçmek:
```bash
git checkout --ours hesap.py     # kendi dalımın hâli
git checkout --theirs hesap.py   # gelen dalın hâli
```

## Nerede kullanılır?
- Uzun yaşayan bir dalı `main` ile güncellerken
- İki kişi aynı SQL veya konfigürasyon dosyasında çalışırken
- Pull Request'te platform "This branch has conflicts" uyarısı verdiğinde

## Benzerleri ve farkları
**Conflict'i azaltmanın yolları:**
- **Küçük ve kısa ömürlü dallar:** Birkaç günde birleştirilen dal az çakışır ([3.7](07-branch-stratejileri.md)).
- **Sık güncelleme:** Dalınıza düzenli olarak `main`'i alın (`git pull origin main` veya `git rebase main`).
- **Formatlayıcı kullanmak:** `black`, `ruff format` gibi araçlar herkesin kodunu aynı biçime getirir; boşluk ve girinti kaynaklı sahte çakışmalar biter.
- **Dosya sahipliği:** Ekip içinde kimin hangi modülle ilgilendiği net olursa aynı satırlara aynı anda dokunulmaz.

**Araçlar:** VS Code çakışmaları renkli gösterir ve "Accept Current / Incoming / Both" butonları sunar. `git mergetool` ile harici araç (Meld, KDiff3) açılabilir.

**Dikkat:** Çözümden sonra kodu **çalıştırın ve test edin**. Git metin olarak birleştirir; birleşen kodun mantıksal olarak doğru olduğunu garanti etmez.

## İlgili sayfalar
- [3.2 Temel kavramlar](02-temel-kavramlar.md)
- [3.6 Pull Request](06-github-pull-request.md)
