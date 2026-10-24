# 15.9 Veri Maskeleme, Anonimleştirme ve KVKK

## Nedir?
Kişisel ve hassas verilerin, ihtiyaç olmayan yerlerde **tanınamaz veya okunamaz** hâle getirilmesi için kullanılan teknikler:
- **Maskeleme (masking):** Değerin bir kısmını veya tamamını gizlemek: `12345678901` → `123******01`.
- **Takma adlandırma (pseudonymization):** Doğrudan tanımlayıcıyı, geri eşleştirilebilir bir takma değerle değiştirmek: `TCKN` → `MUS_8f2c...` (eşleşme anahtarı ayrıca korunur).
- **Anonimleştirme (anonymization):** Verinin, kişiyle **hiçbir şekilde** ilişkilendirilemeyecek hâle getirilmesi (geri dönüşü yok).
- **Tokenizasyon:** Değeri anlamsız bir token ile değiştirmek; orijinali ayrı bir kasada tutmak ([15.8](08-sifreleme.md)).

**KVKK:** 6698 sayılı **Kişisel Verilerin Korunması Kanunu**; kişisel verilerin işlenmesine ilişkin ilke ve yükümlülükleri belirler. Bankalar ayrıca Bankacılık Kanunu'ndaki sır saklama yükümlülüğü ve BDDK düzenlemelerine tabidir.

> Not: Bu sayfa teknik bir giriş sunar; hukuki değerlendirme değildir. Somut işlemler için kurumun hukuk, uyum ve bilgi güvenliği birimlerine danışılmalıdır.

## Hangi problemi çözer?
- Geliştirme ve test ortamlarına üretim verisinin **olduğu gibi** kopyalanması
- Analiz ve modelleme için gereğinden fazla kişisel veri kullanılması
- Dashboard, log, e-posta ve ekran görüntülerinde kişisel verinin açığa çıkması
- Veri sızıntısı olduğunda etkinin büyük olması

## Nasıl çalışır?

**KVKK'nın temel ilkeleri (veri ekipleri açısından)**
| İlke | Pratikteki anlamı |
|---|---|
| Hukuka ve dürüstlük kurallarına uygunluk | İşlemenin bir hukuki dayanağı olmalı |
| Belirli, açık ve meşru amaç | Veri, toplandığı amaç dışında kullanılmamalı |
| Amaçla bağlantılı, sınırlı ve ölçülü olma (**veri minimizasyonu**) | Model için TCKN gerekmiyorsa alınmaz |
| Doğru ve güncel olma | Veri kalitesi süreçleri |
| Gerekli süre kadar saklama | Saklama ve imha politikaları |
| Veri güvenliği | Teknik ve idari tedbirler: erişim kontrolü, şifreleme, maskeleme, loglama |

Ayrıca **özel nitelikli kişisel veriler** (sağlık, biyometrik, din, sendika üyeliği vb.) için daha sıkı koşullar vardır; yurt dışına veri aktarımı ayrıca düzenlenmiştir.

**Veri sınıflandırması: önce neyin hassas olduğunu bilmek**
| Sınıf | Örnek | Tipik muamele |
|---|---|---|
| Doğrudan tanımlayıcı | TCKN, ad soyad, telefon, e-posta, IBAN, kart no | Maskele / takma ad / tokenize |
| Dolaylı tanımlayıcı (quasi-identifier) | Doğum tarihi, posta kodu, meslek, işyeri | Genelleştir (yaş grubu, il), birleşimlere dikkat |
| Hassas finansal | Bakiye, gelir, kredi notu | Erişim kontrolü, gerektiğinde gruplama |
| Özel nitelikli | Sağlık, biyometrik | En sıkı kontroller |
| Kişisel olmayan | Ürün kodu, şube kodu | Normal |

**Teknikler**
| Teknik | Örnek | Geri dönüş | Ne zaman? |
|---|---|---|---|
| Kısmi maskeleme | `5412 75** **** 1234` | Hayır (gösterimde) | Ekranlar, raporlar |
| Statik maskeleme | Test veritabanına sahte ama gerçekçi değerler | Hayır | Dev/test ortamları |
| Dinamik maskeleme | Aynı tablo; yetkisiz kullanıcıya maskeli, yetkiliye açık gösterilir | — | Üretim ortamında rol bazlı gösterim |
| Takma adlandırma (anahtarlı hash, HMAC) | `HMAC(gizli_anahtar, TCKN)` | Anahtarla eşlenebilir | Analitikte müşteriyi birleştirme anahtarı |
| Genelleştirme | Yaş 37 → 35–44; adres → il | Hayır | Analiz, paylaşım |
| Gürültü / karıştırma | Değerlere rastgele küçük sapma, sütun içi karıştırma | Hayır | Test verisi, istatistiksel paylaşım |
| Sentetik veri | Gerçek dağılımları taklit eden üretilmiş veri | — | Test, eğitim, dış paylaşım |
| k-anonimlik | Her kayıt en az k-1 başkasıyla aynı dolaylı tanımlayıcılara sahip | — | Yayın/paylaşım öncesi kontrol |

