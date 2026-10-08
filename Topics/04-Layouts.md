# 04 - Yerleşim Yönetimi ve Layoutlar

> [!TIP]
> Bu konuda VS Code ortamının zaten kurulu ve `main.py` dosyası hazır olduğunun varsayılır. Gerekirse önceki konulardaki VS Code kurulum adımlarını gözden geçirebilirsiniz.

## 1. Neden Layout Kullanmalıyız? move() ile Sabit Konumlandırmanın Riskleri

`move(x, y)` fonksiyonu ile widget'ları ekranda kesik koordinatlarda yerleştirmek mümkün olsa da bu yaklaşım ciddi sınırlamalar getirir:

- **Pencere boyutu değiştiğinde düzgün kalmaz:** Kullanıcı pencereyi büyüttüğün ya da küçülttüğünde widget'lar aynı piksel konumunda kalır, sonuç olarak boşluklar oluşur veya widgetlar görünümden dışarı çıkabilir.
- **Farklı ekran çözünürlüklerinde tutarsızlık:** Bir monitörde güzel görünen arayüz, başka bir çözünürlükte bozulabilir.
- **Ülke ve dil farkları:** Metin uzunlukları dilden dile değişir; sabit konumlandırma ile metin kutusu veya etiketinin yeterli genişliği kalmayabilir, metin kesilebilir.
- **Bakım zorluğu:** Her bir widget için ayrı ayrı koordinat hesaplamak zaman alır ve hata prone'dir.

Layout yöneticileri (`QVBoxLayout`, `QHBoxLayout`, `QGridLayout`, `QFormLayout`) bu sorunları otomatik olarak çözer; widgetları dinamik olarak düzenler, pencere boyutu değiştiğinde uyarlamalar yapar ve esnek boşluklar ile hizalama sağlar.

### Layout Türlerinin Karşılaştırması

| Layout Türü | Yönelim | Kullanım Senaryosu | Özel Özellik |
|-------------|---------|-------------------|--------------|
| `QVBoxLayout` | Dikey (üstten alta) | Buton listesi, menü öğeleri | Widgetlar sırayla dikey olarak eklenir |
| `QHBoxLayout` | Yatay (soldan sağa) | Araç çubuğu, onay/iptal butonları | Widgetlar yan yana sıralanır |
| `QGridLayout` | Izgara (satır-sütun) | Formlar, kalkülatör | Widgetlar satır ve sütun koordinatına göre yerleştirilir |
| `QFormLayout` | Form (etiket-giriş çiftleri) | Kullanıcı bilgileri, ayarlar | Sol etiket, sağ giriş alanı otomatik eşleştirir |

> [!NOTE]
> Layout yöneticileri, bir `QWidget`'ın `setLayout()` metodu ile ana pencereye veya başka bir widget'a bağlanır. Layout'a eklenen tüm widgetlar, layout tarafından yönetilen koordinatlarda görüntülenir; bu yüzden `move()` veya `resize()` gibi mutlak konumlandırma metotları artık gerekmez ve hatta kullanılmamalıdır.

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Bir pencereye iki buton ekleyeceğiz; bu butonlar dikey bir düzenle üstte yer alacak. Daha sonra yatay bir buton grubu (Tamam ve İptal) oluşturup bu grubu dikey layout'un içine gömerek iç içe layout örneği göstereceğiz. Son olarak `addStretch()` kullanarak butonları pencerenin üstüne hizalayacağız ve altta esnek boşluk bırakacağız.

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: QWidget üzerindeki merkezi alana QVBoxLayout ekleme ve iki buton ekleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QVBoxLayout

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Yerleşim Yönetimi - QVBoxLayout")
        self.resize(400, 300)

        # Merkez widget (QMainWindow için zorunlu)
        self.merkez_widget = QWidget()
        self.setCentralWidget(self.merkez_widget)

        # Dikey layout oluştur ve merkez widget'a ata
        self.dikey_layout = QVBoxLayout()
        self.merkez_widget.setLayout(self.dikey_layout)

        # İki buton oluştur ve dikey layout'a ekle
        self.buton1 = QPushButton("Buton 1")
        self.buton2 = QPushButton("Buton 2")
        self.dikey_layout.addWidget(self.buton1)
        self.dikey_layout.addWidget(self.buton2)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `import sys` → Sistemle etkileşim için.
