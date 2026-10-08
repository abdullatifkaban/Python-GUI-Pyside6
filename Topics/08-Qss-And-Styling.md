# 08 - Stil ve Özelleştirme (QSS)

## 1. Stil ve Özelleştirme (QSS) — Kavramsal Temeller

PySide6 uygulamalarının görünümünü özelleştirmek için **Qt Style Sheets (QSS)** kullanılır. QSS, CSS'e benzer bir sözdizimiyle widget'ların renkleri, kenarlıkları, yazı tipleri ve durum stilleri gibi özelliklerini değiştirmeye olanak tanır. Böylece uygulama tutarlı ve çekici bir görünüme kavuşur; ayrıca koyu ve açık tema geçişleri gibi dinamik değişiklikler kolayca yapılabilir.

### Qt Style Sheets (QSS) Nedir? CSS ile Benzerlikleri ve Farkları

QSS, PySide6'te widget'ların görünümünü tanımlamak için kullanılan bir stillendirme dili olup, yapı ve sözdizimi açısından CSS'e benzer. Ancak bazı farklar vardır; örneğin Qt'ye özgü seçiciler ve durum pseudosınıfları (`:hover`, `:pressed`, `:disabled`) mevcuttur.

> [!NOTE]
> QSS, `setStyleSheet()` metodu ile tek bir widget veya整个 uygulama için uygulanabilir. Harici `.qss` dosyaları da yüklenerek stiller merkezi bir yerden yönetilebilir.

### setStyleSheet() Metodu ile Eleman Bazlı Stil Verme

`setStyleSheet()` metodu, bir widget'ın stilini doğrudan değiştirmek için kullanılır. Bu metodu bir widget'ın örnek üzerinden çağırarak sadece o widget'ı etkileyen stiller tanımlayabiliriz; aynı stilin başka widget'lara uygulanması isteniyorsa her birine ayrı ayrı veya merkezi bir şekilde uygulanmalıdır.

| Özellik | Açıklama |
|---------|----------|
| Kapsam | Tek bir widget veya tüm uygulama |
| Yazım Yeri | Python kodu içinde doğrudan veya harici `.qss` dosyası |
| Dinamiklik | Çalışma zamanında değiştirilebilir |
| Kalıtım | Alt widget'lar üst widget'ın stilinden etkilenebilir (CSS benzeri kalıtım) |

> [!TIP]
> Stil değişikliklerini `setStyleSheet()` ile yaparken, mevcut stilin üzerine yazılır; eski stilin korunması isteniyorsa önceki stil bilgisi saklanıp birleştirilmelidir.

### QSS Sözdizimi: Selector, State Pseudoclass (:hover, :pressed, :disabled)

QSS'te stiller, **seçiciler** (selector) ve **durum pseudosınıfları** (state pseudoclass) ile tanımlanır. Seçiciler hangi widget'ların etkilenmesini belirler; durum pseudosınıfları ise widget'ın belirli bir durumunda (fare üzeri, basılı, devre dışı) uygulanacak stilleri belirtir.

| Selector | Açıklama |
|----------|----------|
| `QPushButton` | Tüm `QPushButton` widget'ları |
| `QPushButton#okButton` | `objectName`'i `okButton` olan `QPushButton` |
| `QPushButton[flat="true"]` | `flat` özelliği `true` olan `QPushButton` |
| `QPushButton:hover` | Fare üstünde olan `QPushButton` |
| `QPushButton:pressed` | Basılı olan `QPushButton` |
| `QPushButton:disabled` | Devre dışı bırakılmış `QPushButton` |

> [!WARNING]
> State pseudoclass'lar seçiciye directamente eklenmelidir; boşluk bırakılmamalıdır. Örneğin `QPushButton :hover` geçersizdir; doğru kullanım `QPushButton:hover` olur.

### Harici .qss Dosyalarını Okuma ve Uygulama

Stilleri har bir `.qss` dosyasında tutmak, kodla stil ayırmak ve daha temiz bir yapı sağlamak için uygundur. Bu dosya, çalışma zamanında okunarak `setStyleSheet()` metodu ile uygulanır.

