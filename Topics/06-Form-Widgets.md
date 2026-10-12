# Temel Form Elemanları

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Kullanıcıdan farklı veri türlerini almak için uygun widget'ları (`QLineEdit`, `QTextEdit`, `QCheckBox`, `QRadioButton`, `QComboBox`, `QSpinBox`) seçebilmek.
- Form elemanlarından gelen verileri Python koduna çekebilmek.
- Kullanıcı hatalarını önlemek için basit veri doğrulama (validation) tekniklerini uygulayabilmek.
- Form elemanlarını mantıksal gruplar halinde düzenleyebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Bir uygulamanın kullanıcıyla konuştuğu en temel yer **formlardır**. Bir kullanıcı adı istemek, bir onay kutusuyla sözleşmeyi kabul ettirmek veya bir sayı seçtirmek... Eğer yanlış widget seçerseniz, kullanıcı deneyimi (UX) kötüleşir. 

Örneğin; tek bir kelimelik isim girişi için devasa bir metin alanı (`QTextEdit`) kullanmak, kullanıcının kafasını karıştırır ve "buraya çok fazla şey yazabilirim" algısı yaratarak hatalara yol açar. Doğru bileşeni doğru amaçla kullanmak, profesyonel bir arayüzün ilk kuralıdır.

## 🔍 Birlikte Keşfedelim

Bir otel kayıt formunu hayal edin. Formdaki her alan farklı bir "veri tipi" gerektirir:

1.  **İsim-Soyisim:** Kısa, tek satırlık bir cevap.
2.  **Adres/Notlar:** Uzun, birden fazla satıra yayılan bir cevap.
3.  **Cinsiyet:** Sadece bir seçenek seçilebilen bir durum.
4.  **Oda Tipi:** Bir listeden seçim yapılması gereken bir durum.
5.  **Kişi Sayısı:** Sadece belirli sayı aralıklarında (örn: 1-5) girilebilen bir değer.

Her birinin karşılığı, PySide6 dünyasında farklı bir widget'tır.

> [!NOTE]
> Yanlış widget seçimi sadece estetik bir sorun değildir; aynı zamanda verinin tipini (string, int, bool) yönetirken kodunuzun hata vermesine de neden olabilir.

## 🛠️ Uygulama

Bu derste, bir "Kullanıcı Kayıt Formu" inşa edeceğiz.

### Adım 1: Widget Seçimi ve Oluşturma

Aşağıdaki tablodaki eşleşmeleri kullanarak bileşenlerimizi tanımlayacağız:

| Veri Türü | PySide6 Widget'ı | Kullanım Amacı |
| :--- | :--- | :--- |
| Tek satır metin | `QLineEdit` | İsim, e-posta, şifre gibi kısa girdiler. |
| Çok satırlı metin | `QTextEdit` | Adres, biyografi gibi uzun metinler. |
| Onay/Seçim | `QCheckBox` | "Şartları kabul ediyorum" gibi evet/hayır durumları. |
| Tekli seçim | `QRadioButton` | "Erkek/Kadın" gibi birbirini dışlayan seçenekler. |
| Liste seçimi | `QComboBox` | "Şehir seçiniz" gibi hazır listeden seçimler. |
| Sayısal değer | `QSpinBox` | Yaş, adet gibi belirli aralıktaki sayılar. |

### Adım 2: Formu İnşa Etme ve Veri Okuma

Aşağıdaki kod, bu bileşenleri bir araya getirir ve "Kaydet" butonuna basıldığında verileri konsola yazdırır.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QLineEdit, QTextEdit, QCheckBox, QRadioButton, 
                             QComboBox, QSpinBox, QPushButton, QLabel)

