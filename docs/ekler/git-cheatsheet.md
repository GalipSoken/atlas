# Git Cheat Sheet

Ayrıntılar: [Bölüm 3](../03-git/01-git-nedir.md)

## Kurulum ve başlangıç
| Komut | Açıklama |
|---|---|
| `git config --global user.name "Ad Soyad"` | İsim |
| `git config --global user.email "ad@sirket.local"` | E-posta |
| `git config --global init.defaultBranch main` | Varsayılan dal adı |
| `git init` | Klasörü repo yap |
| `git clone <url>` | Uzak repoyu indir |

## Günlük akış
```bash
git switch main && git pull               # güncel ol
git switch -c feature/is-adi              # dal aç
# ... çalış ...
git status && git diff                    # kontrol et
git add -p                                # parça parça seç (veya: git add .)
git commit -m "feat: kısa açıklama"
git push -u origin feature/is-adi         # gönder → PR aç
```

## Durum ve geçmiş
| Komut | Açıklama |
|---|---|
| `git status -sb` | Kısa durum |
| `git diff` / `git diff --staged` | Staging dışı / staging'deki farklar |
| `git log --oneline --graph --all` | Dallı tek satır geçmiş |
| `git log -p -- dosya.py` | Dosyanın değişim geçmişi |
| `git show <hash>` | Bir commit'in içeriği |
| `git blame dosya.py` | Satırı kim değiştirdi |

## Dallar
| Komut | Açıklama |
|---|---|
| `git branch -a` | Tüm dallar |
| `git switch dal` / `git switch -c yeni` | Geç / oluşturup geç |
| `git merge dal` | Birleştir |
| `git rebase main` | Dalı main'in ucuna taşı (paylaşılmamış dallarda) |
| `git branch -d dal` | Birleşmiş dalı sil |
| `git push origin --delete dal` | Uzaktaki dalı sil |

## Uzak repo
| Komut | Açıklama |
|---|---|
| `git remote -v` | Bağlı uzak repolar |
| `git fetch` | İndir, birleştirme |
| `git pull --rebase` | İndir, kendi commit'lerini üstüne taşı |
| `git push` | Gönder |

## Geçici saklama
| Komut | Açıklama |
|---|---|
| `git stash` / `git stash -u` | Değişiklikleri sakla (izlenmeyen dosyalar dahil: `-u`) |
| `git stash list` | Liste |
| `git stash pop` | Geri getir |

## Geri alma
| Durum | Komut |
|---|---|
| Kaydedilmemiş değişikliği at | `git restore dosya` |
| Staging'den çıkar | `git restore --staged dosya` |
| Son commit mesajını/içeriğini düzelt (push edilmediyse) | `git commit --amend` |
| Son commit'i geri al, değişiklikler kalsın | `git reset --soft HEAD~1` |
| Push edilmiş commit'i güvenle geri al | `git revert <hash>` |
| Her şeyi son commit'e döndür (**tehlikeli**) | `git reset --hard HEAD` |
| "Kaybolan" commit'i bul | `git reflog` |
| Conflict'li merge'den vazgeç | `git merge --abort` |

## Etiketler
| Komut | Açıklama |
|---|---|
| `git tag -a v1.2.0 -m "Sürüm 1.2.0"` | Açıklamalı etiket |
| `git push origin v1.2.0` / `--tags` | Etiketi gönder |

## Merge conflict
```bash
git merge feature/x          # CONFLICT
git status                   # çakışan dosyalar
# dosyadaki <<<<<<< ======= >>>>>>> bölümlerini düzelt
git add dosya && git commit
```

## Commit mesajı kalıbı (Conventional Commits)
```
feat: yeni özellik      fix: hata düzeltmesi     docs: dokümantasyon
refactor: yeniden düzenleme    test: test     chore: bakım işleri
```

## .gitignore başlangıç
```gitignore
.venv/
__pycache__/
.env
*.csv
*.parquet
.ipynb_checkpoints/
```

## GitHub CLI
| Komut | Açıklama |
|---|---|
| `gh pr create` | PR aç |
| `gh pr status` / `gh pr checks` | Durum / CI sonuçları |
| `gh pr merge --squash` | Birleştir |
