# Regex (Düzenli İfadeler) Cheat Sheet

Düzenli ifadeler (regular expressions), metin içinde desen aramak, doğrulamak ve değiştirmek için kullanılır: `grep`, `sed`, Python `re`, pandas `.str` metotları, SQL (`~`, `REGEXP_LIKE`), log sorguları, nginx `location ~`.

## Temel yapı taşları
| Desen | Anlamı | Örnek | Eşleşir |
|---|---|---|---|
| `.` | Herhangi bir karakter (satır sonu hariç) | `a.c` | `abc`, `a-c` |
| `\d` / `\D` | Rakam / rakam olmayan | `\d\d` | `42` |
| `\w` / `\W` | Harf, rakam, `_` / değil | `\w+` | `kredi_2026` |
| `\s` / `\S` | Boşluk (space, tab, satır sonu) / değil | `a\sb` | `a b` |
| `[abc]` | Kümedeki bir karakter | `[KT]` | `K`, `T` |
| `[^abc]` | Küme dışı bir karakter | `[^0-9]` | `a` |
| `[a-z0-9]` | Aralık | `[A-F0-9]` | `B` |
| `^` / `$` | Satır başı / sonu | `^ERROR` | satır `ERROR` ile başlıyorsa |
| `\b` | Kelime sınırı | `\bkredi\b` | `kredi` (ama `krediler` değil) |
| `\` | Özel karakteri kaçışla | `\.` | `.` (nokta) |

## Tekrar (niceleyiciler)
| Desen | Anlamı |
|---|---|
| `*` | 0 veya daha fazla |
| `+` | 1 veya daha fazla |
| `?` | 0 veya 1 (isteğe bağlı) |
| `{n}` | Tam n kez |
| `{n,}` / `{n,m}` | En az n / n ile m arası |
| `*?`, `+?` | **Tembel (lazy)**: mümkün olan en kısa eşleşme |

Açgözlü ve tembel farkı: `<.+>` metni `<a><b>` için tamamını, `<.+?>` ise sadece `<a>`'yı yakalar.

## Gruplar ve alternatifler
| Desen | Anlamı |
|---|---|
| `(abc)` | Yakalayan grup; sonradan `\1` / `$1` / `m.group(1)` |
| `(?:abc)` | Yakalamayan grup |
| `(?P<ad>...)` | İsimli grup (Python) |
| `a\|b` | a veya b |
| `(?=...)` / `(?!...)` | İleri bakış: ardından ... gelirse / gelmezse |
| `(?<=...)` / `(?<!...)` | Geri bakış |

## Bayraklar (flags)
| Python | Anlamı |
|---|---|
| `re.IGNORECASE` (`(?i)`) | Büyük/küçük harf duyarsız |
| `re.MULTILINE` (`(?m)`) | `^` ve `$` her satırda çalışır |
| `re.DOTALL` (`(?s)`) | `.` satır sonunu da eşler |
| `re.VERBOSE` (`(?x)`) | Boşluk ve yorumlarla okunaklı desen |

## Hazır desenler (kurum içi kullanım için başlangıç)
| Amaç | Desen | Not |
|---|---|---|
| TCKN biçimi | `\b[1-9]\d{10}\b` | Sadece biçim; algoritma kontrolü ayrıca yapılmalı |
| Kart numarası (16 hane, boşluklu/boşluksuz) | `\b\d{4}[ -]?\d{4}[ -]?\d{4}[ -]?\d{4}\b` | Maskeleme için ([15.9](../15-guvenlik/09-maskeleme-kvkk.md)) |
| IBAN (TR) | `\bTR\d{2}\s?(?:\d{4}\s?){5}\d{2}\b` | Biçim kontrolü |
| E-posta (basit) | `[\w.+-]+@[\w-]+\.[\w.-]+` | Tam RFC uyumu için kütüphane kullanın |
| ISO tarih | `\d{4}-\d{2}-\d{2}` | `2026-10-13` |
| Dönem (YYYY-AA) | `^\d{4}-(0[1-9]\|1[0-2])$` | `2026-09` |
| Türkçe sayı biçimi | `^\d{1,3}(\.\d{3})*(,\d+)?$` | `1.250.000,50` |
| IPv4 (basit) | `\b(?:\d{1,3}\.){3}\d{1,3}\b` | Değer aralığını ayrıca kontrol edin |
| Log seviyesi | `\b(ERROR\|WARN(ING)?\|CRITICAL)\b` | |
| HTTP 5xx (nginx log) | `" 5\d{2} ` | |

