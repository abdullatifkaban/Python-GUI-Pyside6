# 05 - Görsel Tasarım Aracı - Qt Designer

## 1. Görsel Tasarım Aracı - Qt Designer — Kavramsal Temeller

Gelişmiş arayüzler oluştururken widget'ları tek tek kod ile eklemek ve düzenlemek zaman alıcı ve hata açıktır. İşte bu noktada **Qt Designer** devreye girer; sürükle-bırak ile arayüz tasarlayarak .ui dosyası oluştururuz ve bu dosyayı PySide6 ile çalışma zamanında yükleyerek kullanırız. Böylece tasarım ve mantık ayrı ayrı yönetilir, güncellemeler daha kolay olur.

### Qt Designer Nedir? Kurulumu ve Arayüz Bileşenleri

Qt Designer, Qt ekosunun bir parçası olan görsel bir arayüz tasarım aracıdır. Widget'ları bir forma sürükleyip, özelliklerini (property) düzenleyerek hızlıca prototip oluşturabiliriz. PySide6 ile birlikte kurulan `pyside6-designer` komutuyla başlatılır veya Qt'nin kendi kurulumunda bulunan Designer aracı kullanılır.

> [!TIP]
> Qt Designer'da bir widget seçildiğinde sağ özellik penceresinde (Property Editor) o widget'ın tüm ayarlarını görür ve değiştirebiliriz; bu da her bir özellik için kod yazmak zorunda kalmayımızı sağlar.

### .ui Dosya Formatı ve XML Yapısı

Qt Designer'da tasarlanan arayüzler `.ui` uzantılı XML dosyası olarak kaydedilir. Bu dosya, widget'ların hiyerarşisi, özellikleri ve düzenleri (layout) hakkında ayrıntılı bilgiler içerir. PySide6 bu XML'i çalışma zamanında okuyarak ilgili widget örneklerini oluşturur.

| Özellik | Açıklama |
|---------|----------|
| Format | XML tabanlı |
| İçerik | Widget sınıfları, özellikleri (objectName, geometry, styleSheet vb.), layout bilgileri |
| Kullanım | `QUiLoader` veya `pyside6-uic` ile Python koduna dönüştürülür |

> [!NOTE]
> .ui dosyası içindeki `objectName` değerleri, Python kodundan bu widget'lara erişmek için kritiktir; bu isimler `findChild` veya doğrudan erişim (ui.widget_adi) ile kullanılır.

### PySide6 İçinde .ui Yükleme Yöntemleri: QUiLoader vs pyside6-uic

PySide6'de .ui dosyasını iki ana şekilde yükleyebiliriz:

1. **QUiLoader** (Çalışma Zamanı Yükleme)
   - `.ui` dosyasını doğrudan çalışma zamanında yükler ve widget hiyerarjisini döndürür.
   - Değişiklikleri .ui dosyasında yapıp uygulamayı yeniden başlattığımızda güncellenen tasarımı görürüz (derleme gerekmez).
   - Daha dinamik ve esnektir; tasarım sıkça değişiyorsa tercih edilir.

2. **pyside6-uic** (Derleme Zamanı Dönüşümü)
   - `.ui` dosyası önceden Python koduna dönüştürülür (örneğin `pyside6-uic tasarim.ui -o ui_tasarim.py`).
   - Oluşan Python modülü doğrudan içe aktarılır; bu da çalışma zamanındaki XML ayrıştırma yükünü ortadan kaldırır.
   - Tasarım değiştiğinde dönüşümü tekrarlamak gerekir.

> [!IMPORTANT]
> Öğrenme ve hızlı prototipleme için `QUiLoader` tercih edilir; çünkü .ui dosyasını her değiştirmek tek bir komutla güncellenir. Performans kritik uygulamalarda önceden üretilen Python modülü daha hızlı olabilir.

### Dinamik Tasarım Dosyası Yüklemenin Avantajları

`QUiLoader` ile çalışma zamanında .ui dosyası yüklemek şu avantajlar sağlar:

- **Tasarım-Geliştirme Ayrılığı:** Tasarımcı .ui dosyasını düzenlerken geliştirici mantık kodunu değiştirmez.
- **Hızlı İterasyon:** .ui dosyasında bir değişiklik yapıp uygulamayı yeniden başlatmak yeterlidir; derleme adımı yoktur.
- **Esneklik:** Farklı .ui dosyaları koşullu olarak yüklenerek çoklu dil veya tema desteği kolaylaşır.
- **Az Boilerplate:** Widget'ları tek tek oluşturup özelliklerini ayarlamak yerine tek bir yükleme çağrısı yeterlidir.

