# 0.3 Sözlük

Kitapta sık geçen terimlerin kısa açıklamaları. Ayrıntılı anlatım için parantez içindeki bölüme bakın.

## Genel ve altyapı
| Terim | Açıklama |
|---|---|
| **Sunucu (server)** | Sürekli açık duran ve başka bilgisayarlara hizmet veren makine. |
| **Shell / terminal** | Komut yazarak işletim sistemiyle konuştuğumuz arayüz. (1) |
| **Ortam (environment)** | Kodun çalıştığı yer: dev (geliştirme), test, prod (canlı). |
| **Bağımlılık (dependency)** | Kodun çalışması için gereken başka paket veya kütüphane. (2) |
| **Sanal ortam (virtual env)** | Bir projeye özel, izole Python paket alanı. (2) |
| **Port** | Bir makinedeki servislerin ayrıldığı numaralı kapılar (ör. 80, 443, 5432). |
| **Proxy** | İstekleri başka bir yere ileten aracı sunucu. (8) |

## Kod ve versiyon
| Terim | Açıklama |
|---|---|
| **Repo (repository)** | Bir projenin tüm dosyalarını ve geçmişini tutan depo. (3) |
| **Commit** | Değişikliklerin kaydedilmiş bir anlık görüntüsü. (3) |
| **Branch (dal)** | Ana koddan ayrılıp bağımsız ilerleyen çalışma kolu. (3) |
| **Merge (birleştirme)** | İki dalın değişikliklerini tek dalda toplamak. (3) |
| **Pull / Merge Request** | Bir dalın ana dala alınması için açılan inceleme talebi. (3.6) |

## Container ve orkestrasyon
| Terim | Açıklama |
|---|---|
| **Image** | Uygulamanın ve bağımlılıklarının dondurulmuş, değişmez paketi. (4) |
| **Container** | Bir image'ın çalışan örneği. (4) |
| **Registry** | Image'ların saklandığı depo (Docker Hub, Quay, Harbor). (4.7) |
| **Volume** | Container silinse de kalan kalıcı veri alanı. (4.5) |
| **Orkestrasyon** | Çok sayıda container'ın otomatik yönetimi. (5) |
| **Cluster** | Birlikte yönetilen makineler (node'lar) topluluğu. (5.2) |
| **Pod** | Kubernetes'te çalışan en küçük birim; bir veya birkaç container. (5.3) |
| **Namespace / Project** | Cluster içinde mantıksal bölme; OpenShift'te Project denir. (5.3) |

## Teslimat
| Terim | Açıklama |
|---|---|
| **CI** | Her değişiklikte otomatik test ve build. (6) |
| **CD** | Test edilen sürümün otomatik olarak ortamlara dağıtılması. (6) |
| **Pipeline** | CI/CD adımlarının sıralı tanımı. (6.2) |
| **Deploy** | Yeni sürümü bir ortama yayınlamak. |
| **Rollback** | Sorunlu sürümden önceki sürüme geri dönmek. |
| **API** | Programların birbirleriyle konuştuğu tanımlı arayüz. (7) |
| **Endpoint** | Bir API'deki belirli adres, ör. `/kredi/ozet`. (7) |
| **Reverse proxy** | Kullanıcı isteğini karşılayıp arkadaki uygulamaya ileten sunucu. (8.4) |
| **TLS / HTTPS** | Trafiği şifreleyen protokol. (8.6) |

## Veri
| Terim | Açıklama |
|---|---|
| **OLTP** | Anlık işlem odaklı sistemler (core banking). (9.1) |
| **OLAP** | Analiz odaklı sistemler (warehouse). (9.1) |
| **ETL / ELT** | Veriyi çekme, dönüştürme ve yükleme sırası. (11.1) |
| **Schema-on-write / read** | Şemanın yazarken mi okurken mi uygulandığı. (9.11) |
| **Lakehouse** | Data lake esnekliği ile warehouse düzenini birleştiren mimari. (9.12) |

## İlgili sayfalar
- [0.1 Koddan canlıya](01-koddan-canliya.md)
- [0.2 Veriden rapora](02-veriden-rapora.md)
- [16.1 Dosya ve veri formatları](../16-formatlar/01-genel-bakis.md)
