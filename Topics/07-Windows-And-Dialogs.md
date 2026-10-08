# 07 - Pencere Anatomisi ve Diyaloglar

## 1. Pencere Anatomisi ve Diyaloglar — Kavramsal Temeller

Modern masaüstü uygulamalarında pencere yapısı genellikle standart bileşenlerden oluşur: menü çubuğu, araç çubuğu, durum çubuğu ve içerik alanı. Ayrıca kullanıcıyla etkileşim kurmak için **diyalog pencereleri** (modal veya modeless) kullanılır. PySide6 bu bileşenleri hazır sınıflar olarak sunar; sayesinde geliştirici sıfırdan şeyler inşa etmek yerine bu yapıları hızlıca entegre edebilir.

### Ana Pencere Anatomisi: QMenuBar, QToolBar, QStatusBar

Bir `QMainWindow` içinde aşağıdaki yapı taşları bulunur:

| Bileşen | Açıklama |
|---------|----------|
| `QMenuBar` | Üstteki menü çubuğu; `QMenu` ve `QAction` nesneleriyle oluşturulur. |
| `QToolBar` | Menünün hemen altında veya yanında yer alan araç çubuğu; sık kullanılan eylemler için simge ve metin içerebilir. |
| `QStatusBar` | Pencerenin alt kısımlarında kısa mesajlar gösterir; genellikle geçici bilgi veya durum güncellemeleri için kullanılır. |
| Merkez Widget | `setCentralWidget()` ile yerleştirilen ana içerik alanı; burada başka widget'lar (butonlar, metin kutuları vb.) yer alır. |

> [!NOTE]
> `QMainWindow` kullanıyorsanız mutlaka bir merkez widget (`QWidget`) oluşturup `setCentralWidget()` ile ayarlamalısınız; yoksa bu yapı taşlarını doğrudan pencereye ekleyemezsiniz.

### Diyalog Penceresi Mantığı (QDialog) ve Modal/Modeless Farkı

`QDialog` sınıfı, kullanıcıdan giriş almak, ayar değiştirmek veya bilgi vermek için geçici pencereler oluşturmak için kullanılır. İki temel tip vardır:

- **Modal Diyalog:** Açıldığında diğer tüm pencereler etkisiz hale gelir; kullanıcı diyaloğu kapatmadan önce ana uygulamaya dönemez. Örnek: "Dosya Kaydet" penceresi.
- **Modeless Diyalog:** Modal olmayan diyaloglar; kullanıcı diyalog açıkken diğer pencerelerle de etkileşime geçebilir. Örnek: "Bul ve Değiştir" penceresi.

> [!TIP]
> Çoğu durumda `QDialog` ile modal diyalog yeterlidir; özellikle kullanıcıdan onay veya giriş gerektiğinde tercih edilir.

### Hazır Bilgi/Uyarı Kutuları: QMessageBox (Information, Warning, Critical)

Kullanıcıya hızlı bilgi, uyarı veya hata mesajı göstermek için `QMessageBox` sınıfı kullanılır. Bu sınıf, önceden tanımlanmış simgeler ve butonlarla birlikte çeşitli icon ve mesaj tipleri sunar.

| Tip | Simge | Kullanım Durumu |
|-----|-------|-----------------|
| `QMessageBox.Information` | ℹ️ | Bilgilendirici mesajlar (işlem tamamlandı, ayar kaydedildi). |
| `QMessageBox.Warning` | ⚠️ | Potansiyel sorunları belirten uyarılar (örnek: eksik alan). |
| `QMessageBox.Critical` | ❌ | Hataları gösterir (örnek: dosya okunamadı). |
| `QMessageBox.Question` | ❓ | Kullanıcıdan evet/hayır sorusu sorar. |

> [!IMPORTANT]
> `QMessageBox` çağrısı genellikle statik metodlarla (örneğin `QMessageBox.warning(parent, title, message)`) yapılır; bu da kendi diyalog sınıfı oluşturmak zorunda kalmadan hızlı mesaj gösterimini sağlar.

