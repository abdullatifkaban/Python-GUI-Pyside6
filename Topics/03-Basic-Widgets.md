# 03 - Temel Widgetlar ve Etkileşim

## 1. Olay Yönelimli Programlama ve Temel Widgetlar

### Sinyal ve Slot Kavramı

Kullanıcı bir butona tıkladığında, bir kutucuğu işaretlediğinde veya bir metin kutusuna yazı yazdığında bu eylemler **olay** olarak adlandırılır. PySide6'de her widget belirli olayları (sinyaller) üretir ve biz bu sinyalleri istediğimiz fonksiyonlara (slot) bağlayarak uygulama davranışını belirleriz.

> [!TIP]
> Bir sinyal, "bir şey oldu" mesajıdır; bir slot ise bu mesaja nasıl yanıt verileceğini tanımlayan fonksiyondur.

> [!NOTE]
> Sinyal-slot mekanizması, kullanıcı arayüzünün dinamik ve etkileşimli olmasını sağlar; kod içinde sürekli durum kontrolü yapmaya gerek kalmaz.

### Temel Widgetlar ve Kullanımları

PySide6, kullanıcıyla etkileşim kurmamızı sağlayan çeşitli **widget** (görsel bileşen) sınıfları sunar. En temel bazıları şunlardır:

| Widget | Açıklama | Ortak Sinyaller | En Kullanılan Metotlar |
|--------|----------|------------------|------------------------|
| `QLabel` | Düzenlenemeyen metin veya resim gösterir. | - | `setText()`, `setPixmap()` |
| `QPushButton` | Basılıp bırakılabilecek bir düğme. | `clicked`, `pressed`, `released` | `setText()`, `setEnabled()` |
| `QLineEdit` | Tek satırlı metin girişi alanı. | `textChanged`, `returnPressed`, `editingFinished` | `text()`, `setText()`, `clear()` |
| `QCheckBox` | İki durumlu ( işaretli/işaretsiz ) kutucuk. | `stateChanged`, `toggled` | `isChecked()`, `setChecked()`, `setText()` |
| `QRadioButton` | Bir grup içinde sadece birinin seçilebileceği seçenek düğmesi. | `toggled` | `isChecked()`, `setChecked()` |

> [!IMPORTANT]
> Yukarıdaki tabloyu gördüğünüzde, widgetların **sinyallerini** kullanarak kullanıcı etkileşimini yakalayabileceğimizi ve **slot** fonksiyonlarıyla bu sinyallere yanıt verebileceğimizi unutmayın.

---

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Bir pencereye bir `QPushButton` ve bir `QLabel` ekleyeceğiz. Butona tıklandığında label metnini dinamik olarak değiştireceğiz. Daha sonra ikinci bir buton ekleyip label metnini sıfırlayan bir "Temizle" slotu yazacağız.

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: Pencereye QPushButton ve QLabel Ekleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Temel Widgetlar")
        self.resize(400, 300)

        # QLabel: başlangıç metni
        self.etiket = QLabel("Henüz tıklanmadı!", self)
        self.etiket.move(150, 100)

        # QPushButton: etiketi değiştirecek düğme
        self.buton = QPushButton("Tıkla", self)
        self.buton.move(150, 150)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `import sys` → Sistemle etkileşim (örneğin çıkış kodu) için.
- `from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel` → Kullanacağımız sınıfları içeriye alıyoruz.
- `class AnaPencere(QMainWindow):` → `QMainWindow`'dan miras alır; menü çubuğu, durum çubuğu gibi yapıları otomatik olarak kullanır.
- `def __init__(self):` → Yapıcı metod, nesne oluşturulduğunda ilk çalıştırılır.
- `super().__init__()` → `QMainWindow`'ın yapıcısını çağırır; temel pencere altyapısı kurulur.
- `self.setWindowTitle("Temel Widgetlar")` → Pencere başlığını ayarlar.
- `self.resize(400, 300)` → Pencere boyutunu 400×300 piksel olarak belirler.
- `self.etiket = QLabel("Henüz tıklanmadı!", self)` → 
  - İlk argüman: Label'ın göstereceği metin.
  - İkinci argüman (`self`): Label'ın ana pencereye bağlı olmasını sağlar; bu sayede label pencereyle birlikte gösterilir ve gizlenir.
