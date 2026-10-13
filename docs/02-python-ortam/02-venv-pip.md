# 2.2 venv ve pip

## Nedir?
- **venv:** Python'un içinde hazır gelen sanal ortam modülü. Ek kurulum gerektirmez.
- **pip:** Python'un standart paket yükleyicisi. Paketleri varsayılan olarak **PyPI**'dan (Python Package Index) indirir.

## Hangi problemi çözer?
En sade ve her yerde bulunan araç ikilisidir. Ek bir araca ihtiyaç duymadan izole ortam kurup paket yönetmeyi sağlar.

## Nasıl çalışır?
- `python -m venv .venv` → `.venv/` klasörü oluşur (`bin/python`, `bin/pip`, `lib/.../site-packages`).
- `pip install paket` → paket ve bağımlılıkları PyPI'dan indirilir, aktif ortamın `site-packages` klasörüne kurulur.
- pip paketleri genellikle hazır derlenmiş **wheel** (`.whl`) dosyaları olarak indirir; wheel yoksa kaynak koddan derlemeye çalışır (bu bazen C derleyicisi gerektirir).

## Mini örnek

**Ortam**
```bash
python3 -m venv .venv
source .venv/bin/activate          # Linux/macOS
.venv\Scripts\activate             # Windows
python -m pip install --upgrade pip
```

**Paket yönetimi**
```bash
pip install pandas                  # en son sürüm
pip install "pandas==2.2.2"         # belirli sürüm
pip install "sqlalchemy>=2.0,<3"    # sürüm aralığı
pip install -r requirements.txt     # listeden kur
pip install -e .                    # kendi projeni düzenlenebilir modda kur

pip list                            # kurulu paketler
pip show pandas                     # paket detayı ve bağımlılıkları
pip list --outdated                 # güncellemesi olanlar
pip uninstall pandas
pip freeze > requirements.txt       # tam sürümlerle listeyi kaydet
```

**Ortamı kullanmadan doğrudan çalıştırmak (cron, script)**
```bash
/opt/etl/.venv/bin/python bddk_etl.py
```

## Nerede kullanılır?
- Basit projeler, script'ler, eğitim materyalleri
- Docker image'larında (`RUN pip install -r requirements.txt`)
- Sunucuda çalışan ETL'ler

## Benzerleri ve farkları
- **pip ile conda:** pip sadece Python paketlerini kurar; conda ayrıca Python sürümünü ve sistem kütüphanelerini de yönetir.
- **pip ile uv:** uv, pip ve venv'in yaptığını çok daha hızlı yapar ve kilit dosyası üretir ([2.4](04-uv-poetry.md)).
- **pipx:** Komut satırı araçlarını (ör. `black`, `ruff`) her biri kendi ortamında, global olarak kurmak için.

**Sık hatalar:**
- Ortamı aktive etmeyi unutmak → paket sistem Python'una gider. `which python` ile kontrol edin.
- `pip` ile `python` farklı Python'lara ait olabilir. En güvenlisi `python -m pip install ...` yazmaktır.
- `pip freeze` her şeyi (dolaylı bağımlılıklar dahil) yazar; liste uzun ve okunaksız olabilir. Doğrudan bağımlılıkları ayrı tutmak için [2.5](05-bagimlilik-dosyalari.md)'e bakın.

## İlgili sayfalar
- [2.1 Neden sanal ortam](01-neden-sanal-ortam.md)
- [2.6 Kurum içi paket deposu](06-kurum-ici-paket.md)