class KayitFormu(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Kullanıcı Kayıt Formu")
        self.resize(350, 500)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        # 1. İsim (QLineEdit)
        self.layout.addWidget(QLabel("Ad Soyad:"))
        self.input_isim = QLineEdit()
        self.layout.addWidget(self.input_isim)

        # 2. Notlar (QTextEdit)
        self.layout.addWidget(QLabel("Notlar:"))
        self.input_notlar = QTextEdit()
        self.layout.addWidget(self.input_notlar)

        # 3. Cinsiyet (QRadioButton)
        self.layout.addWidget(QLabel("Cinsiyet:"))
        self.rb_erkek = QRadioButton("Erkek")
        self.rb_kadin = QRadioButton("Kadın")
        self.layout.addWidget(self.rb_erkek)
        self.layout.addWidget(self.rb_kadin)

        # 4. Şehir (QComboBox)
        self.layout.addWidget(QLabel("Şehir:"))
        self.combo_sehir = QComboBox()
        self.combo_sehir.addItems(["İstanbul", "Ankara", "İzmir", "Bursa"])
        self.layout.addWidget(self.combo_sehir)

        # 5. Yaş (QSpinBox)
        self.layout.addWidget(QLabel("Yaş:"))
        self.spin_yas = QSpinBox()
        self.spin_yas.setRange(1, 120) # Yaş aralığı sınırlama
        self.layout.addWidget(self.spin_yas)

        # 6. Onay (QCheckBox)
        self.check_onay = QCheckBox("Kullanım şartlarını kabul ediyorum.")
        self.layout.addWidget(self.check_onay)

        # 7. Kaydet Butonu
        self.btn_kaydet = QPushButton("Kaydet")
        self.btn_kaydet.clicked.connect(self.verileri_oku)
        self.layout.addWidget(self.btn_kaydet)

    def verileri_oku(self):
        # Verileri çekme
        isim = self.input_isim.text()
        notlar = self.input_notlar.toPlainText()
        cinsiyet = "Erkek" if self.rb_erkek.isChecked() else "Kadın"
        sehir = self.combo_sehir.currentText()
        yas = self.spin_yas.value()
        onay = self.check_onay.isChecked()

        # Basit Doğrulama (Validation)
        if not isim:
            print("HATA: İsim alanı boş bırakılamaz!")
            return
        
        if not onay:
            print("HATA: Şartları kabul etmelisiniz!")
            return

        # Sonucu konsola yazdır
        print("--- Yeni Kayıt ---")
        print(f"İsim: {isim}")
        print(f"Yaş: {yas}")
        print(f"Cinsiyet: {cinsiyet}")
        print(f"Şehir: {sehir}")
        print(f"Notlar: {notlar}")
        print(f"Onaylandı mı?: {onay}")
        print("------------------")

def main():
    app = QApplication(sys.argv)
    pencere = KayitFormu()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Boş Bırakma Testi:** İsim kısmını boş bırakıp "Kaydet"e basın. Konsolda ne görüyorsunuz?
2.  **Sayı Sınırı:** `QSpinBox` (Yaş) bileşeninde 120'den büyük bir sayı girmeye çalışın. Uygulama buna izin veriyor mu?
3.  **Metin Tipi:** `QLineEdit` içine çok uzun bir paragraf yapıştırmayı deneyin. `QTextEdit` ile farkı nedir?

## 🧠 Kendini Test Et

1. **Soru:** Kullanıcının sadece sayı girmesini istediğimiz bir alan için hangi widget en uygundur?
2. **Soru:** `QRadioButton` ve `QCheckBox` arasındaki temel fark nedir?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** `QSpinBox` veya `QDoubleSpinBox` (ondalıklı sayılar için).
2. **Cevap:** `QRadioButton` kullanıldığında, aynı grup içinden sadece **bir** seçenek seçilebilir. `QCheckBox` ise kullanıcının birden fazla seçeneği aynı anda işaretlemesine izin verir.

</details>

## 🚀 Kendin Dene

**Görev:**
Mevcut formunuzu geliştirerek aşağıdaki özellikleri ekleyin:
1.  **E-posta Alanı:** Bir `QLineEdit` ekleyin.
2.  **Basit Doğrulama:** Eğer e-posta alanı boşsa veya içinde `@` karakteri yoksa, konsola "Geçersiz E-posta!" yazdırın ve kaydı durdurun.
3.  **Görsel İpucu:** `QLineEdit` için `setPlaceholderText("Örn: kullanıcı@mail.com")` metodunu kullanarak kullanıcıya ne yazması gerektiğini gösterin.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: QLineEdit](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QLineEdit.html)
- [PySide6 Documentation: QSpinBox](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QSpinBox.html)

## 📝 Kısa Özet

- **Doğru Widget Seçimi:** Verinin türüne göre (metin, sayı, seçim) uygun bileşeni kullanmak UX için kritiktir.
- **Veri Alma:** `text()`, `toPlainText()`, `value()`, `isChecked()` ve `currentText()` metotları ile verileri okuruz.
- **Doğrulama (Validation):** Veriyi işlemeden önce `if` blokları ile kontrol etmek hataları önler.

---
[⬅️ Önceki Ders](05-Qt-Designer.md) | [Sonraki Ders ➡️](07-Windows-And-Dialogs.md)
