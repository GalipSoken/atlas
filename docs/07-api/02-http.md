# 7.2 HTTP: Metotlar, Durum Kodları, Header'lar

## Nedir?
**HTTP (HyperText Transfer Protocol)**, web'in temel iletişim protokolüdür. Tarayıcıların web sayfası istemesi de, programların API çağırması da HTTP ile yapılır. **HTTPS**, aynı protokolün TLS ile şifrelenmiş hâlidir ([8.6](../08-nginx/06-https-tls.md)).

## Hangi problemi çözer?
İstemci ve sunucunun "ne istendiği", "işlemin sonucu" ve "verinin biçimi" konusunda ortak bir dil konuşmasını sağlar. HTTP'yi anlamak, API hatalarını okumanın ve nginx/Ingress ayarlarını yapmanın temelidir.

## Nasıl çalışır?
Her HTTP etkileşimi bir **istek** (request) ve bir **cevaptan** (response) oluşur.

**İstek**
```http
POST /model/skor HTTP/1.1               ← metot, yol, sürüm
Host: api.sirket.local                  ← header'lar
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...

{"musteri_id": 123, "tutar": 50000}     ← gövde (body)
```

**Cevap**
```http
HTTP/1.1 200 OK                          ← durum kodu
Content-Type: application/json
X-Request-Id: 8f2c...

{"skor": 0.12, "karar": "onay"}
```

**Metotlar**

| Metot | Amaç | Gövde | Idempotent* |
|---|---|---|---|
| `GET` | Veri oku | Yok | Evet |
| `POST` | Yeni kayıt oluştur / işlem başlat | Var | Hayır |
| `PUT` | Kaydın tamamını değiştir | Var | Evet |
| `PATCH` | Kaydın bir kısmını değiştir | Var | Duruma göre |
| `DELETE` | Sil | Genelde yok | Evet |
| `HEAD` / `OPTIONS` | Sadece header / izin verilen metotlar | Yok | Evet |

\* **Idempotent:** Aynı isteği bir veya on kez göndermek aynı sonucu doğurur. Ağ hatasında güvenle tekrar denenebilir.

**Durum kodları**

| Aralık | Anlamı | Sık görülenler |
|---|---|---|
| **1xx** | Bilgi | 101 Switching Protocols (WebSocket) |
| **2xx** | Başarılı | 200 OK, 201 Created, 204 No Content |
| **3xx** | Yönlendirme | 301 Kalıcı taşındı, 302 Geçici, 304 Değişmedi |
| **4xx** | **İstemci hatası** | 400 Bad Request, 401 Unauthorized (kimlik yok), 403 Forbidden (yetki yok), 404 Not Found, 409 Conflict, 413 Payload Too Large, 422 Unprocessable Entity (doğrulama hatası), 429 Too Many Requests |
| **5xx** | **Sunucu hatası** | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

Kural: **4xx → isteği düzelt; 5xx → sunucu tarafında sorun var.** 502/504 genellikle proxy (nginx, Ingress) ile arkadaki uygulama arasındaki sorunu gösterir ([8.8](../08-nginx/08-log-hatalar.md)).

**Önemli header'lar**

| Header | Görevi |
|---|---|
| `Host` | Hangi alan adına istek yapıldığı (bir sunucuda birçok site) |
| `Content-Type` | Gövdenin biçimi: `application/json`, `text/csv` |
| `Accept` | İstemcinin kabul ettiği biçim |
| `Authorization` | Kimlik bilgisi: `Bearer <token>`, `Basic ...` |
| `Cookie` / `Set-Cookie` | Oturum bilgisi |
| `Cache-Control` | Önbellek kuralları |
| `User-Agent` | İstemci yazılımı |
| `X-Forwarded-For` | Proxy arkasında gerçek istemci IP'si |
| `X-Request-Id` | İsteği loglarda takip etmek için kimlik |

**URL yapısı**
```
https://api.sirket.local:443/kredi/ozet?donem=2026-09&tur=kobi#bolum
└─┬─┘  └──────┬───────┘└┬┘└────┬─────┘└──────────┬─────────┘└──┬─┘
şema        alan adı   port   yol         sorgu parametreleri   parça
```

## Mini örnek
```bash
curl -i https://api.sirket.local/health            # -i: header'ları da göster
curl -v https://api.sirket.local/health            # -v: tüm konuşmayı göster
curl -X POST https://api.sirket.local/model/skor \
     -H "Content-Type: application/json" \
     -d '{"musteri_id": 123, "tutar": 50000}'
curl -o /dev/null -s -w "%{http_code} %{time_total}s\n" https://api.sirket.local/health
```

## Nerede kullanılır?
- API tasarlarken doğru metot ve durum kodunu seçmek
- Hata ayıklarken: 401 mi 403 mü? 502 mi 504 mü?
- nginx ve Ingress ayarlarında header ve boyut sınırları

## Benzerleri ve farkları
- **HTTP/1.1, HTTP/2, HTTP/3:** Anlamlar (metot, kod, header) aynıdır; sonraki sürümler performansı artırır (tek bağlantıda çoklu istek, QUIC).
- **WebSocket:** HTTP ile başlayıp sürekli açık, iki yönlü bağlantıya dönüşür (canlı dashboard güncellemeleri, Streamlit bunu kullanır).
- **gRPC:** HTTP/2 üzerinde, ikili (binary) formatla çalışan yüksek performanslı alternatif ([7.6](06-diger-yaklasimlar.md)).

## İlgili sayfalar
- [7.3 REST ve JSON](03-rest-json.md)
- [8.8 nginx hataları](../08-nginx/08-log-hatalar.md)