### Dosya Seçim Diyalogları: QFileDialog (Dosya Açma / Kaydetme)

Kullanıcıdan dosya seçmesini veya kaydetmesini istediğimizde `QFileDialog` sınıfı kullanılır. Bu sınıf platformun yerel dosya seçiciyi çağırarak tutarlı bir deneyim sunar.

- `QFileDialog.getOpenFileName()`: Tek dosya seçimi için kullanılır; seçilen dosya yolunu ve seçtiği filtreyi döndürür.
- `QFileDialog.getSaveFileName()`: Dosya kaydetmek için kullanılır; kullanıcı bir konum ve dosya adı belirler.
- Her iki metod da `(seçilen_yol, seçilen_filtre)` tuple'ı döndürür; kullanıcı iptal ettiyse yol boş string olur.

> [!WARNING]
> Dosya seçimi sonrası dönen yolun boş olup olmadığını kontrol etmeyi unutmayın; boş string kullanıcı tarafından iptal edildiğini gösterir ve bu durumda işleme devam etmemeliyiz.

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Bir ana pencere oluşturacağız; üstte "Dosya" menüsü ve içinde "Aç" aksiyonu bulunacak. Kullanıcı "Aç" seçtiğinde bir dosya seçim diyalogu açılacak; dosya seçilmezse uyarı penceresi gösterilecek, seçilirse dosya yolu durum çubuğunda görüntülenecek. Ayrıca menüye "Hakkında" seçeneği ekleyip tıklandığında `QMessageBox.about` penceresi açılacak (bu görev sırasımdaki ödev).

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: QMenuBar üzerinde "Dosya" menüsü ve "Aç" aksiyonu (QAction) oluşturma

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QLabel, QVBoxLayout, QMenuBar, QStatusBar

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Pencere Anatomisi ve Diyaloglar")
        self.resize(600, 400)

        # Merkez widget ve basit bir etiket (örnek içerik)
        merkez = QWidget()
        self.setCentralWidget(merkez)
        duzen = QVBoxLayout(merkez)
        self.etiket = QLabel("Bir dosya açmak için menüyü kullanın.")
        duzen.addWidget(self.etiket)

        # Menü çubuğu oluşturma
        menubar = self.menuBar()  # QMainWindow'ın menü çubuğunu alır
        dosya_menu = menubar.addMenu("Dosya")  # "Dosya" menüsü ekler

        # "Aç" aksiyonu (QAction)
        self.ac_aksiyonu = dosya_menu.addAction("Aç")
        self.ac_aksiyonu.triggered.connect(self.dosya_ac)

        # "Hakkında" aksiyonu (QAction)
        self.hakkinda_aksiyonu = dosya_menu.addAction("Hakkında")
        self.hakkinda_aksiyonu.triggered.connect(self.hakkinda_goster)

        # Durum çubuğu (QStatusBar)
        self.durum_cubugu = QStatusBar()
        self.setStatusBar(self.durum_cubugu)

    def dosya_ac(self):
        # Bu fonksiyon sonraki adımda doldurulacak
        pass

    def hakkinda_goster(self):
        # Bu fonksiyon sonraki adımda doldurulacak
        pass

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- **Menü çubuğu ve menü:** `self.menuBar()` çağrısıyla `QMainWindow`'ın menü çubuğu elde edilir; `addMenu("Dosya")` ile yeni bir menü eklenir.
- **QAction:** `dosya_menu.addAction("Aç")` ile bir aksiyon oluşturulur; bu aksiyonun `triggered` sinyali `self.dosya_ac` slotuna bağlanır. Benzer şekilde `dosya_menu.addAction("Hakkında")` ile "Hakkında" aksiyonu oluşturulur ve `triggered` sinyali `self.hakkinda_goster` slotuna bağlanır.
- **Durum çubuğu:** `QStatusBar()` örneği oluşturulur ve `setStatusBar()` ile pencereye eklenir; burada geçici mesajlar gösterilecektir.
- **Merkez widget:** İçerik alanı olarak basit bir `QLabel` kullanılır; ilerleyen adımlarda güncellenecek.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Pencere Anatomisi ve Diyaloglar" başlıklı bir pencere görünür.
Pencerenin üst kısmında "Dosya" menüsü bulunur; menüye tıklandığında "Aç" ve "Hakkında" seçenekleri görünür.
Pencerenin alt kısmında boş bir durum çubuğu (StatusBar) yer alır.
Pencerenin ortasında "Bir dosya açmak için menüyü kullanın." yazan bir etiket bulunur.
Henüz bir işlev bağlı olmadığı için "Aç" seçeneğine tıklandığında bir şey olmaz, "Hakkında" seçeneğine tıklandığında da bir şey olmaz.
```

---

### Adım 2: "Aç" aksiyonuna tıklandığında QFileDialog.getOpenFileName çalıştırarak bilgisayardan metin dosyası seçtirme ve dosya yolunu durum çubuğunda gösterme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QLabel, QVBoxLayout, QMenuBar, QStatusBar, QFileDialog, QMessageBox

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Pencere Anatomisi ve Diyaloglar")
        self.resize(600, 400)

        # Merkez widget ve etiket
        merkez = QWidget()
        self.setCentralWidget(merkez)
        duzen = QVBoxLayout(merkez)
        self.etiket = QLabel("Bir dosya açmak için menüyü kullanın.")
        duzen.addWidget(self.etiket)

        # Menü çubuğu
        menubar = self.menuBar()
        dosya_menu = menubar.addMenu("Dosya")
        self.ac_aksiyonu = dosya_menu.addAction("Aç")
        self.ac_aksiyonu.triggered.connect(self.dosya_ac)
        self.hakkinda_aksiyonu = dosya_menu.addAction("Hakkında")
        self.hakkinda_aksiyonu.triggered.connect(self.hakkinda_goster)

        # Durum çubuğu
        self.durum_cubugu = QStatusBar()
        self.setStatusBar(self.durum_cubugu)

    def dosya_ac(self):
        # Dosya seçimi diyalogunu aç
        dosya_adi, _ = QFileDialog.getOpenFileName(
            self,
            "Metin Dosyası Aç",
            "",
            "Metin Dosyaları (*.txt);;Tüm Dosyalar (*)"
        )

        # Kullanıcı bir dosya seçmediyse (iptal ettiyse)
        if not dosya_adi:
            QMessageBox.warning(
                self,
                "Dosya Seçilmedi",
                "Lütfen bir dosya seçin veya işlemi iptal edin."
            )
            return

        # Dosya seçildiyse etiketi güncelle ve durum çubuğunda göster
        self.etiket.setText(f"Seçilen dosya: {dosya_adi}")
        self.durum_cubugu.showMessage(f"Dosya açıldı: {dosya_adi}", 5000)  # 5 saniye göster

    def hakkinda_goster(self):
        QMessageBox.about(
            self,
            "Hakkında",
            "Bu uygulama PySide6 ile geliştirilmiş bir dosya seçici örneğidir."
        )

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- **QFileDialog.getOpenFileName():** Kullanıcıya bir dosya seçme penceresi gösterir; ilk parametre pencereyi belirtilen ebeveyn (`self`) ile ilişkilendirir, ikinci parametre diyalog başlığı, üçüncü parametre başlangıç dizini (boş String varsayılan dizin), dördüncü parametre dosya filtreleri.
- **Dönüş değeri:** Tuple olarak `(seçilen_dosya_yolu, seçilen_filtre)` döndürür; iptal edilirse `seçilen_dosya_yolu` boş string olur.
- **QMessageBox.warning():** Kullanıcı dosya seçmezse uyarı penceresi gösterilir; `showMessage()` ile durum çubuğunda geçici mesaj (5 saniye) görüntülenir.
- **Etiket güncellemesi:** Seçilen dosya yolu arayüzdeki `QLabel` ile gösterilir.
- **QMessageBox.about():** "Hakkında" seçeneğine tıklandığında uygulama hakkında kısa bilgi içeren bir bilgi kutusu gösterir; bu pencerede sadece bir "Tamam" düğmesi bulunur.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Pencere Anatomisi ve Diyaloglar" başlıklı bir pencere görünür.
Kullanıcı menüden "Dosya → Aç" seçeneğine tıklar.
Bir dosya seçim penceresi açılır; kullanıcı bir metin dosyası (*.txt) veya herhangi bir dosya seçebilir.
- Eğer kullanıcı "İptal" düğmesine basarsa ya da dosya seçmeden pencereyi kapatırsa:
    - "Dosya Seçilmedi" başlıklı bir uyarı penceresi (QMessageBox.warning) görünür.
    - Durum çubuğu ve etiket değişmez.
- Eğer kullanıcı bir dosya seçer ve "Aç" düğmesine basarsa:
    - Pencerenin ortasındaki etiket "Seçilen dosya: /kullanıcı/ev/belgeler/not.txt" şeklinde güncellenir.
    - Durum çubuğunda 5 saniye boyunca "Dosya açıldı: /kullanıcı/ev/belgeler/not.txt" mesajı görünür.
Kullanıcı menüden "Dosya → Hakkında" seçeneğine tıklar:
    - "Hakkında" başlıklı bir bilgi kutusu (QMessageBox.about) görünür, içinde uygulama hakkında kısa açıklama ve bir "Tamam" düğmesi bulunur.
```

