# 02 - OOP ve Yapı

## 1. Nesne Yönelimli Programlama ile Profesyonel Arayüzler

PySide6 ile basit bir `QWidget` penceresi oluşturmak işe yarar, ancak gerçek dünya uygulamaları için daha düzenli ve ölçeklenebilir bir yapıya ihtiyaç duyarız. İşte bu noktada **Nesne Yönelimli Programlama (OOP)** devreye giriyor.

### Neden Arayüz Kodlarında OOP Kullanılır?

Bir pencereyi sadece işlevsel fonksiyonlarla yönetmek, uygulama büyüdükça kodun karışık ve sürdürülemez olmasına yol açar. OOP sayesinde:

- **Encapsulation (Kapsülleme):** Pencereye ait tüm özellikler (başlık, boyut, bileşenler) ve davranışları (buton tıklama, veri güncelleme) tek bir sınıf içinde gruplanır.
- **Inheritance (Miras Alma):** `QMainWindow` gibi güçlü Qt sınıflarından özellikleri miras alarak sıfırdan her şeyi yapıyoruz yerine, hazır altyapıyı genişletiyoruz.
- **Modülerlik:** Uygulamanın farklı parçalarını (menü çubuğu, durum çubuğu, ana içerik) ayrı metodlarda yöneterek kodun okunabilirliğini artırıyoruz.

> [!TIP]
> Bir sınıfı düşünün; kendi içinde hem veri (pencere boyutu, başlık) hem de işlev (başlangıç ayarları, etiket oluşturma) bulunduran bir şablon olarak. Bu şablondan istediğimiz kadar örnek (instance) oluşturabiliriz.

### QMainWindow vs QWidget: Ne Zaman Hangisini Kullanmalıyız?

`QWidget` en temel görsel pencere türüdür; `QMainWindow` ise daha fazla yapıya sahip bir penceredir — menü çubuğu, araç çubuğu, durum çubuğu ve中央 widget alanı gibi yerleşik bölümleri bulunur.

| Özellik | `QWidget` | `QMainWindow` |
|---------|-----------|---------------|
| Menü çubuğu (`QMenuBar`) | Yok (manuel ekleme gerekir) | Yerleşik |
| Araç çubuğu (`QToolBar`) | Yok | Yerleşik |
| Durum çubuğu (`QStatusBar`) | Yok | Yerleşik |
| Merkez widget alanı | Yok (tüm pencereyi yönetir) | `setCentralWidget()` ile özel widget yerleştirilebilir |
| Kullanım Senaryosu | Basit diyaloglar, araçlar | Ana uygulama pencereleri |

> [!NOTE]
> Eğer uygulamanız sadece bir buton ve bir etiketten oluşuyorsa `QWidget` yeterli olabilir; ancak menü, araç çubuğu veya durum çubuğu gibi standart bileşenler planlıyorsanız `QMainWindow` doğru seçimdir.

### Miras Alma ve `super().__init__()`

PySide6'de bir pencere sınıfı oluştururken genellikle mevcut Qt sınıflarından **miras** alırız. Bu sayede tüm hazır işlevleri (olay yönetimi, çizim motoru, pencereler arası iletişim) kazanırız ve sadece ihtiyacımız olan ekstra özellikleri ekleriz.

```python
class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()  # QMainWindow'ın kurucusunu çalıştırır
        # Şimdi kendi ekstra ayarlarımızı yapabiliriz
```

`super().__init__()` ifadesi, miras alınan sınıfın (`QMainWindow`) `__init__` metodunu çağırır; böylece pencere sisteminin iç yapısı doğru şekilde kurulur. Bu çağrıyı yapmazsak, `QMainWindow`'ın temel özellikleri (menü çubuğu gibi) eksik kalır ve pencere beklendiği gibi çalışmayabilir.

### Sınıf İçi Değişkenler ve Metot Düzeni

Bir pencere sınıfı içinde iki tür öğe tutarız:

1. **Özellikler (Attributes):** `self.label`, `self.button` gibi, sınıfın yaşamı boyunca değişebilen değerler.
2. **Metotlar (Methods):** `self.initUI()`, `self.buton_tiklandi()` gibi, belirli görevleri yapan işlevler.

Bu düzenlemeyi şu şekilde yaparız:

```python
class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.pencere_basligi = "OOP ile PySide6"  # Özellik örneği
        self.initUI()  # Metod çağrısı

    def initUI(self):
        # Arayüzü kurmak için ayrı bir metod
        self.setWindowTitle(self.pencere_basligi)
        self.resize(400, 300)
```