> [!IMPORTANT]
> Harici `.qss` dosyası okunurken dosya yolunun mutlak veya çalışma dizinine göreceli olduğundan emin olun; aksi takdirde `FileNotFoundError` ile karşılaşılır.

### Koyu/Açık Tema (Dark/Light Mode) Geçiş Mantığı

Uygulamada tema geçişi yapmak için iki farklı `.qss` dosyası hazırlanabilir: birisi koyu tema, diğeri açık tema için. Kullanıcı bir geçiş butonu (toggle) ile temayı değiştirdiğinde, ilgili `.qss` dosyası yeniden yüklenip `setStyleSheet()` ile uygulanır. Böylece tüm uygulamanın görünümü anında güncellenir.

> [!TIP]
> Tema geçişini yaparken, eski stilin tamamen kaldırılması gerekebilir; bunun için önce boş bir string (`""`) ile stil sıfırlanabilir, ardından yeni stil uygulanabilir.

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Bir buton ve bir metin kutusu (`QLineEdit`) üzerinde stil uygulayacağız. İlk olarak doğrudan kod içinde `setStyleSheet()` ile renk ve kenarlık vermeyi göreceğiz. Daha sonra `:hover` ve `:pressed` durumları için efektler tanımlayacağız. Son olarak harici bir `style.qss` dosyası oluşturup tüm uygulamaya uygulayacağız. Görev olarak da bir geçiş butonu (toggle) ekleyerek uygulamanın temasını koyu ve açık arasında değiştireceğiz.

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: Buton ve metin kutusuna doğrudan kod içinde setStyleSheet() ile renk ve kenarlık verme

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget, QPushButton, QLineEdit, QVBoxLayout