**Önemli:** Sadece TCKN'yi silmek anonimleştirme değildir. Doğum tarihi + ilçe + meslek birleşimi bir kişiyi tek başına tanımlayabilir. Gerçek anonimleştirme zordur; çoğu durumda yapılan şey **takma adlandırmadır** ve veri hâlâ kişisel veri sayılır.

## Mini örnek
**Python'da takma adlandırma ve maskeleme**
```python
import hmac, hashlib, os
import pandas as pd

ANAHTAR = os.environ["TAKMA_AD_ANAHTARI"].encode()   # Vault'tan; asla koda gömülmez

def takma_ad(tckn: str) -> str:
    return "MUS_" + hmac.new(ANAHTAR, tckn.encode(), hashlib.sha256).hexdigest()[:16]

def maskele_kart(kart: str) -> str:
    return kart[:6] + "*" * (len(kart) - 10) + kart[-4:]

df = pd.read_sql("SELECT tckn, ad_soyad, dogum_tarihi, il, kart_no, bakiye FROM musteri", engine)
analiz = pd.DataFrame({
    "musteri_anahtar": df["tckn"].map(takma_ad),                    # birleştirme için tutarlı
    "yas_grubu": pd.cut((pd.Timestamp.today() - pd.to_datetime(df["dogum_tarihi"])).dt.days // 365,
                        bins=[17, 24, 34, 44, 54, 64, 120],
                        labels=["18-24", "25-34", "35-44", "45-54", "55-64", "65+"]),
    "il": df["il"],
    "bakiye": df["bakiye"],
})                                                                  # ad soyad ve kart no hiç alınmadı
```

**Basit k-anonimlik kontrolü**
```python
k = analiz.groupby(["yas_grubu", "il"], observed=True).size()
print("En küçük grup:", k.min())         # örneğin k < 5 olan kombinasyonlar riskli
```

**PostgreSQL'de maskeli view ve rol bazlı erişim**
```sql
CREATE VIEW gold.musteri_rapor AS
SELECT
    musteri_id,
    LEFT(tckn, 3) || '******' || RIGHT(tckn, 2)            AS tckn,
    LEFT(ad_soyad, 1) || '***'                              AS ad_soyad,
    il,
    segment
FROM gold.musteri;

GRANT SELECT ON gold.musteri_rapor TO rol_rapor_okuma;     -- ham tabloya yetki yok
```

**Loglarda maskeleme (regex)**
```python
import re
TCKN = re.compile(r"\b[1-9]\d{10}\b")
KART = re.compile(r"\b(\d{6})\d{6,9}(\d{4})\b")

def log_temizle(metin: str) -> str:
    metin = TCKN.sub("[TCKN]", metin)
    return KART.sub(r"\1******\2", metin)
```

**Test verisi üretimi (Faker, Türkçe)**
```python
from faker import Faker
fake = Faker("tr_TR")
sahte = [{"ad_soyad": fake.name(), "il": fake.city(), "telefon": fake.phone_number()} for _ in range(1000)]
```

## Nerede kullanılır?
- Üretimden test/geliştirme ortamına veri kopyalarken (statik maskeleme veya sentetik veri)
- Analitik ve model geliştirme ortamlarında (takma ad anahtarı ile birleştirme, gereksiz alanların hiç alınmaması)
- Dashboard ve raporlarda (dinamik maskeleme, sadece toplam/özet gösterme)
- Loglar, hata mesajları, destek talepleri
- Dış servislere (bulut, LLM API'leri) veri gönderiminden önce ([13.12](../13-mlops/12-genai-rag.md))

## Benzerleri ve farkları
- **Şifreleme ile maskeleme:** Şifreli veri anahtarla tam olarak geri açılır; maskeleme gösterimi gizler. Amaçları farklıdır, birlikte kullanılır ([15.8](08-sifreleme.md)).
- **GDPR:** AB'nin genel veri koruma tüzüğü; KVKK ile benzer ilkelere sahiptir, AB vatandaşlarının verisi söz konusu olduğunda devreye girer.
- **Diferansiyel gizlilik (differential privacy):** İstatistiksel sonuçlara kontrollü gürültü ekleyerek bireyin varlığının çıkarılamamasını matematiksel olarak sınırlayan ileri teknik.
- **Araçlar:** Veritabanlarının dinamik maskeleme özellikleri (Oracle Data Redaction, SQL Server Dynamic Data Masking), veri kataloglarının hassas veri sınıflandırması, Microsoft Presidio (metinde kişisel veri tespiti).

## İlgili sayfalar
- [15.6 RBAC ve en az yetki](06-rbac-en-az-yetki.md)
- [16.1 Veri yönetişimi](../16-veri-yonetisimi/01-veri-yonetisimi-nedir.md)
- [14.3 Merkezi loglama](../14-gozlemlenebilirlik/03-merkezi-loglama.md)
- [16.6 Veri sınıflandırma](../16-veri-yonetisimi/06-siniflandirma.md)
