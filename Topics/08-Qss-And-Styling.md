# Stil ve Özelleştirme (QSS)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- **Qt Style Sheets (QSS)** nedir ve CSS ile ne kadar benzer olduğunu açıklayabilmek.
- `setStyleSheet()` metodu ile tek bir bileşene veya tüm uygulamaya stil uygulayabilmek.
- QSS sözdizimini kullanarak seçiciler (selector) ve durum sınıfları (`:hover`, `:pressed`, `:disabled`) tanımlayabilmek.
- Harici `.qss` dosyaları oluşturup uygulamanıza temalandırma (theming) yapabilmek.
- Koyu ve açık tema arasında dinamik geçiş yapan bir uygulama geliştirebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Şu ana kadar oluşturduğumuz uygulamalar, varsayılan işletim sistemi temasına benziyordu. Ancak modern uygulamalar, marka kimliğini yansıtmak, kullanıcı deneyimini iyileştirmek veya sadece göz alıcı olmak için özel renkler, fontlar ve efektler kullanır.

Eğer her butonun rengini, kenarlık uzunluğunu veya fare üzerine geldiğinde değişmesini tek tek kodla ayarlamaya kalkarsanız, kodunuz hızla karışık ve okunmaz hale gelir. **Qt Style Sheets (QSS)**, bu stil tanımlarını HTML/CSS'de yaptığımız gibi, ayrı bir "stil sayfasında" toplamamızı sağlar. Bu sayede:
1.  **Ayrıştırma:** Görsel tasarım (stil) ile iş mantığı (mantık) ayrı tutulur.
2.  **Tekrar Kullanılabilirlik:** Aynı stil birden fazla bileşende uygulanabilir.
3.  **Dinamik Değiştirme:** Uygulama çalışırken stil değiştirilebilir (örneğin koyu/ açık tema geçişi).

## 🔍 Birlikte Keşfedelim

Bir web sayfasını düşünün. HTML sayfası sadece yapıyı (başlıklar, paragraflar, butonlar) tanımlar. Bu yapının nasıl görüneceği ise ayrı bir CSS dosyasıyla belirlenir. Renkler, fontlar, boşluklar ve efektler tümünde CSS içerisindedir.

Qt ile de aynı mantık geçerlidir. QSS, Qt bileşenlerinizin (butonlar, etiketler, metin kutuları vb.) nasıl görüneceğini tanımlayan stil dili-dir.

> [!NOTE]
> QSS, CSS'e çok benzer olduğundan, web deneyiminiz olanlar için çok kolay öğrenilebilir. Ancak bazı farklar vardır (örneğin Qt-specific seçiciler ve durum sınıfları).

## 🛠️ Uygulama

Bu derste, basit bir giriş formunu adım adım stilize edeceğiz. İlk olarak düz kodla başlayıp, sonrada harici bir `.qss` dosyası kullanarak temaya geçeceğiz.

### Adım 1: Düz QSS ile Stil Verme (setStyleSheet)

Öncelikle, bir buton ve bir metin kutusuna doğrudan kod içinde stil vererek başlayalım.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QVBoxLayout, QLineEdit, QPushButton, QLabel)

class StilUygulamasi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("QSS ile Stil Uygulaması")
        self.resize(300, 200)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        self.etiket = QLabel("Kullanıcı Adı:")
        self.layout.addWidget(self.etiket)

        self.input_kadi = QLineEdit()
        self.layout.addWidget(self.input_kadi)

        self.buton_giris = QPushButton("Giriş Yap")
        self.layout.addWidget(self.buton_giris)

        # --- BURASI QSS KISMI ---
        # 1. Metin kutusuna doğrudan stil verme
        self.input_kadi.setStyleSheet("""
            QLineEdit {
                border: 2px solid #007BFF;
                border-radius: 5px;
                padding: 8px;
                font-size: 14px;
            }
        """)

        # 2. Butonuna doğrudan stil verme
        self.buton_giris.setStyleSheet("""
            QPushButton {
                background-color: #28A745;
                color: white;
                border: none;
                padding: 10px 20px;
                border-radius: 5px;
                font-size: 14px;
                font-weight: bold;
            }
            QPushButton:hover {
                background-color: #218838;
            }
            QPushButton:pressed {
                background-color: #1E7E34;
            }
        """)
        # --- BİTİŞ ---

def main():
    app = QApplication(sys.argv)
    pencere = StilUygulamasi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Açıklaması:**