- `from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QVBoxLayout` → Kullanacağımız sınıfları içeriye alıyoruz.
- `class AnaPencere(QMainWindow):` → `QMainWindow`'dan miras alır.
- `def __init__(self):` → Yapıcı metod.
- `super().__init__()` → `QMainWindow`'ın yapıcısını çağırır.
- `self.setWindowTitle("Yerleşim Yönetimi - QVBoxLayout")` → Pencere başlığını ayarlar.
- `self.resize(400, 300)` → Pencere boyutunu 400×300 piksel olarak belirler.
- `self.merkez_widget = QWidget()` → `QMainWindow`'ın merkezi widget alanı için zorunlu bir `QWidget` oluşturur.
- `self.setCentralWidget(self.merkez_widget)` → Oluşturulan merkezi widget'ı `QMainWindow`'ın merkezine yerleştirir; bu sayede layout'u bu widget üzerinden yönetebiliriz.
- `self.dikey_layout = QVBoxLayout()` → Yeni bir dikey layout nesnesi oluşturur.
- `self.merkez_widget.setLayout(self.dikey_layout)` → Merkez widget'a bu layout'u atar; artık bu widget üzerindeki tüm yerleşim bu layout tarafından yönetilir.
- `self.buton1 = QPushButton("Buton 1")` ve `self.buton2 = QPushButton("Buton 2")` → İki buton oluşturur.
- `self.dikey_layout.addWidget(self.buton1)` ve `self.dikey_layout.addWidget(self.buton2)` → Butonları dikey layout'a sırayla ekler; layout üstten alta doğru yerleştirir.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Yerleşim Yönetimi - QVBoxLayout" başlıklı, 400 × 300 piksellik bir pencere belirir.
Pencerenin tam ortasında (merkez widget alanı) iki buton dikey olarak birbirinin altında görünür:
  • Üst buton: "Buton 1"
  • Alt buton: "Buton 2"
