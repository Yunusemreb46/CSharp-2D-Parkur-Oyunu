# CSharp-2D-Parkur-Oyunu
C# (.NET 10) ve Avalonia UI DrawingContext altyapısıyla nesne yönelimli (OOP) ve olay güdümlü mimaride geliştirilmiş; yerçekimi, çarpışma algılama, hareketli platformlar ve eğilme mekaniği içeren 10 bölümlük 2D parkur oyunu.



# C# 2D Parkur Oyunu (10 Bölüm — Avalonia UI)

Harici bir oyun motoruna (Unity, Godot vb.) veya elektronik donanıma ihtiyaç duymadan, saf **C# (.NET 10)** ve **Avalonia UI** grafik altyapısı kullanılarak geliştirilmiş; 60 FPS özel 2D fizik ve çarpışma motoruna sahip, klavye (`W`, `A`, `S`, `D`, `Space`, `R`) ile oynanan 10 bölümlük masaüstü platform/parkur oyunudur.

---

## 1. Çalışma Prensibi ve Mimari Yapı

Uygulama, **Nesne Yönelimli Programlama (OOP)** ve **Olay Güdümlü (Event-Driven)** masaüstü yazılım mimarisi üzerine inşa edilmiştir:

1. **Tek Merkezli Modüler Sınıf Mimarisi (`MainWindow.axaml.cs`):**
   * **`Oyuncu` Sınıfı:** Karakterin konum (`X`, `Y`), hız vektörleri (`HizX`, `HizY`), yön durumu (`SagaBakiyor`), zemin teması (`YerdeMi`) ve eğilme (`EgiliyorMu`) durumlarını kapsüller. Normal boyu `36 px` iken `S` tuşuna basıldığında çarpışma kutusu (`Rect`) dinamik olarak `22 px` yüksekliğe düşer.
   * **`Platform` ve `HareketliPlatform` Sınıfları (Kalıtım):** `HareketliPlatform`, temel `Platform` sınıfından miras alır. Belirlenen `MinX/MaxX` veya `MinY/MaxY` sınırları arasında sabit hızla salınım yapar ve her karedeki konum farkını (`SonDeltaX`) üzerindeki oyuncuya aktararak karakterin platformla birlikte taşınmasını sağlar.
   * **`Tehlike` Sınıfı:** Kırmızı lav havuzlarını (`LavMi = true`) ve üçgen formundaki sabit diken tuzaklarını (`LavMi = false`) dikdörtgen çarpışma alanlarıyla tanımlar.
   * **`BolumVerisi` ve `BolumYoneticisi` Sınıfları:** 1'den 10'a kadar tüm bölümlerin başlangıç koordinatlarını, çıkış portalını, sabit/hareketli platform dizilimlerini ve ipucu metinlerini yönetir.
2. **60 FPS Oyun Döngüsü (`DispatcherTimer`):**
   * `MainWindow` başlatıldığında `16 ms` aralıklı (`~60 FPS`) bir zamanlayıcı döngüsü (`OyunAdimi`) devreye girer. Her karede sırasıyla hareketli platformlar güncellenir, basılı tuşlar (`HashSet<Key>`) okunur, yerçekimi (`0.65`) uygulanır, yatay/dikey çarpışmalar çözülür ve sahne yeniden çizilir (`InvalidateVisual`).
3. **2D Vektörel Çizim Motoru (`OyunAlani : Control`):**
   * Hazır görsel (sprite) dosyaları yerine Avalonia'nın donanım hızlandırmalı `DrawingContext` sınıfı kullanılır. Arka plan ızgarası, platformlar, üçgen dikenler (`StreamGeometry`), çıkış portalı ve üst bilgi paneli (HUD) her karede doğrudan matematiksel koordinatlarla çizilir.