- **`setStyleSheet(""" ... """)`**: Bu metot, bir widget'a doğrudan CSS benzeri stil kuralları uygular.
- **Seçiciler**: `QLineEdit` ve `QPushButton` gibi sınıf isimlerini kullanarak hedeflediğimizi söyleriz.
- **Durum Sınıfları (Pseudo-classes)**: `:hover` (fare üzerine geldiğinde) ve `:pressed` (basılı olduğunda) gibi durumlar için farklı stiller tanımlayabiliriz.
- **CSS Özellikleri**: `background-color`, `border`, `border-radius`, `padding`, `font-size` gibi özellikler CSS ile aynı şekilde kullanılır.

### Adım 2: Harici QSS Dosyası Kullanımı

Stillerimizi kod içinde değil, ayrı bir `.qss` dosyasında tutmak daha düzenlidir. Bu sayede tasarımcı ve geliştirici rollerini ayırabiliriz.

1.  Proje klasörünüzde `style.qss` adında bir dosya oluşturun ve içine aşağıdaki stilleri yapıştırın:

```css
/* style.qss */

/* Genel widget stilleri */
QLineEdit {
    border: 2px solid #007BFF;
    border-radius: 5px;
    padding: 8px;
    font-size: 14px;
    background-color: #F8F9FA;
}

QPushButton {
    background-color: #28A745;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    font-size: 14px;
    font-weight: bold;
}

/* Buton durum stilleri */
QPushButton:hover {
    background-color: #218838;
}

QPushButton:pressed {
    background-color: #1E7E34;
}

/* Etiket stili */
QLabel {
    font-size: 14px;
    font-weight: bold;
    margin-bottom: 5px;
}
```

2.  Şimdi bu `.qss` dosyasını uygulama seviyesinde yükleyelim:

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QVBoxLayout, QLineEdit, QPushButton, QLabel)

class StilUygulamasi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Harici QSS ile Stil Uygulaması")
        self.resize(300, 200)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        self.etiket = QLabel("Kullanıcı Adı:")
        self.layout.addWidget(self.etiket)

        self.input_kadi = QLineEdit()
        self.layout.addWidget(self.input_kadi)

        self.buton_giris = QPushButton("Giriş Yap")
        self.layout.addWidget(self.buton_giris)

        # --- HARICI QSS YÜKLEME ---
        with open("style.qss", "r") as f:
            qss_stili = f.read()
        self.setStyleSheet(qss_stili) # Uygulama seviyesinde stil uygula
        # --- BİTİŞ ---

def main():
    app = QApplication(sys.argv)
    pencere = StilUygulamasi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

### Adım 3: Koyu/Açık Tema Geçişi

Şimdi de bir buton ile temayı dinamik olarak değiştirelim.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QVBoxLayout, QLineEdit, QPushButton, QLabel)

class TemaDegistirici(QMainWindow):
    def __init__(self):
        super().__init__()
        self.acik_tema = True # Başlangıçta açık tema
        self.initUI()
        self.tema_yukle() # Başlangıç temayı uygula

    def initUI(self):
        self.setWindowTitle("Tema Değiştirici Uygulaması")
        self.resize(300, 250)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        self.etiket = QLabel("Kullanıcı Adı:")
        self.layout.addWidget(self.etiket)

        self.input_kadi = QLineEdit()
        self.layout.addWidget(self.input_kadi)

        self.buton_giris = QPushButton("Giriş Yap")
        self.layout.addWidget(self.buton_giris)

        # Tema değiştirme butonu
        self.buton_tema = QPushButton("Tema Değiştir (Açık → Koyu)")
        self.buton_tema.clicked.connect(self.tema_degistir)
        self.layout.addWidget(self.buton_tema)

    def tema_yukle(self):
        """Mevcut tema durumuna göre QSS stilini yükler."""
        if self.acik_tema:
            # Açık tema stilleri
            qss = """
                QLineEdit {
                    border: 2px solid #007BFF;
                    border-radius: 5px;
                    padding: 8px;
                    font-size: 14px;
                    background-color: #FFFFFF;
                    color: #212529;
                }
                QPushButton {
                    background-color: #28A745;
                    color: white;
                    border: none;
                    padding: 10px 20px;
                    border-radius: 5px;
                    font-size: 14px;
                    font-weight: bold;
                }
                QPushButton:hover {
                    background-color: #218838;
                }
                QPushButton:pressed {
                    background-color: #1E7E34;
                }
                QLabel {
                    font-size: 14px;
                    font-weight: bold;
                    color: #212529;
                    margin-bottom: 5px;
                }
                QMainWindow {
                    background-color: #F8F9FA;
                }
            """
        else:
            # Koyu tema stilleri
            qss = """
                QLineEdit {
                    border: 2px solid #007BFF;
                    border-radius: 5px;
                    padding: 8px;
                    font-size: 14px;
                    background-color: #343A40;
                    color: #FFFFFF;
                }
                QPushButton {
                    background-color: #28A745;
                    color: white;
                    border: none;
                    padding: 10px 20px;
                    border-radius: 5px;
                    font-size: 14px;
                    font-weight: bold;
                }
                QPushButton:hover {
                    background-color: #218838;
                }
                QPushButton:pressed {
                    background-color: #1E7E34;
                }
                QLabel {
                    font-size: 14px;
                    font-weight: bold;
                    color: #FFFFFF;
                    margin-bottom: 5px;
                }
                QMainWindow {
                    background-color: #212529;
                }
            """
        self.setStyleSheet(qss)

    def tema_degistir(self):
        """Tema durumunu toggler ve stilini yeniden yükler."""
        self.acik_tema = not self.acik_tema
        self.tema_yukle()
        # Buton metnini güncelle
        self.buton_tema.setText(
            f"Tema Değiştir ({'Koyu → Açık' if self.acik_tema else 'Açık → Koyu'})"
        )