## Python `re`
```python
import re

re.search(r"\d{4}-\d{2}", "Dönem: 2026-09")            # ilk eşleşme (Match veya None)
re.fullmatch(r"\d{4}-\d{2}", "2026-09")                # tüm metin desene uymalı (doğrulama)
re.findall(r"\b\d{11}\b", metin)                       # tüm eşleşmeler (liste)
re.sub(r"(\d{6})\d{6}(\d{4})", r"\1******\2", kart)    # değiştir (grup referansıyla)
re.split(r"[;,]\s*", "a; b,c")                         # böl

m = re.search(r"(?P<yil>\d{4})-(?P<ay>\d{2})", "2026-09")
m.group("yil"), m.groupdict()

DONEM = re.compile(r"^\d{4}-(0[1-9]|1[0-2])$")         # tekrar kullanılacaksa derle
bool(DONEM.match("2026-13"))                           # False
```
Desenleri her zaman **ham metin** (`r"..."`) olarak yazın; aksi hâlde `\b`, `\d` Python tarafından yorumlanır.

## pandas
```python
df[df["unvan"].str.contains(r"\bbank", case=False, regex=True)]
df["il"] = df["adres"].str.extract(r"/\s*(\w+)\s*$")           # grup → yeni sütun
df["tutar"] = df["tutar_metin"].str.replace(r"\.", "", regex=True).str.replace(",", ".")
df["donem_gecerli"] = df["donem"].str.fullmatch(r"\d{4}-\d{2}")
```

## Komut satırı
```bash
grep -E "ERROR|CRITICAL" app.log                 # -E: genişletilmiş regex
grep -oE "\b[0-9]{11}\b" dosya.txt               # sadece eşleşen kısmı yazdır
grep -P "(?<=user=)\w+" app.log                  # -P: Perl uyumlu (lookbehind vb.)
sed -E 's/([0-9]{6})[0-9]{6}([0-9]{4})/\1******\2/g' log.txt   # kart maskeleme
```

## SQL
```sql
-- PostgreSQL
SELECT * FROM musteri WHERE unvan ~* '\mbank';                 -- ~* duyarsız; \m kelime başı
SELECT regexp_replace(tutar_metin, '\.', '', 'g') FROM ham;
SELECT substring(dosya_adi FROM '(\d{4}-\d{2})') AS donem FROM yuklemeler;

-- Oracle
SELECT * FROM musteri WHERE REGEXP_LIKE(unvan, 'bank', 'i');
SELECT REGEXP_SUBSTR(dosya_adi, '\d{4}-\d{2}') FROM yuklemeler;
```

## Lehçe farkları
| Özellik | POSIX ERE (`grep -E`, `sed -E`) | PCRE (`grep -P`, Python, JS) |
|---|---|---|
| `\d`, `\w` | Desteklenmeyebilir → `[0-9]`, `[[:alnum:]_]` | Var |
| Tembel `*?` | Yok | Var |
| Lookahead/behind | Yok | Var |

**Uyarılar:** Güvenilmeyen girdiyle karmaşık ve iç içe tekrarlı desenler (`(a+)+`) aşırı yavaşlayabilir (ReDoS). JSON, HTML, CSV gibi yapılı verileri regex yerine kendi ayrıştırıcılarıyla işleyin. Deneme için: [regex101.com](https://regex101.com) (hassas veriyi yapıştırmayın).