4. **AABB Çarpışma Algılama (`Rect.Intersects`):**
   * Oyuncunun dikdörtgen sınır kutusu (`Oyuncu.Alan`) ile platformlar, tehlikeler ve çıkış portalı arasındaki kesişimler eksen bazlı (önce X ekseni, sonra Y ekseni) kontrol edilir. Üstünde engel varken `S` tuşu bırakılırsa karakterin sıkışmaması için tavan kontrolü (`PlatformlaCakisiyorMu`) yapılır.

---

## 2. Gereksinimler ve Kullanılan Kütüphaneler

Projeyi derlemek ve çalıştırmak için bilgisayarınızda **.NET 10.0 SDK** ve **Visual Studio Code** (C# Dev Kit eklentisi ile) kurulu olmalıdır.

* **NuGet Paket Bağımlılıkları (`ParkurOyunu.csproj` içinde tanımlıdır, `dotnet restore` ile otomatik yüklenir):**
  * `Avalonia` (`11.2.5`): Çapraz platform kullanıcı arayüzü ve `DrawingContext` 2D çizim çekirdeği.
  * `Avalonia.Desktop` (`11.2.5`): Windows, Linux ve macOS pencere ve klavye olay yönetim altyapısı.
  * `Avalonia.Themes.Fluent` (`11.2.5`): Karanlık tema (`FluentTheme`) bileşenleri.
  * `Avalonia.Fonts.Inter` (`11.2.5`): Oyun içi HUD ve bilgilendirme metinleri için yazı tipi paketi.
  * `AvaloniaUI.DiagnosticsSupport` (`2.1.1`): Geliştirme aşaması arayüz hata ayıklama desteği.
* **.NET Standart Sınıf Kütüphaneleri (SDK ile hazır gelir):**
  * `System.Collections.Generic`: Basılı tuşların (`HashSet<Key>`) ve bölüm nesnelerinin (`List<T>`) yönetimi.
  * `System.Globalization`: Ekrana çizilen metinlerin (`FormattedText`) kültür ve yönlendirme ayarları.
  * `Avalonia.Threading`: 60 FPS oyun döngüsünü yürüten `DispatcherTimer` zamanlayıcısı.

---

## 3. Adım Adım Kurulum ve Çalıştırma

Depo boyutunu küçük ve temiz tutmak amacıyla arşiv içerisinde geçici derleme klasörleri (`bin/` ve `obj/`) yer almamaktadır. Kurulum adımları uygulandığında **.NET SDK** bu klasörleri otomatik olarak oluşturur.

### Adım 1: Proje Dosyalarının İndirilmesi ve Çıkarılması
1. Depo sayfasının sağ üst köşesindeki **`<> Code`** butonuna tıklayıp **`Download ZIP`** seçeneği ile `ParkurOyunu.zip` arşivini bilgisayarınıza indirin.
2. İndirdiğiniz ZIP dosyasına sağ tıklayıp **Tümünü Ayıkla (Extract All)** seçeneği ile dosyaları yerel bir klasöre çıkartın.

### Adım 2: Klasörün Visual Studio Code ile Açılması
1. **Visual Studio Code** uygulamasını açın.
2. Üst menüden **File > Open Folder...** (`Ctrl + K Ctrl + O`) yolunu izleyerek arşivden çıkardığınız **`ParkurOyunu`** klasörünü seçin.
3. Sol taraftaki **Explorer** panelinde kaynak dosyaların (`ParkurOyunu.csproj`, `Program.cs`, `App.axaml`, `App.axaml.cs`, `MainWindow.axaml`, `MainWindow.axaml.cs`, `app.manifest`) listelendiğini doğrulayın.

### Adım 3: Paketlerin Geri Yüklenmesi (`dotnet restore`)
1. Visual Studio Code üst menüsünden **Terminal > New Terminal** (`Ctrl + "`) seçeneği ile terminali açın.
2. Terminal satırına aşağıdaki komutu yazıp `Enter` tuşuna basarak Avalonia UI paketlerini projeye indirin:
   ```bash
   dotnet restore
   ```

### Adım 4: Derleme Klasörlerinin Oluşturulması ve Oyunun Başlatılması (`dotnet run`)
1. Paket yüklemesi tamamlandıktan sonra proje klasöründe **`bin/`** ve **`obj/`** derleme klasörlerinin oluşması ve oyunun başlatılması için aynı terminalde şu komutu çalıştırın:
   ```bash
   dotnet run
   ```
2. **Çalışma Aşaması:** Komut çalıştırıldığında .NET SDK eksik olan `bin/` ve `obj/` klasörlerini otomatik olarak üretir, kodu derler ve **`900x540`** boyutundaki karanlık temalı oyun penceresini ekranda açar.

---

## 4. Proje Dosya Yapısı

```text
ParkurOyunu/
├── ParkurOyunu.csproj   # .NET 10 hedef çerçevesi ve Avalonia NuGet paket tanımları
├── Program.cs           # Uygulama giriş noktası (Main metodu ve Avalonia başlatıcısı)
├── App.axaml            # Uygulama geneli Karanlık Tema (Dark) ve FluentTheme tanımı
├── App.axaml.cs         # Uygulama yaşam döngüsü ve MainWindow pencere oluşturucusu
├── MainWindow.axaml     # 900x540 oyun penceresi ve OyunAlani çizim tuvali tanımı
├── MainWindow.axaml.cs  # Fizik motoru, çarpışma kontrolleri, 2D çizim ve 10 bölüm verisi
├── app.manifest         # Windows uygulama uyumluluk manifestosu
└── README.md            # Proje dokümantasyonu
```
*(Not: `bin/` ve `obj/` klasörleri `dotnet run` komutu çalıştırıldıktan sonra proje dizininde otomatik olarak oluşacaktır.)*

---

## 5. Oyun Kontrolleri ve 10 Bölüm Akışı

Pencere açıldığında **Bölüm 1: Başlangıç Adımları** doğrudan başlar. Her bölümün amacı tuzaklara değmeden sağ üst veya sağ alt bölgedeki **yeşil çıkış portalına** ulaşmaktır:

| Tuş | İşlev |
| :--- | :--- |
| **`A` / `D`** | Karakteri sola ve sağa hareket ettirir. |
| **`W` veya `Space`** | Zemindeyken yukarı zıplar. |
| **`S`** | Yerdeyken **eğilir** (alçak dikenli tünellerden geçmek için boyu küçülür); havadayken **hızlı iniş** yapar. |
| **`R`** | Mevcut bölümü sıfırlar (10. bölüm tamamlandığında oyunu 1. bölümden yeniden başlatır). |

* **Bölüm 1 — Başlangıç Adımları:** Temel zıplama ve platform geçişleri.
* **Bölüm 2 — Lav Çukurları:** Kırmızı lav zemini üzerindeki adalardan geçiş.
* **Bölüm 3 — Alçak Dikenli Tünel:** `S` tuşuyla eğilerek tavandaki dikenlerin altından geçiş.
* **Bölüm 4 — Hareketli Platformlar:** Yatay ve dikey hareket eden platformlarla zamanlama.
* **Bölüm 5 — Zikzak Kule:** Sağlı sollu basamaklarla dikey tırmanış.
* **Bölüm 6 — Hızlı İniş:** Havadayken `S` tuşuyla tavan dikenlerinden kaçınma.
* **Bölüm 7 — Zamanlama Köprüsü:** Dikenli orta ada ve çift hareketli platform geçişi.
* **Bölüm 8 — Dar Adalar ve Geçit:** Küçük basamaklar ve dar dikenli tünel kombinasyonu.
* **Bölüm 9 — Asansör ve Tuzaklar:** Dikey asansör çıkışı ve dikenli köprü geçişi.
* **Bölüm 10 — Büyük Final:** Hareketli platform, eğilme tüneli ve son asansörün birleştiği final parkuru.