def main():
    app = QApplication(sys.argv)
    pencere = TemaDegistirici()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Inline QSS:** İlk örneği çalıştırın. Metin kutusu ve butonun görünümünün değiştiğini gözlemleyin. Fareyi butonun üzerine getirdiğinizde rengin değişip değişmediğini kontrol edin.
2.  **Harici QSS:** `style.qss` dosyasını oluşturun ve ikinci örneği çalıştırın. Şimdi stil değiştirmek için sadece `style.qss` dosyasını düzenlemeniz yeterli olacak.
3.  **Tema Değişikliği:** Üçüncü örneği çalıştırın ve "Tema Değiştir" butonuna basın. Uygulamanın arka plan rengi ve metin renklerinin değiştiğini gözlemleyin.

## 🧠 Kendini Test Et

1. **Soru:** QSS'te bir widget'ın sadece belirli bir durumunda (örneğin devre dışı bırakılmış) stilini nasıl değiştiririz?
2. **Soru:** `setStyleSheet()` metodu ile uygulama seviyesinde stil verdiğimizde, bu stil hangi bileşenler üzerinde etkili olur?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Durum sınıfı (pseudo-class) kullanırız: `QPushButton:disabled { background-color: #CCCCCC; }`
2. **Cevap:** Uygulama içindeki **tüm** bileşenler üzerinde etkili olur. Daha spesifik bir stil (örneğin sadece bir buton için) istenirse, o bileşene doğrudan `setStyleSheet()` uygulanmalı veya seçici daha spesifik hale getirilmeli (örneğin `QPushButton#ozelButon`).

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Bir `QTextEdit` ve iki `QPushButton` ("Temizle" ve "Gönder") içeren basit bir mesaj kutusu uygulamasını oluşturun.
2.  Harici bir `style.qss` dosyası kullanarak:
    - Metin kutusunu yuvarlak köşeli ve hafif gri arka planlı yapın.
    - Butonları yuvarlak köşeli, farklı renklerde ve hover эффекти yapın.
3.  "Tema Değiştir" butonu ekleyerek kullanıcıya koyu ve açık tema arasında geçiş yapma imkanı verin.

## 📚 Daha Fazlasını Öğrenmek

- [Qt Style Sheets Official Documentation](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QWidget.html#PySide6.QtWidgets.QWidget.setStyleSheet)
- [Qt Style Sheets Syntax Reference](https://doc.qt.io/qt-6/stylesheet-syntax.html)

## 📝 Kısa Özet

- **QSS**, Qt bileşenlerini stilize etmek için CSS benzeri bir dildir.
- **`setStyleSheet()`**, tek bir bileşene veya整个应用에スタイルを適用するために使用されます。
- **Durum sınıfları** (`:hover`, `:pressed`, `:disabled`) etkileşimli stiller için kullanılır.
- **Harici `.qss` dosyaları**, stil ve mantığı ayrı tutmak için önerilen yaklaşımdır.
- **Tema geçişi**, farklı QSS string'lerini çalışma zamanında yükleyerek elde edilir.

---
[⬅️ Önceki Ders](07-Windows-And-Dialogs.md) | [Sonraki Ders ➡️](09-Data-Listing-Widgets.md)
