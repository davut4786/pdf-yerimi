# PDF Yer İmi — Android Uygulaması

PDF yükler → içindeki **gerçek highlight/underline annotasyonlarını** otomatik bulur →
bunları yer imi listesi olarak gösterir (sürükleyip sıralayabilir, sola kaydırıp silebilirsiniz) →
"Word Oluştur ve İndir" butonuyla düzenli bir **.docx** dosyası üretip cihaza indirir.

## Nasıl açılır

1. Android Studio'yu aç → **Open** → bu klasörü (`PdfYerimi`) seç.
2. Gradle sync bitene kadar bekle (ilk seferde internetten bağımlılıkları indirir,
   `pdfbox-android` dahil).
3. Bir emülatör veya gerçek cihaz seç, **Run ▶** ile çalıştır.
4. minSdk 24 (Android 7.0) ve üzeri her cihazda çalışır.

## Nasıl çalışıyor

- **PDF Yükle**: `ACTION_OPEN_DOCUMENT` ile sistem dosya seçiciyi açar, herhangi bir
  depolama izni istemez.
- **Vurgu tespiti** (`PdfHighlightExtractor.kt`): `pdfbox-android` kütüphanesiyle PDF'i
  açar, her sayfadaki annotasyonları tarar; subtype'ı `Highlight` veya `Underline` olanları
  bulur, annotasyonun `QuadPoints` koordinatlarını kullanarak o bölgedeki metni çıkarır.
  **Önemli**: Bu, PDF içine gömülü *gerçek* vurgu/altı çizgi annotasyonlarını yakalar
  (Adobe Reader, Xodo, çoğu PDF okuyucu ile eklenen türden). PDF'de böyle bir annotasyon
  yoksa (örneğin sadece görsel olarak taranmış bir sayfa) hiçbir şey bulunmaz.
- **Liste ekranı**: `RecyclerView` + `ItemTouchHelper` ile yukarı/aşağı sürükleyerek
  sırayı değiştirebilir, sola kaydırarak istemediğiniz bir yer imini silebilirsiniz.
- **Word oluşturma** (`DocxBuilder.kt`): Apache POI gibi ağır bir kütüphaneye ihtiyaç
  duymadan, geçerli bir `.docx` (OOXML) dosyasını doğrudan zip+XML olarak üretir. Her
  yer imi, "N. Sayfa X (Vurgu/Alt Çizgi)" başlığı ve altında metniyle ayrı bir paragraf
  olarak yazılır — yani alt alta, düzenli bir liste.
- **İndirme**: `ACTION_CREATE_DOCUMENT` ile sistem "farklı kaydet" ekranını açar; kullanıcı
  isterse doğrudan **İndirilenler (Downloads)** klasörünü seçebilir. Bu, ayrıca bir
  depolama izni gerektirmeyen, Android'in önerdiği güncel yoldur.

## Sınırlamalar / geliştirilebilecek noktalar

- Üretilen `.docx` şu an sade paragraflardan oluşuyor (başlık + metin). İstersen sayfa
  numaralarını tablo halinde, ya da her vurguyu kendi renginde göstermek gibi ek
  biçimlendirmeler ekleyebilirim.
- Bazı PDF üreticileri (özellikle bazı tarayıcı eklentileri) `QuadPoints` alanını standart
  dışı sırada yazabiliyor; bu nadir durumda çıkarılan metin küçük kaymalar gösterebilir.
- Şu an yer imleri sadece o oturumda (uygulama kapanana kadar) hafızada tutuluyor.
  İstersen bir sonraki adımda Room veritabanı ekleyip PDF geçmişini kalıcı hale getirebiliriz.

## Paket yapısı

```
app/src/main/java/com/davee/pdfyerimi/
  PdfYerimiApp.kt            -> PDFBox başlatma
  MainActivity.kt            -> ekran akışı, dosya seçiciler
  PdfHighlightExtractor.kt   -> PDF'ten vurgu/altı çizgi + metin çıkarma
  DocxBuilder.kt             -> .docx üretimi
  Bookmark.kt                -> veri modeli
  BookmarkAdapter.kt         -> liste (RecyclerView) adaptörü
app/src/main/res/layout/     -> ekran ve liste öğesi XML'leri
```