- `self.etiket.move(150, 100)` → Label'ı pencerenin sol üst köşesinden (0,0) 150px sağ, 100px aşağı konumlandırır.
- `self.buton = QPushButton("Tıkla", self)` → 
  - İlk argüman: Düğmenin üstüne yazacak metin.
  - İkinci argüman (`self`): Düğmeyi ana pencereye bağlar.
- `self.buton.move(150, 150)` → Düğmeyi label'ın hemen altında (y ekseninde 50px daha aşağı) konumlandırır.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Temel Widgetlar" başlıklı, 400 × 300 piksellik bir pencere belirir.
Pencerenin içinde:
  • Üstte: "Henüz tıklanmadı!" yazısı (QLabel)
  • Labelın hemen altında: "Tıkla" yazılı bir buton (QPushButton)
Henüz butona tıklanmadığı için label metni değişmedi.
```

> [!WARNING]
> Widget oluştururken ikinci argüman olarak `self` (ana pencere) vermezseniz, widget **bağımsız bir pencere** olarak açılır ve ana pencerenin parçası olmaz.

---

### Adım 2: Sınıf İçine Özel Slot Fonksiyonu Ekleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Temel Widgetlar")
        self.resize(400, 300)

        self.etiket = QLabel("Henüz tıklanmadı!", self)
        self.etiket.move(150, 100)

        self.buton = QPushButton("Tıkla", self)
        self.buton.move(150, 150)

        # Butonun clicked sinyalini slot fonksiyonuna bağlama
        self.buton.clicked.connect(self.buton_tiklandi)

    def buton_tiklandi(self):
        """Buton tıklandığında çalışacak slot fonksiyonu."""
        self.etiket.setText("Buton tıklandı!")

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `self.buton.clicked.connect(self.buton_tiklandi)` → 
  - `self.buton.clicked`: QPushButton'ın tıklandığında yayınladığı sinyal.
  - `.connect()`: Bu sinyali belirtilen fonksiyona (slot) bağlar.
  - `self.buton_tiklandi`: Tıklandığında çalışacak metod (slot).
- `def buton_tiklandi(self):` → Slot fonksiyonu; buton tıklandığında çalışır.
  - `self.etiket.setText("Buton tıklandı!")` → Label'ın metnini günceller.

#### Görsel / İşlevsel Sonuç

```
Program çalıştırıldığında:
  • Başlangıçta label: "Henüz tıklanmadı!"
  • Butona tıklandığında label anında "Buton tıklandı!" olarak değişir.
  • Her tıklandığında aynı metin görünür (değişmez, çünkü her zaman aynı yazıyı set ediyoruz).
```

> [!TIP]
> Slot fonksiyonu içinde `self.` ön ekiyle sınıf değişkenlerine (örneğin `self.etiket`) erişebiliriz; bu sayetiket gibi widget'ları başka metodlardan da yönetebiliriz.

---

### Adım 3: İkinci Buton ve "Temizle" Slotu Ekleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Temel Widgetlar")
        self.resize(400, 300)

        self.etiket = QLabel("Henüz tıklanmadı!", self)
        self.etiket.move(150, 100)

        self.buton = QPushButton("Tıkla", self)
        self.buton.move(150, 150)

        self.temizle_butonu = QPushButton("Temizle", self)
        self.temizle_butonu.move(150, 200)

        # Sinyal-slot bağlantıları
        self.buton.clicked.connect(self.buton_tiklandi)
        self.temizle_butonu.clicked.connect(self.temizle)

    def buton_tiklandi(self):
        self.etiket.setText("Buton tıklandı!")

    def temizle(self):
        self.etiket.setText("Henüz tıklanmadı!")
        self.etiket.adjustSize()  # Metin değiştiğinde label boyutunu yeniden hesaplar

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `self.temizle_butonu = QPushButton("Temizle", self)` → Yeni bir buton oluşturuyoruz.
- `self.temizle_butonu.move(150, 200)` → İlk butonun altında (y=200) konumlandırıyoruz.
- `self.temizle_butonu.clicked.connect(self.temizle)` → Temizle butonuna tıklandığında `temizle` slot fonksiyonunu çalıştıracak şekilde bağlıyoruz.
- `def temizle(self):` → 
  - Label metnini başlangıç durumuna (`Henüz tıklanmadı!`) getirir.
  - `self.etiket.adjustSize()` → Metin değiştiğindeki label boyutunu otomatik olarak yeniden hesaplar; böylece metin kısa olduğunda label gereksiz büyük kalmaz.

#### Görsel / İşlevsel Sonuç

```
Ekranda iki buton görürüz:
  • Üst buton: "Tıkla"
  • Alt buton: "Temizle"
  • Aralarında: QLabel