class AnaPencere(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("QSS Örneği - Temel Stil")
        self.resize(300, 150)

        self.layout = QVBoxLayout()
        self.setLayout(self.layout)

        self.button = QPushButton("Tıkla")
        self.line_edit = QLineEdit()
        self.line_edit.setPlaceholderText("Bir şey yazın...")

        self.layout.addWidget(self.button)
        self.layout.addWidget(self.line_edit)

        # Buton için doğrudan stil
        self.button.setStyleSheet("""
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border: none;
                padding: 8px 16px;
                font-size: 14px;
                border-radius: 4px;
            }
            QPushButton:hover {
                background-color: #45a049;
            }
            QPushButton:pressed {
                background-color: #3d8b40;
            }
        """)

        # Metin kutusu için doğrudan stil
        self.line_edit.setStyleSheet("""
            QLineEdit {
                border: 2px solid #ccc;
                border-radius: 4px;
                padding: 6px;
                font-size: 14px;
            }
            QLineEdit:focus {
                border-color: #4CAF50;
            }
        """)

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- **QPushButton stil Yeşil tonlar:** Arka plan yeşil, yazı beyaz, kenarlık yok, iç boşluk 8x16 piksel, yazı boyutu 14px, köşe yuvarlaklığı 4px.
- **:hover durumu:** Fare buton üstünde olduğunda arka plan daha koyu yeşile (`#45a049`) değişir.
- **:pressed durumu:** Buton basılıyken arka plan en koyu yeşile (`#3d8b40`) değişir.
- **QLineEdit stil:** Gri kenarlık (2px), iç boşluk 6px, yazı boyutu 14px, köşe yuvarlaklığı 4px. Odaklandığında (`:focus`) kenarlık yeşile (`#4CAF50`) değişir.
- Bu stiller doğrudan Python kodunda tanımlanarak widget'ların görünümü anında etkilenir.

#### Görsel / İşlevsel Sonuç

```
Ekranda "QSS Örneği - Temel Stil" başlıklı bir pencere görünür.
Üstte yeşil arkaplanlı, beyaz yazılı "Tıkla" butonu bulunur.
Butonun üzerinde fare beklediğinde arka parlak daha koyu yeşile, basıldığında daha da koyu tonlara geçer.
Altta bir metin kutusu (QLineEdit) vardır; kutunun kenarığı gri ve yuvarlak köşelidir.
Kutucuğa tıklandığında odak olur ve kenarlık yeşile değişir.
Kutucuğa bir metin yazılabilir; buton henüz herhangi bir işlev görmez.
```

---

### Adım 2: Hover (:hover) ve basılma (:pressed) durumları için efektler tanımlama

> **Nedir/Ne işe yarar?** Kullanıcı bir widget'ın üzerine gelindiğinde veya basındığında farklı bir görünüm göstermek, arayüzü daha canlı ve etkileşimli hale getirir. QSS'te bu durumlar `:hover` ve `:pressed` pseudosınıflarıyla tanımlanır.

Bu adımda daha önceki kodda zaten yer alan `:hover` ve `:pressed` stillerini vurgulamak istiyoruz. Stil tanımları içinde bu pseudosınıflar kullanılarak widget'ın etkileşim durumlarına göre farklı arka plan, kenarlık veya yazı rengi verilebilir.

> [!TIP]
> `:hover` ve `:pressed` dışında `:disabled` (devre dışı) durumu da tanımlanabilir; bu durumda genellikle daha soluk renkler ve diminished etkiler kullanılır.

> [!WARNING]
> Fazla efekt kullanmak (gölge, animasyon vb.) performansı olumsuz etkileyebilir; basit renk ve kenarlık değişiklikleri genellikle yeterlidir.

#### Ek Açıklama: Daha Fazla Efekt Örneği

İsterseniz stil tanımlarına ek özellikler ekleyebilirsiniz:

```css
QPushButton {
    background-color: #2196F3;
    color: white;
    border: none;
    padding: 10px;
    font-size: 13px;
}
QPushButton:hover {
    background-color: #1976D2;
}
QPushButton:pressed {
    background-color: #0D47A1;
}
QPushButton:disabled {
    background-color: #BDBDBD;
    color: #757575;
}
```

Bu şekilde butonun normal, hover, pressed ve disabled stilleri ayrı ayrı tanımlanabilir.

---

### Adım 3: Harici bir style.qss dosyası oluşturup tüm uygulamaya app.setStyleSheet() ile temalandırma uygulama

Stilleri kod içinde değil, harici bir `.qss` dosyasında tutmak daha okunabilir ve yeniden kullanılabilir bir yapı sağlar. Bu dosyayı okuyarak `QApplication` örneğine uygulayarak tüm uygulama üzerindeki stil değişikliğini yapabiliriz.

#### Eklenecek Kod

Önce `style.qss` dosyasını oluşturalım (aynı dizinde):

```css
/* style.qss */
QPushButton {
    background-color: #FF9800;
    color: white;
    border: none;
    padding: 8px 14px;
    font-size: 14px;
    border-radius: 4px;
}
QPushButton:hover {
    background-color: #FB8C00;
}
QPushButton:pressed {
    background-color: #F57C00;
}
QLineEdit {
    border: 2px solid #9E9E9E;
    border-radius: 4px;
    padding: 6px;
    font-size: 14px;
}
QLineEdit:focus {
    border-color: #FF9800;
}
```

Şimdi bu dosyayı okuyarak uygulamaya uygulayan Python kodu:

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget, QPushButton, QLineEdit, QVBoxLayout
from PySide6.QtCore import QFile

class AnaPencere(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("QSS Örneği - Harici Dosya")
        self.resize(300, 150)

        self.layout = QVBoxLayout()
        self.setLayout(self.layout)

        self.button = QPushButton("Tıkla")
        self.line_edit = QLineEdit()
        self.line_edit.setPlaceholderText("Bir şey yazın...")

        self.layout.addWidget(self.button)
        self.layout.addWidget(self.line_edit)

        # Harici QSS dosyasını yükle ve uygula
        style_file = QFile("style.qss")
        if style_file.open(QFile.ReadOnly | QFile.Text):
            style_stream = style_file.readAll()
            style_qss = str(style_stream, encoding='utf-8')
            QApplication.instance().setStyleSheet(style_qss)
            style_file.close()

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- **QFile ile dosya okuma:** `QFile("style.qss")` nesnesi oluşturulur, `ReadOnly` ve `Text` modunda açılır; okunan veri `readAll()` ile alınır ve `utf-8` ile decode edilir.
- **Uygulama seviyesinde stil:** `QApplication.instance().setStyleSheet(style_qss)` çağrısıyla oluşturulan stil string'i tüm uygulamaya uygulanır; bu sayede tüm `QPushButton` ve `QLineEdit` widget'ları aynı stilleri alır.
- **Dosya yolu:** `style.qss` dosyası Python betiğiyle aynı dizinde bulunmalıdır; aksi takdirde dosya bulunamadı hatası alınır.
- **instance() kullanımı:** `QApplication.instance()` zaten oluşturulmuş uygulama örneğini alır; bu sayede `setStyleSheet()` uygulama genelinde etkili olur.

#### Görsel / İşlevsel Sonuç

```
Ekranda "QSS Örneği - Harici Dosya" başlıklı bir pencere görünür.
Buton turuncu arkaplanlı, beyaz yazılı; fare üstünde daha açık turuncu, basıldığında daha koyu tonlar görünür.
Metin kutusunun kenarığı gri ve yuvarlak köşeli; odaklandığında kenarlık turuncu olur.
Kullanıcı metin kutusuna yazabilir, buton etkileşimde hâlâ bir işlev görmez.
```

---

### Adım 4: Uygulamada koyu ve açık tema geçişi (toggle butonu) - Sıra Sende Görevi

> **Sıra Sende Görevi:** Bir geçiş butonu (Toggle) koyarak uygulamanın temasını Koyu ve Açık arasında dinamik değiştiren slot yazma.

Bu görevi tamamlamak için iki ayrı `.qss` dosyası hazırlayacağız: birisi `dark.qss` (koyu tema), diğeri `light.qss` (açık tema). Kullanıcı bir buton ile temayı değiştirdiğinde, ilgili dosya okunup `setStyleSheet()` ile uygulanır.

#### İlk Adım: Tema Dosyalarını Oluşturma

**dark.qss** (koyu tema):
```css
/* dark.qss */
QWidget {
    background-color: #2E2E2E;
    color: #FFFFFF;
}
QPushButton {
    background-color: #0D47A1;
    color: white;
    border: none;
    padding: 8px 14px;
    font-size: 14px;
    border-radius: 4px;
}
QPushButton:hover {
    background-color: #1565C0;
}
QPushButton:pressed {
    background-color: #0A3D91;
}
QLineEdit {
    background-color: #424242;
    border: 2px solid #9E9E9E;
    border-radius: 4px;
    padding: 6px;
    font-size: 14px;
    color: #FFFFFF;
}
QLineEdit:focus {
    border-color: #0D47A1;
}
```

**light.qss** (açık tema):
```css
/* light.qss */
QWidget {
    background-color: #FFFFFF;
    color: #212121;
}
QPushButton {
    background-color: #1976D2;
    color: white;
    border: none;
    padding: 8px 14px;
    font-size: 14px;
    border-radius: 4px;
}
QPushButton:hover {
    background-color: #1565C0;
}
QPushButton:pressed {
    background-color: #0D47A1;
}
QLineEdit {
    background-color: #FFFFFF;
    border: 2px solid #9E9E9E;
    border-radius: 4px;
    padding: 6px;
    font-size: 14px;
    color: #212121;
}
QLineEdit:focus {
    border-color: #1976D2;
}
```

#### İkinci Adım: Temayı Değiştiren Slot Fonksiyonu

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget, QPushButton, QLineEdit, QVBoxLayout, QCheckBox
from PySide6.QtCore import QFile

class AnaPencere(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("QSS Tema Geçişi")
        self.resize(350, 150)

        self.layout = QVBoxLayout()
        self.setLayout(self.layout)

        self.button = QPushButton("Tıkla")
        self.line_edit = QLineEdit()
        self.line_edit.setPlaceholderText("Bir şey yazın...")
        self.toggle_theme = QCheckBox("Koyu Tema")
        self.toggle_theme.setChecked(False)  # Başlangıçta açık tema
        self.toggle_theme.stateChanged.connect(self.tema_degistir)

        self.layout.addWidget(self.button)
        self.layout.addWidget(self.line_edit)
        self.layout.addWidget(self.toggle_theme)

        # Başlangıçta açık temayı uygula
        self.tema_yukle("light.qss")

    def tema_yukle(self, qss_dosyasi):
        style_file = QFile(qss_dosyasi)
        if style_file.open(QFile.ReadOnly | QFile.Text):
            style_stream = style_file.readAll()
            style_qss = str(style_stream, encoding='utf-8')
            QApplication.instance().setStyleSheet(style_qss)
            style_file.close()
        else:
            print(f"Stil dosyası bulunamadı: {qss_dosyasi}")

    def tema_degistir(self, state):
        if state == 2:  # Checked
            self.tema_yukle("dark.qss")
        else:  # Unchecked
            self.tema_yukle("light.qss")

# Uygulama döngüsü
app = QApplication(sys.argv)
pencere = AnaPencere()
pencere.show()
sys.exit(app.exec())
```

#### Kod Analizi / Bu Kod Ne Sağlar?

- **QCheckBox:** Kullanıcı temayı değiştirmek için bir onay kutusu (toggle) kullanılır; durum değişikliği `stateChanged` sinyaliyle `tema_degistir` slotuna bağlanır.
- **`tema_yukle()` metodu:** Belirtilen `.qss` dosyasını okuyarak uygulama seviyesinde stil uygular; dosya yoksa hata mesajı yazdırılır.
- **`tema_degistir()` metodu:** Onay kutusu işaretliyse (`state == 2`) `dark.qss`, işaretli değilse `light.qss` dosyası yüklenir.
- **Başlangıç teması:** Nesne oluşturulurken `light.qss` yüklenerek açık tema ile başlatılır.
- **Uygulama seviyesi stil değişikliği:** `QApplication.instance().setStyleSheet()` ile tüm widget'lar anında yeni stilleri alır.

#### Görsel / İşlevsel Sonuç

```
Ekranda "QSS Tema Geçişi" başlıklı bir pencere görünür.
Üstte bir buton, altında bir metin kutusu ve onay kutusu "Koyu Tema" bulunur.
Başlangıçta onay kutusu işaretli değil, açık tema aktif: arka plan beyaz, yazı koyu gri, buton mavi, metin kutusu beyaz arkaplan gri kenarlık.
Kullanıcı onay kutusunu işaretlerse (Koyu Tema):
    - Pencere arka planı koyu gri (#2E2E2E) olur.
    - Yazı beyaz olur.
    - Buton koyu mavi (#0D47A1), hoverda daha açık mavi, basılıyken daha koyu mavi.
    - Metin kutusu arka planı daha koyu gri (#424242), yazı beyaz, kenarlık gri, odaklandığında mavi kenarlık.
Kullanıcı onay kutusunu işaretini kaldırırsa yeniden açık temaya döner.
```

## 3. Sıra Sende (Pekiştirme Görevi)

- **Görev:** Bir geçiş butonu (Toggle) koyarak uygulamanın temasını Koyu ve Açık arasında dinamik değiştiren slot yazma.
- **İpucu:** `> [!TIP]` İki ayrı `.qss` dosyası (`dark.qss` ve `light.qss`) oluşturun; bunları okuyarak `QApplication.instance().setStyleSheet()` ile uygulamaya uygulayın. Temayı değiştiren bir `QCheckBox` veya `QToggleButton` kullanarak `stateChanged` sinyalini bir slot fonksiyonuna bağlayın; slot içinde ilgili dosyayı seçip stil yapın.
- **Beklenen Çıktı:** Kodu çalıştırdığınızda, bir buton, bir metin kutusu ve bir "Koyu Tema" onay kutusu görürünüz. Onay kutusu işaretli olduğunda uygulama koyu temaya, işaretli değilse açık temaya geçer; geçiş anında ve sorunsuz olur.

> [!TIP]
> Temada kullanmak istediğiniz widget'ları seçmek için nesne adı (`objectName`) veya özelliği bazlı seçiciler de kullanabilirsiniz; örneğin `QPushButton#toggleButton { ... }` sadece belirli bir butonu hedef alır.