> [!IMPORTANT]
> `self.` ön eki, sınıfın bir özelliğini veya metotunu gösterir; bu sayede `__init__` içinde tanımlanan bir değişkene `initUI` gibi başka bir metottan da erişebiliriz.

---

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Bir önceki konuda oluşturduğumuz boş pencereyi OOP yapısıyla yeniden yapılandıracağız. Şimdi `QMainWindow` dan miras alacak bir `AnaPencere` sınıfı tanımlayıp, içinde bir `QLabel` göstereceğiz. Ayrıca pencereyi başlatan ve gösteren `main.py` dosyasını VS Code içinde düzenleyeceğiz.

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: QMainWindow'tan Miras Alan Sınıfı ve `__init__` Metodu

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Ana Pencere")
        self.resize(400, 300)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- `import sys` → Komut satırı argümanlarını ve çıkış kodunu yönetmek için gerekli.
- `from PySide6.QtWidgets import QApplication, QMainWindow` → Uygulama döngüsü için `QApplication`, ana pencere sınıfı için `QMainWindow` alır.
- `class AnaPencere(QMainWindow):` → `QMainWindow`'dan miras alır; bu sayede menü çubuğu, araç çubuğu gibi yapıları otomatik olarak kullanır.
- `def __init__(self):` → Sınıfın yapıcı (constructor) metodu; nesne oluşturulduğunda ilk çalıştırılan bloktur.
- `super().__init__()` → `QMainWindow`'ın kendi yapıcısını çalıştırır; pencerenin temel altyapısı kurulur.
- `self.setWindowTitle("Ana Pencere")` → Pencerenin başlığını ayarlar; `self.` ile sınıf özelliği gibi kullanılır (aslında doğrudan `QMainWindow` metodu).
- `self.resize(400, 300)` → Pencere boyutunu 400×300 piksel olarak belirler.
- `app = QApplication(sys.argv)` → Qt olay döngüsünü yöneten uygulama nesnesi oluşturur.
- `pencere = AnaPencere()` → `AnaPencere` sınıfından bir örnek (instance) oluşturur; bu aşamada `__init__` çalışır ve pencere yapılandırılır.
- `pencere.show()` → Pencereyi ekranda görünür kılar.
- `sys.exit(app.exec())` → Olay döngüsünü başlatır ve pencere kapatıldığında güvenli çıkış sağlar.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Ana Pencere" başlıklı, 400 × 300 piksellik boş bir pencere belirir.
Henüz hiçbir etiket, buton veya diğer bileşen eklemediğimiz için pencerenin içi tamamen boştur.
```

> [!WARNING]
> `super().__init__()` çağrısını unutursanız, `QMainWindow`'ın menü çubuğu gibi temel özellikleri eksik olur ve pencere beklendiği gibi görüntülenmeyebilir.

---

### Adım 2: Arayüzü Ayıran `initUI()` Metodu ve Pencere Özellikleri

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()  # Arayüzü ayrı bir metotta topluyoruz

    def initUI(self):
        self.setWindowTitle("OOP ile PySide6")
        self.resize(400, 300)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `__init__` içinde doğrudan pencere ayarlarını yazmak yerine, `self.initUI()` çağrısıyla bu işi ayrı bir metota devrettik.
- Bu sayede `__init__` daha temiz ve okunabilir olur; özellikle arayüz kompleksleştiğinde (menüler, araç çubuğu, çok sayıda widget) `initUI` metoduだけで tüm arayüz kurulumunu yönetebiliriz.
- `initUI` metodu içinde yine `self.setWindowTitle` ve `self.resize` kullanıyoruz; bu metotlar `QMainWindow`'dan miras alınmıştır ve `self.` ile sınıf içinde erişilebilir.

#### Görsel / İşlevsel Sonuç

```
Önceki adımla aynı: "OOP ile PySide6" başlıklı 400 × 300 piksellik boş bir pencere.
Fark, kod yapısında: pencere ayarları artık `initUI` metodu içinde toplu olarak yönetiliyor.
```

> [!TIP]
> `initUI` gibi ayrı bir metod kullanmak, özellikle aynı pencereyi farklı yapılandırmalarla (örneğin test modu ve üretim modu) tekrar tekrar oluşturmanız gerektiğinde kod tekrarını önler.

---

### Adım 3: QLabel Bileşeni Oluşturup Pencereye Ekleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QLabel

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("OOP ile PySide6")
        self.resize(400, 300)

        # QLabel oluşturup metin atıyoruz
        self.etiket = QLabel("Merhaba, PySide6!", self)
        self.etiket.move(150, 130)  # Pencere içinde konumlandırma (x=150, y=130)

        # Pencereyi göster
        self.show()

        # Ekran ortalaması:
        from PySide6.QtCore import QScreen
        pencere.move(
            QScreen().availableGeometry().center().x() - self.width() // 2,
            QScreen().availableGeometry().center().y() - self.height() // 2
        )

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- `from PySide6.QtWidgets import QApplication, QMainWindow, QLabel` → Etiket göstermek için `QLabel` sınıfını da içeri aktarıyoruz.
- `etiket = QLabel("Merhaba, PySide6!", self)` → 
  - İlk argüman: Etiketin göstereceği metin.
  - İkinci argüman (`self`): Etiketin **ana pencereye** bağlı olmasını sağlar; bu sayede etiket pencereyle birlikte gösterilir ve gizlenir.
- `etiket.move(150, 130)` → Etiketin pencere'nin sol üst köşesinden (0,0) koordinat bazında 150 piksel sağa, 130 piksel aşağı konumlandırır.
  > [!NOTE]
  > `move()` metodu mutlak konumlandırma yapar; bu yüzden pencere boyutu değiştiğinde etiketin konumu elle ayarlanması gerekebilir. Daha esnek bir yerleşim için layout yöneticileri (Bölüm 4) kullanacağız.

#### Görsel / İşlevsel Sonuç

```
Ekranda "OOP ile PySide6" başlıklı 400 × 300 piksellik bir pencere görürüz.
Pencerenin içinde, sol üst köşesinden 150 piksel sağ ve 130 piksel aşağıda,
"Merhaba, PySide6!" yazısı siyah renkte bir etiket olarak görünür.
```

> [!IMPORTANT]
> `QLabel` oluştururken ikinci argüman olarak `self` (ana pencere) vermek zorunludur; aksi takdirde etiket bir **bağımsız pencere** olarak açılır ve ana pencerenin bir parçası olmaz.

---

### Nihai Tam Kod Bloğu

Öğrencinin kopyalayıp doğrudan çalıştırabilecek, tüm adımların birleştiği hali:

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QLabel
from PySide6.QtCore import QScreen

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("OOP ile PySide6")
        self.resize(400, 300)

        # QLabel oluşturup metin atıyoruz
        self.etiket = QLabel("Merhaba, PySide6!", self)
        self.etiket.move(150, 130)  # Pencere içinde konumlandırma (x=150, y=130)

        # Pencereyi göster
        self.show()

        # Ekran ortalaması:
        self.move(
            QScreen().availableGeometry().center().x() - self.width() // 2,
            QScreen().availableGeometry().center().y() - self.height() // 2
        )

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

> [!TIP]
> Etiketin konumunu denemek istiyorsanız, `etiket.move(x, y)` içindeki `x` ve `y` değerlerini değiştirip kaydedip `python main.py` ile yeniden çalıştırarak anlık güncellemi görüntüleyebilirsiniz.

---

## 3. Bireysel Öğrenme Görevi (Sıra Sende)

> **Görev:** `AnaPencere` sınıfının içine ikinci bir `QLabel` ekleyip konumunu ayarlayın. Yeni etiketin metni `"İkinci Etiket"` olsun ve pencerenin sol alt kısmında görünsüz (örneğin `x=50`, `y=250`) konumlandırın.

- **Ipucu:** `QLabel` sınıfını yeniden içeri aktarmanıza gerek yok; zaten üstte import edilmiştir. Yeni bir değişken (örneğin `etiket2`) oluşturup yine `QLabel(..., self)` şeklinde oluşturun ve `move()` metodu ile konum ayarlayın.
- **Beklenen çıktı:** 
  - İlk etiket: `"Merhaba, PySide6!"` konum (150, 130)
  - İkinci etiket: `"İkinci Etiket"` konum (50, 250)
  - İki etiket de aynı pencere içinde, belirtilen koordinatlarda görünür.

Başarıyla tamamladığınızda, OOP yapısıyla sınıf içinde birçok GUI bileşenini yönetebileceğinizi ve her birini `self.` ile sınıf içinde tutarak gerekirse başka metotlardan da erişebileceğinizi görmüş olursunuz. Bir sonraki konuda bu bileşenleri düzenli bir şekilde yerleştirmek için **layout yöneticileri** (QVBoxLayout, QHBoxLayout vb.) öğreneceğiz.