Pencerenin boyutu değiştiğinde butonlar orantılı olarak yer değiştirir ve hizalı kalır.
```

> [!WARNING]
> `QMainWindow` kullanıyorsanız mutlaka bir merkez widget (`QWidget`) oluşturup `setCentralWidget()` ile ayarlamalısınız; yoksa layout'u doğrudan pencereye uygulayamazsınız.

---

### Adım 2: Yatay buton grubu (QHBoxLayout) oluşturup dikey layout içine gömme (İç İçe Layout)

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QVBoxLayout, QHBoxLayout

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Yerleşim Yönetimi - İç İçe Layout")
        self.resize(400, 300)

        self.merkez_widget = QWidget()
        self.setCentralWidget(self.merkez_widget)

        # Ana dikey layout
        self.ana_dikey_layout = QVBoxLayout()
        self.merkez_widget.setLayout(self.ana_dikey_layout)

        # Üstte iki buton (dikey düzende)
        self.buton_a = QPushButton("Buton A")
        self.buton_b = QPushButton("Buton B")
        self.ana_dikey_layout.addWidget(self.buton_a)
        self.ana_dikey_layout.addWidget(self.buton_b)

        # ---- İç İçe Layout Başlangıç ----
        # Yatay layout için bir widget kapsayıcısı gerekir
        self.yatay_kapsayici = QWidget()
        self.yatay_layout = QHBoxLayout()
        self.yatay_kapsayici.setLayout(self.yatay_layout)

        # Yatay layout'a iki buton ekle (Tamam ve İptal)
        self.tamam_butonu = QPushButton("Tamam")
        self.iptal_butonu = QPushButton("İptal")
        self.yatay_layout.addWidget(self.tamam_butonu)
        self.yatay_layout.addWidget(self.iptal_butonu)

        # Yatay kapsayıcı widget'ını ana dikey layout'a ekle
        self.ana_dikey_layout.addWidget(self.yatay_kapsayici)
        # ---- İç İçe Layout Bitiş ----

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `from PySide6.QtWidgets import ... QHBoxLayout` → Yatay layout için gerekli sınıfı içeriye alıyoruz.
- `self.ana_dikey_layout = QVBoxLayout()` → Ana dikey layout oluşturulur ve merkez widget'a atanır.
- İlk iki buton (`buton_a`, `buton_b`) bu ana dikey layout'a eklenir; yukarıdan aşağıya sıralanır.
- **İç içe layout için bir ara widget gerekir:** `QHBoxLayout`'u doğrudan başka bir layout'a ekleyemeyiz; onu yönetecek bir `QWidget` konteyneri oluştururuz.
  - `self.yatay_kapsayici = QWidget()` → Yatay layout'u taşıyacak ara widget.
  - `self.yatay_layout = QHBoxLayout()` → Yatay layout oluşturur.
  - `self.yatay_kapsayici.setLayout(self.yatay_layout)` → Ara widget'a yatay layout'u atar.
- `self.tamam_butonu = QPushButton("Tamam")` ve `self.iptal_butonu = QPushButton("İptal")` → İki buton oluşturur.
- `self.yatay_layout.addWidget(self.tamam_butonu)` ve `self.yatay_layout.addWidget(self.iptal_butonu)` → Butonları yatay layout'a ekler; yan yana görünür.
- `self.ana_dikey_layout.addWidget(self.yatay_kapsayici)` → Oluşturulan yatay kapsayıcı widget'ını ana dikey layout'a ekler; yani yatay buton grubu, üstteki iki butonun altında gösterilir.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Yerleşim Yönetimi - İç İçe Layout" başlıklı, 400 × 300 piksellik bir pencere belirir.
Pencerenin içinde (merkez widget alanı) aşağıdaki sırayla görünür:
  1. Üstte: "Buton A" (dikey düzen)
  2. İkinci: "Buton B" (dikey düzen)
  3. Altta: Yan yana "Tamam" ve "İptal" butonları (yatay düzen)
Pencere boyutu değiştiğinde tüm öğeler orantılı olarak yer değiştirir, düzen bozulmaz.
```

> [!TIP]
> İç içe layoutlar, karmaşık arayüzler yapmak için esastır. Bir layout'ı doğrudan başka bir layout'a ekleyemeyiz; her zaman bir ara `QWidget` konteyneri kullanmalıyız.

---

