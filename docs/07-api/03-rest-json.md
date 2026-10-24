# 7.3 REST ve JSON

## Nedir?
- **REST (Representational State Transfer):** HTTP'nin doğal yapısını kullanarak API tasarlamaya yönelik bir mimari tarzdır. Her şey bir **kaynak** (resource) olarak düşünülür ve URL ile adreslenir; kaynaklar üzerindeki işlemler HTTP metotlarıyla yapılır.
- **JSON (JavaScript Object Notation):** İnsan tarafından okunabilir, hafif veri formatı. REST API'lerin fiilî standart veri dilidir.

## Hangi problemi çözer?
API tasarımında ortak bir dil sağlar: Bir geliştirici `GET /musteriler/123` gördüğünde ne yapacağını tahmin edebilir. Tutarlı, öngörülebilir ve dokümantasyonu kolay API'ler ortaya çıkar.

## Nasıl çalışır?

**REST prensipleri (sadeleştirilmiş)**
1. **Kaynak odaklı URL'ler:** URL'de isim (kaynak) olur, fiil olmaz. `/krediler`, ~~`/getKrediler`~~
2. **Metot = işlem:** Ne yapılacağını HTTP metodu söyler.
3. **Durumsuz:** Her istek gereken her bilgiyi (kimlik dahil) taşır.
4. **Standart durum kodları:** Sonuç HTTP koduyla bildirilir ([7.2](02-http.md)).
5. **Temsil:** Kaynak, istemciye bir temsil (genellikle JSON) olarak gönderilir.

**Tasarım örneği: kredi başvuruları**

| İşlem | Metot ve URL | Başarılı cevap |
|---|---|---|
| Listele | `GET /basvurular?durum=beklemede&sayfa=2` | 200 + liste |
| Tek kayıt | `GET /basvurular/42` | 200 veya 404 |
| Oluştur | `POST /basvurular` | 201 + yeni kayıt |
| Tamamen güncelle | `PUT /basvurular/42` | 200 |
| Kısmen güncelle | `PATCH /basvurular/42` | 200 |
| Sil | `DELETE /basvurular/42` | 204 |
| Alt kaynak | `GET /musteriler/123/basvurular` | 200 |
| İşlem (fiil gerekiyorsa) | `POST /basvurular/42/onay` | 200 |

**İyi pratikler**
- Çoğul isimler: `/basvurular`
- Filtreleme, sıralama, sayfalama sorgu parametresiyle: `?durum=onay&sirala=-tarih&limit=50`
- **Versiyonlama:** `/v1/basvurular` → kırıcı değişiklikte `/v2/`
- Tutarlı hata gövdesi: `{"hata": "GECERSIZ_TUTAR", "mesaj": "Tutar 0'dan büyük olmalı"}`

**JSON**
```json
{
  "basvuru_id": 42,
  "musteri": {
    "id": 123,
    "segment": "KOBİ"
  },
  "tutar": 250000.0,
  "para_birimi": "TRY",
  "onaylandi": false,
  "teminatlar": ["ipotek", "kefalet"],
  "olusturma_zamani": "2026-10-13T09:30:00+03:00",
  "not": null
}
```

| JSON tipi | Python karşılığı |
|---|---|
| object `{}` | `dict` |
| array `[]` | `list` |
| string `"..."` | `str` |
| number | `int` / `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

**Dikkat edilecekler**
- JSON'da **tarih tipi yoktur**; ISO 8601 metni kullanın: `2026-10-13T09:30:00+03:00`.
- **Para tutarları:** Float yuvarlama hatalarına açıktır. Hassas işlerde tutarlar metin (`"250000.00"`) veya kuruş cinsinden tamsayı olarak taşınır.
- Türkçe karakterler UTF-8 ile sorunsuz taşınır; Python'da `json.dumps(veri, ensure_ascii=False)`.

## Mini örnek
```python
import json
import requests

# JSON gönder ve al
yanit = requests.post(
    "https://api.sirket.local/v1/basvurular",
    json={"musteri_id": 123, "tutar": 250000},   # json= otomatik Content-Type ekler
    timeout=10,
)
print(yanit.status_code)          # 201
basvuru = yanit.json()

# Dosyaya yazma / okuma
with open("basvuru.json", "w", encoding="utf-8") as f:
    json.dump(basvuru, f, ensure_ascii=False, indent=2)

# pandas ile
import pandas as pd
df = pd.json_normalize(basvuru)    # iç içe JSON'ı düz tabloya çevir
```

Terminalde JSON incelemek için **jq**:
```bash
curl -s https://api.sirket.local/v1/basvurular | jq '.[] | {id: .basvuru_id, tutar}'
```

## Nerede kullanılır?
- Kurum içi servisler arası iletişim
- ML model skorlama API'leri
- Dış veri kaynakları (TCMB EVDS gibi açık veri API'leri JSON döndürür)

## Benzerleri ve farkları
| Format | Özellik | Nerede? |
|---|---|---|
| **JSON** | Okunabilir, yaygın | REST API'ler |
| **XML** | Daha ağır, şema (XSD) desteği | SOAP, eski kurumsal ve bankacılık sistemleri |
| **YAML** | Okunabilir, yorum destekli | Konfigürasyon (Kubernetes) |
| **Protobuf / Avro** | İkili, şemalı, hızlı | gRPC, Kafka |
| **CSV / Parquet** | Tablo verisi | Veri aktarımı, analitik |

- **SOAP:** XML tabanlı, sıkı sözleşmeli (WSDL) eski web servis standardı. Bankacılıkta hâlâ yaygın.
- REST'in alternatifleri için: [7.6](06-diger-yaklasimlar.md).

## İlgili sayfalar
- [7.2 HTTP](02-http.md)
- [7.4 FastAPI](04-fastapi.md)
- [17.3 JSON ve JSON Lines](../17-formatlar/03-json.md)