Interaksiyon:
  • "Tıkla" butonuna basıldığında label: "Buton tıklandı!"
  • "Temizle" butonuna basıldığında label: "Henüz tıklanmadı!" (başlangıç metni) olur.
```

> [!IMPORTANT]
> `adjustSize()` metodu, label gibi widget'ların metin değiştiğinde kendi boyutunu yeniden hesaplamasını sağlar; böylece metin kısa olduğunda label gerekenden büyük kalmaz ve kullanıcı arayüzü daha temiz görünür.

---

### Nihai Tam Kod Bloğu

Öğrencinin kopyalayıp doğrudan çalıştırabilecek, tüm adımların birleştiği hali:

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Temel Widgetlar")
        self.resize(400, 300)

        self.etiket = QLabel("Henüz tıklanmadı!", self)
        self.etiket.move(150, 100)

        self.buton = QPushButton("Tıkla", self)
        self.buton.move(150, 150)

        self.temizle_butonu = QPushButton("Temizle", self)
        self.temizle_butonu.move(150, 200)

        # Sinyal-slot bağlantıları
        self.buton.clicked.connect(self.buton_tiklandi)
        self.temizle_butonu.clicked.connect(self.temizle)

    def buton_tiklandi(self):
        self.etiket.setText("Buton tıklandı!")

    def temizle(self):
        self.etiket.setText("Henüz tıklanmadı!")
        self.etiket.adjustSize()

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

> [!TIP]
> Label metnini daha uzun bir cümle yaparak (örneğin `"Buton başarıyla tıklandı ve sayaç 1 arttı."`) deneyebilirsiniz; ardından `adjustSize()` ile label'ın metne göre doğru boyutlandığını gözlemleyebilirsiniz.

---

## 3. Bireysel Öğrenme Görevi (Sıra Sende)

> **Görev:** Yukarıdaki uygulamaya bir `QLineEdit` ekleyin. Kullanıcı bu metin kutusuna bir yazı yazıp Enter tuşuna bastığında, label metnini yazının içeriğiyle güncelleyen bir slot fonksiyonu yazın.

- **Ipucu:** 
  - `QLineEdit` sınıfını import edin (`from PySide6.QtWidgets import QLineEdit`).
  - `self.line_edit = QLineEdit(self)` şeklinde oluşturun ve uygun bir konuma (`move()`) yerleştirin.
  - `returnPressed` sinyalini kullanın; bu sinyal kullanıcı Enter tuşuna bastığında yayınlanır.
  - Slot fonksiyonunuz içinde `self.etiket.setText(self.line_edit.text())` yazın.

- **Beklenen çıktı:** 
  - Program çalıştığında pencerenin alt kısmında bir metin kutusu görünür.
  - Kutuda bir şey yazıp Enter'a basın; label anında yazdığınız metni gösterir.
  - Kutuyu temizlemek için aynı zamanda bir "Metin Kutusu Temizle" butonu ekleyebilir ve `self.line_edit.clear()` kullanabilirsiniz.

Başarıyla tamamladığınızda, temel widgetların (QLabel, QPushButton, QLineEdit) nasıl oluşturulduğunu, sinyal-slot mekanizmasıyla nasıl etkileşim kurduğunu ve `self.` ön ekiyle sınıf değişkenleri olarak nasıl yönetildiğini görmüş olursunuz. Bir sonraki konuda bu widget'ları düzenli bir şekilde yerleştirmek için **layout yöneticileri** (QVBoxLayout, QHBoxLayout vb.) öğreneceğiz.