> [!WARNING]
> `QUiLoader` kullanırken .ui dosyasının yolunun mutlak veya çalışma dizinine göreceli olduğundan emin olmalıyız; aksi takdirde `FileNotFoundError` ile karşılaşırız.

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Qt Designer ile oluşturulmuş basit bir `tasarim.ui` dosyasını QUiLoader ile yükleyeceğiz. Bu dosyada bir `QLineEdit`, bir `QLabel` ve bir `QPushButton` bulunacak. Kullanıcı `QLineEdit` metnini girip butona bastığında, metin label'a kopyalanacaktır. Ayrıca .ui dosyasındaki widget'lara Python kodundan `findChild` yoluyla erişip sinyal/slot bağlayacağız. Son olarak, sırasımdaki görev olarak yeni bir .ui dosyasına `QLineEdit` ekleyip yazılan metni konsola yazdıracak bağlantıyı kuracağız.

> [!TIP]
> Bu adımları VS Code'daki `main.py` ve `tasarim.ui` dosyalarında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: Qt Designer ile oluşturulmuş basit bir tasarim.ui dosyasını QUiLoader ile Python'a yükleme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication
from PySide6.QtUiTools import QUiLoader
from PySide6.QtCore import QFile

class AnaPencere:
    def __init__(self):
        # QUiLoader ile .ui dosyasını yükle
        loader = QUiLoader()
        ui_file = QFile("tasarim.ui")
        ui_file.open(QFile.ReadOnly)
        self.pencere = loader.load(ui_file)
        ui_file.close()
        self.pencere.setWindowTitle("Qt Designer Örneği")

        # .ui dosyasındaki widget'lara erişim
        self.line_edit = self.pencere.findChild(QLineEdit, "lineEdit")
        self.label = self.pencere.findChild(QLabel, "label")
        self.button = self.pencere.findChild(QPushButton, "pushButton")

        # Butonun clicked sinyalini slotumuza bağla
        self.button.clicked.connect(self.goster)

    def goster(self):
        metin = self.line_edit.text()
        self.label.setText(metin)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi

- **QUiLoader ve QFile:** `QUiLoader` örneği oluşturulur; `QFile` ile `tasarim.ui` dosyası okunur ve `loader.load()` çağrısıyla widget hiyerarjisi oluşturulur.
- **Widget Erişimi:** `findChild()` metodu, verilen `objectName` ve widget tipiyle .ui dosyasındaki belirli widget'ı bulur. Burada `"lineEdit"`, `"label"` ve `"pushButton"` isimli öğeler alınır.
- **Sinyal-Bağlantı:** Butonun `clicked` sinyali `self.goster` slotuna bağlanır; bu metod `QLineEdit` metnini alır ve `QLabel` metnini günceller.
- **Pencere Gösterimi:** Yüklenmiş olan üst-level widget (genellikle bir `QMainWindow` veya `QWidget`) `.show()` ile ekranda görünür hale getirilir.

#### Görsel / İşlevsel Sonuç

```
Ekranda tasarim.ui dosyasında tanımlanan pencere görünür.
Pencerenin içinde:
- Üstte metin girilecek bir kutucuk (QLineEdit, objectName: lineEdit)
- Ortasında boş bir etiket (QLabel, objectName: label)
- Altta bir düğme (QPushButton, objectName: pushButton) bulunur ve metni "Göster" olabilir.

Kullanıcı QLineEdit'e bir metin yazar ve "Göster" butonuna tıklar.
Butona tıklandığında, lineEdit'teki metin label'a kopyalanır ve anında görünür.
```

> [!NOTE]
> Yukarıdaki örnekte "whose" ifadesi bir çeviri hatasıydı ve düzeltildi. Doğru ifade: "Altta bir düğme (QPushButton, objectName: pushButton) bulunur ve metni 'Göster' olabilir."

---

### Adım 2: .ui içerisindeki widget'lara Python kodundan erişme (findChild veya ui.widget_adi)

.ui dosyasını yükledikten sonra widget'ların özelliklerini değiştirmek, metin almak/ayarlamak veya sinyal/slot bağlamak için bu widget'lara Python tarafından erişmemiz gerekir. İki yaygın yol vardır: `findChild()` ve `pyside6-uic` ile üretilen modülde doğrudan özellik erişimi (`ui.widget_adi`).