### Adım 3: addStretch() ile Esnek Boşluk Ekleyip Butonları Pencerenin Üstüne Hizalama

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QVBoxLayout

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Yerleşim Yönetimi - addStretch")
        self.resize(400, 300)

        self.merkez_widget = QWidget()
        self.setCentralWidget(self.merkez_widget)

        self.dikey_layout = QVBoxLayout()
        self.merkez_widget.setLayout(self.dikey_layout)

        # İlk olarak esnek boşluk ekleyerek sonraki widget'ları yukarı itiyoruz
        self.dikey_layout.addStretch()

        # Şimdi iki buton ekleyelim; bu butonlar pencerenin üst tarafında görünecek
        self.buton_yukari = QPushButton("Yukarı")
        self.buton_asagi = QPushButton("Aşağı")
        self.dikey_layout.addWidget(self.buton_yukari)
        self.dikey_layout.addWidget(self.buton_asagi)

        # Butonların altına da esnek boşluk ekleyebiliriz; bu da butonları ortalar.
        # Sadece üst hizalama isteniyorsa bu satır eklenmez.
        self.dikey_layout.addStretch()

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `self.dikey_layout.addStretch()` → Layout'a esnek boşluk (stretch) ekler. Bu boşluk, mevcut tüm widget'ları dolayan alanda eşit şekilde daşitılır; yani önceki esnek boşluk, sonraki widget'ları layout’un başlangıcından olması gereken yere kadar iterek boş bırakır.
- İlk `addStretch()` çağrısı, sonra eklenen butonların layout’un en üstüne değil, boş bırakılmış bir alan sonrasına yerleşmesini sağlar; sonuç olarak butonlar pencerenin üst kısmına yaklaşır, üstte boşluk kalır.
- İkinci `addStretch()` (isteğe bağlı) butonların altına da esnek boşluk ekleyerek butonları dikeyde ortalar; ancak görevimiz sadece üst hizalama olduğu için bu satırı yorum satırı haline getirebiliriz (kodu gibi bırakarak da aynı etkiyi yapar, çünkü üstteki esnek boşluk zaten butonları yukarı itmiş olur).
- Butonlar (`buton_yukari`, `buton_asagi`) `addWidget` ile eklendikten sonra, yukarıda olan esnek boşluktan dolayı pencereye göre daha yüksek konumda olur.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Yerleşim Yönetimi - addStretch" başlıklı, 400 × 300 piksellik bir pencere belirir.
Pencerenin içinde:
  • Üstte biraz boşluk (esnek boşluk)
  • Sonra "Yukarı" butonu
  • Sonra "Aşağı" butonu
  • (İsteğe bağlı) butonların altında başka esnek boşluk (kodda eklenmediği için görünmez)
Pencereyi dikey olarak büyüttüğün veya küçülttüğünde, butonlar birbirine göre aynı oranda hareket eder ve üstteki boşluk da orantılı olarak değişir; bu yüzden butonlar daima pencerenin üst kısmında görünür kalır.
```

> [!IMPORTANT]
> `addStretch()` verilen değer (varsayılan 0) esnekliğin oranını belirler; farklı sayılar vererek daha hassas dağınıklık sağlanabilir. Örneğin `addStretch(2)` iki kat daha fazla yer kaplar.

---

### Nihai Tam Kod Bloğu

Öğrencinin kopyalayıp doğrudan çalıştırabilecek, tüm adımların birleştiği hali (İç içe layout + üst hizalama için `addStretch` örneği birleştirildi; ancak özen göstermek için ayrı ayrı gösterelim – nihai kod da üst hizalı iç içe layout örneği olacak; ayrı ayrı örnekler üzerinden anlatımdan bahsetmiş olduk. Öğrenci isteğe göre sadece birini seçebilir.)

Aşağıdaki nihai kod, **Adım 1 ve Adım 2** (İç içe layout) ile **Adım 3** (üst hizalama için ek `addStretch`) birleştiren daha kapsayıcı bir örnek sağlar: Üstte esnek boşluk, sonra iki buton (dikey), sonra yatay buton grubu (iç içe), ve en alta da esnek boşluk – bu sayede bütün içeriğe dikeyde ortalı bir görünüm elde ederiz; fakat ödevde sadece üst hizalama istenildiği için sadece ilk esnek boşluk yeterlidir. Biz ödev talimatına uygun olarak, **sadece üst hizalamayı gösteren** nihai kodu verelim (yani iç içe layout + sadece tek `addStretch` başta). Bu şekilde hem ödev kapsamını hem de öğrenmeyi karşılarız.

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QVBoxLayout, QHBoxLayout

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Yerleşim Yönetimi - İç İçe + Üst Hizalama")
        self.resize(400, 300)

        self.merkez_widget = QWidget()
        self.setCentralWidget(self.merkez_widget)

        # Ana dikey layout
        self.ana_dikey_layout = QVBoxLayout()
        self.merkez_widget.setLayout(self.ana_dikey_layout)

        # ---- Üst Hizalama için Esnek Boşluk ----
        self.ana_dikey_layout.addStretch()
        # ----------------------------------------

        # Üstte iki buton (dikey düzende)
        self.buton_bir = QPushButton("Buton Bir")
        self.buton_iki = QPushButton("Buton İki")
        self.ana_dikey_layout.addWidget(self.buton_bir)
        self.ana_dikey_layout.addWidget(self.buton_iki)

        # ---- İç İçe Layout (Yatay Buton Grubu) ----
        self.yatay_kapsayici = QWidget()
        self.yatay_layout = QHBoxLayout()
        self.yatay_kapsayici.setLayout(self.yatay_layout)

        self.tamam_butonu = QPushButton("Tamam")
        self.iptal_butonu = QPushButton("İptal")
        self.yatay_layout.addWidget(self.tamam_butonu)
        self.yatay_layout.addWidget(self.iptal_butonu)

        self.ana_dikey_layout.addWidget(self.yatay_kapsayici)
        # -------------------------------------------

        # (Opsiyonel) Alta esnek boşluk eklemek istersen:
        # self.ana_dikey_layout.addStretch()

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

> [!TIP]
> Sadece `addStretch()` bölümünü yorum satırı yaparak (`# self.ana_dikey_layout.addStretch()`) etkisini kaldırabilir ve butonların pencerenin tam ortasında görünmesini sağlayabilirsiniz.