---

### Adım 3: Menüye "Hakkında" seçeneği ekleyip tıklandığında QMessageBox.about penceresi açma

Bu adım zaten **Adım 1** ve **Adım 2** içinde uygulanmıştır: "Hakkında" aksiyonu oluşturulmuş, `triggered` sinyali `self.hakkinda_goster` slotuna bağlanmış ve bu slot içinde `QMessageBox.about` çağrısı yapılmıştır. Böylece kullanıcı menüden "Hakkında" seçeneğini seçtiğinde uygulama hakkında bilgi penceresi görüntülenir.

> [!NOTE]
> `showMessage()` metodunun ikinci parametresi mesajın görüntüleme süresini milisaniye cinsinden belirtir; örnekte `5000` (5 saniye) kullanıldı. Süre vermesizse mesaj başka bir mesaj gelene kadar gösterilir.

## 3. Sıra Sende (Pekiştirme Görevi)

- **Görev:** Menüye "Hakkında" seçeneği ekleyip tıklandığında `QMessageBox.about` penceresi açma.
- **İpucu:** Menüye yeni bir `QAction` ekleyin (örneğin `self.hakkinda_aksiyonu = dosya_menu.addAction("Hakkında")`), bu aksiyonun `triggered` sinyalini `self.hakkinda_goster` adlı bir slot fonksiyonuna bağlayın. Slot içinde `QMessageBox.about(self, "Hakkında", "Bu uygulama PySide6 ile geliştirilmiş bir dosya seçici örneğidir.")` komutunu çalıştırın.
- **Beklenen Çıktı:** Kodu çalıştırdığınızda, menüde "Dosya" altında "Aç" ve "Hakkında" seçenekleri görürsünüz. "Hakkında" seçeneğine tıklandığında uygulama hakkında kısa bilgi içeren bir bilgi kutusu (QMessageBox.about) aparecer; bu pencerede sadece bir "Tamam" düğmesi bulunur.

> [!TIP]
> `QMessageBox.about()` metodu, uygulama hakkında standart bilgi kutusu sunar; özel bir ikon ve başlık ile birlikte uygulama adı ve kısa açıklama gösterilir.