`QUiLoader` kullandığımız için `findChild()` en uygun yoldur; çünkü yükleme sonucu elde ettiğimiz nesne hiyerarjisi üzerinde arama yapar. Eğer `pyside6-uic` ile önceden üretilen bir Python modülü kullanıyorsak, o modülün içinde `ui` adlı bir nesne bulunur ve bu nesne üzerinden widget'lara doğrudan erişebiliriz (`ui.lineEdit`, `ui.label` vb.).

> [!TIP]
> `findChild()` çağrısında widget tipi belirtilmezse (`self.pencere.findChild(QObject, "lineEdit")`) tüm nesneler arasında arama yapar; tip belirtilmesi performansı artırır ve yanlış eşleşmeleri önler.

> [!WARNING]
> Eğer .ui dosyasındaki bir widget'ın `objectName`'i boş ise veya duplicated ise, `findChild()` sadece ilk eşleşeni döndürür; bu yüzden .ui dosyasındaki tüm isimlerin benzersiz ve anlamlı olduğundan emin olmamız gerekir.

#### Ek Açıklama: ui.widget_adi Kullanımı (pyside6-uic)

Eğer tercih edersek `pyside6-uic` komutuyla .ui dosyasını Python koduna dönüştürebiliriz:

```bash
pyside6-uic tasarim.ui -o ui_tasarim.py
```

Oluşan `ui_tasarim.py` modülü içinde bir `Ui_MainWindow` (veya tasarımımızın üst-sınıfına göre) sınıfı bulunur. Bu sınıfı kendi pencere sınıfımıza çok kalım (multiple inheritance) ya da içer olarak kullanırız:

```python
from ui_tasarim import Ui_MainWindow
from PySide6.QtWidgets import QMainWindow

class AnaPencere(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)  # .ui üzerinden widget'ları oluşturur ve self'e bağlar
        # Şimdi doğrudan erişim:
        self.lineEdit.setText("Merhaba")
        self.pushButton.clicked.connect(self.goster)
```

Bu yöntem, çalışma zamanındaki XML ayrıştırma yükünü ortadan kaldırır; ancak .ui dosyası değiştirildiğinde dönüşümü tekrarlamak gerekir.

---

### Adım 3: Yüklenen tasarımdaki butona Python tarafında sinyal/slot bağlayarak etkileşim katma

Bir web sayfasındaki bir butona tıklayınca bir işlem yapmak için JavaScript'te `addEventListener` kullanırız; PySide6'te de benzer mantıkla widget'ın sinyallerini (olayları) kendi fonksiyonlarımıza (slotlara) bağlarız.

Bu adımda daha önceki kodda zaten yer alan `self.button.clicked.connect(self.goster)` satırını vurgulamak istiyoruz. Butonun `clicked` sinyali, kullanıcı farla butona bastığında tetiklenir; bu sinyal `self.goster` metodunu çalıştırır. Metod içinde `QLineEdit` metnini alır ve `QLabel`'a yazar; böylece etkileşim sağlanır.

> [!NOTE]
> Sinyal-slot mekanizması PySide6'nın temelidir; birden fazla sinyal tek bir slota, veya tek bir sinyal birden fazla slota bağlanabilir. Ayrıca özel (custom) sinyaller tanımlayıp kendi widget'larımızda da kullanabiliriz.

> [!TIP]
> Lambda fonksiyonlarıyla hızlı bağlantılar kurulabilir: `self.button.clicked.connect(lambda: self.label.setText(self.lineEdit.text()))`. Ancak okunabilirlik adına ayrı bir slot metodu tercih edilir.

---

## 3. Sıra Sende (Pekiştirme Görevi)

- **Görev:** Qt Designer'da yeni bir .ui dosyasına QLineEdit ekleyip Python'da yazılan metni konsola yazdıracak bağlantıyı kurma.
- **İpucu:** Qt Designer'ı açarak yeni bir Form oluşturun, içine bir `QLineEdit` (objectName: consoleLineEdit) ve bir `QPushButton` (objectName: printButton) yerleştirin. Butonun `clicked` sinyali bir slot fonksiyonuna bağlayıp, o fonksiyonda `self.consoleLineEdit.text()` ile metni alın ve `print()` fonksiyonuyla konsola yazdırın.
- **Beklenen Çıktı:** Kodu çalıştırdığınızda, pencereye bir metin kutusu ve bir "Yazdır" butonu görürünüz. Kutuyu bir metin doldurup butona tıkladığınızda, aynı metin konsola (terminal) yazdırılmalıdır.

> [!TIP]
> Konsola yazdırma işlemi için `print()` fonksiyonu yeterlidir; ancak gerçek uygulamalarda bu bilgiyi bir dosyaya yazmak veya başka bir sisteme göndermek istenebilir.