---

## 3. Bireysel Öğrenme Görevi (Sıra Sende)

> **Görev:** `QFormLayout` kullanarak basit bir iletişim formu oluşturun. Formda iki satır olacak:
> 1. "Ad Soyad:" etiketi ve bir `QLineEdit`
> 2. "E-posta:" etiketi ve bir `QLineEdit`
> 
> Formu bir `QGroupBox` içinde (başlıklı bir çerçeve) gösterip, grup kutusunu pencerenin merkezine yerleştirin. Ayrıca formun altına bir "Gönder" butonu ekleyin ve bu butona tıklandığında girdi temizleyen bir slot fonksiyonu yazın.

- **Ipucu:** 
  - `QFormLayout` oluşturun, `addRow("Ad Soyad:", self.ad_soyad_input)` ve `addRow("E-posta:", self.email_input)` gibi satırları ekleyin.
  - Formu taşımak için bir `QWidget` konteyneri (veya doğrudan `QGroupBox` kullanarak `setLayout`) gerekir.
  - `QGroupBox` ile başlıklı bir çerçeve oluşturup içine formu yerleştirin; grup kutusunu pencerenin merkezine yine bir layout üzerinden yerleştirin (örneğin dikey layout).
  - "Gönder" butonu için `clicked` sinyalini `self.formu_temizle` slotuna bağlayın.
  - Slot fonksiyonunda `self.ad_soyad_input.clear()` ve `self.email_input.clear()` kullanın.

- **Beklenen çıktı:** 
  - Pencerenin içinde başlıklı bir grup kutusu ("Kişi Bilgileri" gibi) görünür.
  - Grup kutusunun içindeki iki satırda etiketler solunda, giriş kutuları sağda hizalı şekilde yer alır.
  - Grup kutusunun altında bir "Gönder" butonu bulunur.
  - Kutulara bilgi girip "Gönder" butonuna bastıktan sonra iki giriş kutusu temizlenir.

Başarıyla tamamladığınızda, farklı layout türlerinin (QVBoxLayout, QHBoxLayout, QFormLayout) nasıl oluşturulduğunu, iç içe kullanımını ve `addStretch()` ile esnek boşluk ekleyerek duyarlı (responsive) arayüzler tasarlayabileceğinizi görmüş olursunuz. Bir sonraki konuda bu bilgileriyle veri görüntüleme widget'ları (`QListWidget`, `QTableWidget`) ve onlarla etkileşim kurmayı öğreneceğiz.
