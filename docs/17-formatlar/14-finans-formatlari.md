# 17.14 Bankacılık ve Finans Formatları: Sabit Uzunluklu Dosyalar, EBCDIC, ISO 20022, SWIFT MT, XBRL

## Nedir?
Bankacılıkta genel amaçlı formatların (CSV, JSON, Parquet) yanında, sektöre özgü ve çoğu onlarca yıllık geçmişi olan formatlarla karşılaşılır:

| Format | Ne? | Nerede? |
|---|---|---|
| **Sabit uzunluklu (fixed-width) dosya** | Her alanın satırda belirli bir konumda ve uzunlukta olduğu metin dosyası; ayraç yok | Ana bilgisayar (mainframe) çıktıları, kurumlar arası toplu dosyalar, eski çekirdek bankacılık arayüzleri |
| **EBCDIC + COBOL copybook** | IBM ana bilgisayar karakter kodlaması ve dosya düzenini tanımlayan COBOL yapısı | Mainframe çekirdek bankacılık sistemleri |
| **ISO 20022 (MX)** | Ödeme ve finansal mesajlar için XML tabanlı uluslararası standart | SWIFT, SEPA, merkez bankası ödeme sistemleri, hesap ekstreleri |
| **SWIFT MT** | Etiketli (`:20:`, `:32A:`) metin tabanlı eski SWIFT mesajları | Sınır ötesi ödemeler (ISO 20022'ye geçiş), ekstreler |
| **XBRL** | Finansal tabloları makinece okunabilir, etiketli biçimde raporlamak için XML tabanlı dil | Halka açık şirketlerin finansal raporları, düzenleyici raporlama |
| **FIX** | Menkul kıymet alım-satım emirleri için `etiket=değer` protokolü | Aracı kurumlar, borsalar |

## Hangi problemi çözer?
- Eski ama hâlâ kritik sistemlerden (çekirdek bankacılık, kart sistemleri, takas) veri almak
- Kurumlar arası mesajlaşmada ortak dil: ödeme talimatı, hesap hareketi, iade nedeni gibi bilgilerin herkesçe aynı anlaşılması
- Finansal tabloları elle kopyalamadan, kalem kalem analiz edilebilir hâlde almak (XBRL)
- Bu formatları doğru okuyamamak; yanlış tutar (ondalık kayması), bozuk karakter ve kayıp kayıt demektir

## Nasıl çalışır?

### Sabit uzunluklu dosyalar
Her satır aynı uzunluktadır; alanlar konumlarıyla tanımlanır. Düzen genellikle ayrı bir **kayıt düzeni (record layout)** belgesinde verilir.
```
HD202609300000BANKA ADI                                      
DT0000012345678ŞAHİN TİCARET A.Ş.            000001250000TRY+
DT0000098765432YILMAZ GIDA LTD.              000000087550TRY-
TR00000000020000000001337550                                 
```
| Alan | Başlangıç | Uzunluk | Açıklama |
|---|---|---|---|
| Kayıt tipi | 1 | 2 | `HD` başlık, `DT` detay, `TR` kapanış (trailer) |
| Müşteri no | 3 | 13 | Başında sıfırlarla doldurulmuş |
| Unvan | 16 | 30 | Sağdan boşlukla doldurulmuş |
| Tutar | 46 | 12 | **Ondalık ayırıcı yok**; son 2 hane kuruş (`000001250000` = 12.500,00) |
| Para birimi | 58 | 3 | |
| İşaret | 61 | 1 | `+` / `-` |

**Dikkat noktaları**
- **Örtük ondalık (implied decimal):** Tutar alanında virgül yoktur; kaç hanenin ondalık olduğu belgede yazar. Yanlış varsayım 100 kat hata demektir.
- **Başlık ve kapanış kayıtları:** Kapanış kaydındaki kayıt sayısı ve toplam tutar, dosyanın eksiksiz geldiğini doğrulamak için **mutabakat kontrolü** olarak kullanılmalıdır ([11.5](../11-veri-akisi/05-veri-kalitesi.md)).
- **Konum karakter mi, bayt mı?** Tek baytlık kodlamada (cp1254, EBCDIC) aynıdır; dosya UTF-8'e çevrilmişse Türkçe karakterli satırlarda bayt konumları kayar ([17.11](11-karakter-kodlamasi.md)).
- **Boşluk doldurma:** Metin alanlarında sondaki boşluklar, sayısal alanlarda baştaki sıfırlar temizlenmelidir; ama müşteri/hesap numarası gibi alanlarda baştaki sıfırlar **anlamlıdır**, sayıya çevrilmemelidir.

### EBCDIC ve COBOL copybook
Ana bilgisayar dosyaları ASCII/UTF-8 değil **EBCDIC** kodlamasındadır (Türkçe: `cp1026`). Dosya düzeni COBOL **copybook** ile tanımlanır:
```cobol
       01  HESAP-KAYDI.
           05  HESAP-NO         PIC X(16).
           05  MUSTERI-ADI      PIC X(30).
           05  BAKIYE           PIC S9(13)V99 COMP-3.
           05  ACILIS-TARIHI    PIC 9(8).
```
| COBOL tanımı | Anlamı |
|---|---|
| `PIC X(16)` | 16 karakterlik metin |
| `PIC 9(8)` | 8 haneli sayı (metin olarak saklanır, ör. `20260930`) |
| `S9(13)V99` | İşaretli, 13 tam + 2 ondalık hane; `V` örtük ondalık noktası |
| `COMP-3` | **Packed decimal:** her baytta iki rakam, son yarım bayt işaret; metin olarak okunamaz, özel çözümleme gerekir |
| `COMP` / `BINARY` | İkili tamsayı |

EBCDIC dosyaları **ikili (binary) modda** aktarılmalıdır; FTP'nin "ASCII mod" dönüşümü COMP-3 alanlarını bozar. Pratikte bu dosyalar çoğu zaman mainframe tarafında metne çevrilip gönderilir; doğrudan okumak gerekiyorsa Spark için **Cobrix** gibi copybook okuyabilen kütüphaneler kullanılır.

### ISO 20022 (MX mesajları)
XML tabanlı, zengin ve yapılandırılmış veri taşıyan mesaj standardıdır. Mesaj türleri iş alanına göre kodlanır:
| Mesaj | Anlamı | Eski MT karşılığı |
|---|---|---|
| `pain.001` | Müşteriden bankaya ödeme talimatı (toplu ödeme dosyası) | — (MT101) |
| `pacs.008` | Bankalar arası müşteri adına kredi transferi | MT103 |
| `pacs.009` | Bankalar arası finansal kurum transferi | MT202 |
| `pacs.004` | İade | — |
| `camt.053` | Gün sonu hesap ekstresi | MT940 |
| `camt.054` | Hesaba borç/alacak bildirimi | MT942 / MT900 / MT910 |

SWIFT, sınır ötesi ödeme talimatlarında MT ile ISO 20022'nin birlikte kullanıldığı geçiş dönemini **22 Kasım 2025**'te sonlandırdı; bu tarihten sonra ödeme mesajlarının ISO 20022 formatında gönderilmesi bekleniyor. Ekstre ve bildirim mesajları gibi bazı kategorilerin geçiş takvimi farklı olabilir. Türkiye'de ve dünyada merkez bankalarının ödeme sistemleri de ISO 20022'ye geçmekte veya geçmiş bulunmaktadır; kurumunuzdaki sistemin hangi mesaj sürümünü kullandığını ilgili teknik dokümandan doğrulayın.

```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:pacs.008.001.08">
  <FIToFICstmrCdtTrf>
    <GrpHdr>
      <MsgId>BNK20261024-000123</MsgId>
      <CreDtTm>2026-10-24T10:15:00+03:00</CreDtTm>
      <NbOfTxs>1</NbOfTxs>
    </GrpHdr>
    <CdtTrfTxInf>
      <PmtId><EndToEndId>FATURA-2026-0912</EndToEndId></PmtId>
      <IntrBkSttlmAmt Ccy="EUR">12500.00</IntrBkSttlmAmt>
      <Dbtr><Nm>SAHIN TICARET AS</Nm></Dbtr>
      <DbtrAcct><Id><IBAN>TR330006100519786457841326</IBAN></Id></DbtrAcct>
      <Cdtr><Nm>MUSTER GMBH</Nm></Cdtr>
      <RmtInf><Ustrd>Fatura 2026-0912</Ustrd></RmtInf>
    </CdtTrfTxInf>
  </FIToFICstmrCdtTrf>
</Document>
```
Avantajı: Alıcı/gönderen adı, adresi, ödeme amacı, açıklama gibi bilgiler ayrı ve yapılandırılmış alanlarda gelir; uyum (AML/yaptırım taraması) ve otomatik mutabakat kolaylaşır. Mesajlar **XSD şemasıyla** doğrulanır ([17.9](09-digerleri.md)).

### SWIFT MT
```
{1:F01BANKTRISAXXX0000000000}{2:I103BANKDEFFXXXXN}{4:
:20:BNK20261024-000123
:32A:261024EUR12500,00
:50K:/TR330006100519786457841326
SAHIN TICARET AS
:59:/DE89370400440532013000
MUSTER GMBH
:70:FATURA 2026-0912
:71A:SHA
-}
```
`:32A:` alanı tarih (YYMMDD), para birimi ve tutarı (ondalık ayırıcı virgül) birlikte taşır. Serbest metin alanları sınırlı karakter kümesindedir (Türkçe karakter yoktur) ve uzunluk sınırlıdır; bu, ISO 20022'ye geçişin nedenlerinden biridir.

### XBRL
**XBRL (eXtensible Business Reporting Language)**, finansal tablo kalemlerini bir **taksonomiye** (kalemlerin tanımlandığı sözlük; ör. TFRS/IFRS taksonomisi) göre etiketleyen XML tabanlı dildir. Her değer; hangi kalem olduğu, hangi dönem, hangi şirket ve hangi para birimiyle birlikte gelir. Türkiye'de KAP (Kamuyu Aydınlatma Platformu), halka açık şirketlerin ve fonların finansal tablolarını XBRL tabanlı olarak almaktadır.
```xml
<ifrs-full:Revenue contextRef="FY2025" unitRef="TRY" decimals="-3">1250000000</ifrs-full:Revenue>
```
**iXBRL (Inline XBRL):** Etiketlerin insan tarafından okunan HTML raporun içine gömüldüğü sürüm; aynı belge hem okunur hem makine tarafından işlenir.

### FIX (kısaca)
Alım-satım emirleri için `etiket=değer` çiftleri; ayraç SOH karakteridir (belgelerde `|` ile gösterilir):
```
8=FIX.4.4|35=D|49=ARACIKURUM|56=BORSA|11=EMIR123|55=THYAO|54=1|38=1000|40=2|44=312.50|
```
`35=D` yeni emir, `54=1` alış, `38` miktar, `44` fiyat.

## Mini örnek
**Sabit uzunluklu dosya: pandas**
```python
import pandas as pd

kolonlar = [(0, 2), (2, 15), (15, 45), (45, 57), (57, 60), (60, 61)]   # Python: 0 tabanlı, bitiş hariç
adlar = ["kayit_tipi", "musteri_no", "unvan", "tutar_ham", "para_birimi", "isaret"]

df = pd.read_fwf("hareket_20260930.txt", colspecs=kolonlar, names=adlar,
                 encoding="cp1254", dtype=str)               # hepsi metin: baştaki sıfırlar korunur

detay = df[df["kayit_tipi"] == "DT"].copy()
detay["unvan"] = detay["unvan"].str.strip()
detay["tutar"] = detay["tutar_ham"].astype("int64") / 100     # örtük 2 ondalık
detay.loc[detay["isaret"] == "-", "tutar"] *= -1

# Kapanış kaydıyla mutabakat
trailer = df[df["kayit_tipi"] == "TR"].iloc[0]
beklenen_adet = int(trailer["musteri_no"][:10])               # düzen belgesine göre konumlar
assert len(detay) == beklenen_adet, "Kayıt sayısı kapanış kaydıyla uyuşmuyor"
```
Para tutarlarında `float` yerine `Decimal` veya kuruş cinsinden tamsayı kullanmak yuvarlama hatalarını önler.

**EBCDIC metin alanını çözmek ve COMP-3 okumak**
```python
ham = open("hesap.dat", "rb").read()
metin = ham[0:16].decode("cp1026")                  # Türkçe EBCDIC

def comp3_coz(b: bytes, ondalik: int = 2):
    hex_ = b.hex()                                   # ör. '0000001250000c'
    isaret = -1 if hex_[-1] in "dD" else 1           # c/f pozitif, d negatif
    return isaret * int(hex_[:-1]) / 10**ondalik

comp3_coz(bytes.fromhex("0000001250000c"))           # 12500.0
```

**ISO 20022 ekstresini (camt.053) okumak**
```python
from lxml import etree

ns = {"c": "urn:iso:std:iso:20022:tech:xsd:camt.053.001.08"}
agac = etree.parse("ekstre_20261023.xml")
for ntry in agac.iterfind(".//c:Ntry", ns):
    tutar = ntry.findtext("c:Amt", namespaces=ns)
    yon = ntry.findtext("c:CdtDbtInd", namespaces=ns)        # CRDT / DBIT
    tarih = ntry.findtext("c:BookgDt/c:Dt", namespaces=ns)
    aciklama = ntry.findtext(".//c:RmtInf/c:Ustrd", namespaces=ns)
    print(tarih, yon, tutar, aciklama)

# Şema doğrulaması
sema = etree.XMLSchema(etree.parse("camt.053.001.08.xsd"))
sema.assertValid(agac)
```
Ad alanı (namespace) sürümü (`camt.053.001.08`) karşı tarafın kullandığı sürümle aynı olmalıdır; farklı sürümlerde alan adları değişebilir.

**XBRL'den finansal tablo kalemi çekmek**
```python
# pip install arelle-release  (XBRL işlemcisi)  veya basit dosyalar için lxml
from lxml import etree
x = etree.parse("finansal_rapor.xbrl")
for e in x.iter("{*}Revenue"):
    print(e.get("contextRef"), e.get("unitRef"), e.text)
```
Gerçek kullanımda bağlam (`context`) tanımları dönem ve konsolide/solo ayrımını içerir; taksonomiyi bilen bir işlemci (Arelle) daha güvenilir sonuç verir.

## Nerede kullanılır?
- Çekirdek bankacılık ve kart sistemlerinden gelen gün sonu dosyaları
- Ödeme sistemleri, SWIFT mesajları, kurumsal müşterilerin toplu ödeme dosyaları (`pain.001`)
- Hesap ekstresi entegrasyonları ve otomatik mutabakat (`camt.053`, MT940)
- AML ve yaptırım taraması için ödeme verisinin yapılandırılması
- Finansal tablo analizi: halka açık şirketlerin raporlarından kalem kalem veri çekmek (XBRL)

## Benzerleri ve farkları
- **CSV:** Ayraçlı; alan konumları değişebilir ama alan sırası ve ayraç sabittir ([17.2](02-csv.md)). Sabit uzunluklu dosyalarda ayraç yoktur, konum her şeydir.
- **Genel XML:** ISO 20022 ve XBRL, XML'in üzerine kurulmuş, şeması ve anlamı sabitlenmiş sektör dilleridir ([17.9](09-digerleri.md)).
- **JSON tabanlı açık bankacılık API'leri:** Yeni nesil entegrasyonlar (hesap bilgisi, ödeme başlatma) REST + JSON kullanır; ISO 20022 veri modelinden esinlenen alanlar içerebilir ([7.3](../07-api/03-rest-json.md)).

## İlgili sayfalar
- [17.11 Karakter kodlaması](11-karakter-kodlamasi.md)
- [17.9 Diğerleri: Avro, ORC, Arrow, XML, Excel](09-digerleri.md)
- [11.5 Veri kalitesi](../11-veri-akisi/05-veri-kalitesi.md)
- [Regex cheat sheet](../ekler/regex-cheatsheet.md)
