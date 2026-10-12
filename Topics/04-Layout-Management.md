# Yerleşim Yönetimi (Layouts)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Pencere boyutu değiştiğinde bileşenlerin düzgün kalmasını sağlayan **Responsive (Duyarlı) Tasarım** mantığını kavrayabilmek.
- Dikey (`QVBoxLayout`) ve Yatay (`QHBoxLayout`) yerleşim düzenlerini kullanabilmek.
- Karmaşık arayüzler oluşturmak için **İç İçe Yerleşim (Nested Layouts)** tekniğini uygulamak.
- Elemanları belirli bir bölgeye toplamak için `addStretch()` ve boşluk yöneticilerini kullanabilmek.
- Veri giriş formları için optimize edilmiş `QFormLayout` yapısını kullanabilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Bir yazılım geliştirirken en büyük hatalardan biri, butonları veya metin kutularını pencere içinde sabit koordinatlara (örneğin: "X=100, Y=200") yerleştirmektir. Buna **Sabit Konumlandırma (Fixed Positioning)** denir.

Peki, kullanıcı pencereyi büyütürse veya küçültürse ne olur? Sabit yerleştirilmiş butonlar pencerenin bir köşesinde küçük bir nokta gibi kalır veya pencerenin dışında kalarak görünmez hale gelir. Bu, kullanıcı deneyimi açısından felakettir.

**Yerleşim Düzenleri (Layouts)**, bileşenleri koordinatlarla değil, aralarındaki ilişkilere göre yönetir. Bir pencere büyüdüğünde, layout otomatik olarak hesaplama yapar ve bileşenleri yeni boyuta göre yeniden dağıtır.

## 🔍 Birlikte Keşfedelim

Layout mantığını bir **kitaplık** benzetmesiyle anlayabiliriz:

1.  **Sabit Konumlandırma:** Kitapları kütüphaneye rastgele, sadece belirli noktalara yapıştırmak gibidir. Kütüphane büyütülürse kitaplar yerinde çakılı kalır, boşluklar anlamsız görünür.
2.  **Yerleşim Düzeni (Layout):** Kitapları raflara dizmek gibidir. Raf genişlediğinde kitaplar aralarındaki boşluğu korur veya rafa yayılır. Raf yeni bir bölme eklediğinde kitaplar otomatik olarak yer değiştirir.

> [!WARNING]
> Bir arayüz tasarlarken `widget.move(x, y)` metodunu kullanmaktan kaçının! Bu yöntem, uygulamanızı pencere boyutu değiştiğinde bozulmaya mahkum eder. Bunun yerine her zaman bir **Layout** kullanın.

## 🛠️ Uygulama

Bu derste, iç içe geçmiş layout yapısını kullanarak profesyonel görünümlü bir buton grubu oluşturacağız.

### Adım 1: Dikey Düzen (QVBoxLayout) Kurulumu

Önce bileşenleri yukarıdan aşağıya dizecek olan dikey düzeni oluşturalım.

### Adım 2: Yatay Düzen (QHBoxLayout) ile Buton Grubu

Butonları yan yana getirmek için yatay bir düzen oluşturup, bu düzeni dikey düzenin içine bir "alt bileşen" olarak yerleştireceğiz.

### Adım 3: Esneklik Sağlama (addStretch)

Butonların pencerenin en üstüne yapışık kalmasını değil, yukarıda toplu durmasını sağlamak için boşluk (stretch) ekleyeceğiz.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QPushButton, QVBoxLayout, QHBoxLayout, QLabel)

class LayoutUygulamasi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Layout Yönetimi Uygulaması")
        self.resize(400, 300)

        # 1. Merkezi Widget ve Ana Dikey Layout (Vertical Layout)
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.ana_layout = QVBoxLayout(self.merkezi_alan)

        # Başlık etiketi
        self.etiket = QLabel("Lütfen bir işlem seçin:")
        self.ana_layout.addWidget(self.etiket)

        # 2. Yatay Düzen (Horizontal Layout) Oluşturma
        # Bu layout, butonları yan yana getirmek için bir 'grup' görevi görecek.
        self.buton_grubu_layout = QHBoxLayout()

        self.buton1 = QPushButton("Tamam")
        self.buton2 = QPushButton("İptal")

        # Butonları yatay layout'a ekliyoruz
        self.buton_grubu_layout.addWidget(self.buton1)
        self.buton_grubu_layout.addWidget(self.buton2)

        # 3. İÇ İÇE YERLEŞİM (NESTED LAYOUT)
        # Yatay layout'u, ana dikey layout'un içine bir parça olarak ekliyoruz.
        self.ana_layout.addLayout(self.buton_grubu_layout)

        # 4. Esneklik (Stretch) Ekleme
        # Alttaki boşluğu doldurarak butonların ve etiketin yukarıda kalmasını sağlar.
        self.ana_layout.addStretch()

def main():
    app = QApplication(sys.argv)
    pencere = LayoutUygulamasi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Analizi:**
- `self.ana_layout.addLayout(self.buton_grubu_layout)`: Bu satır sihirli andır. Bir layout'u başka bir layout'un içine gömerek çok karmaşık arayüzler yapmamıza olanak tanır.
- `addStretch()`: Bu komut, layout içine "görünmez bir yay" yerleştirir. Bu yay, mevcut tüm boşluğu kaplar ve diğer bileşenleri (butonları) iterek yukarıda toplu tutar.

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Pencereyi Büyütün:** Uygulamayı çalıştırın ve pencereyi farenizle genişletin. Butonlar ve etiket nasıl hareket ediyor? (Yan yana kalıyorlar mı yoksa pencereye yayılıyorlar mı?)
2.  **Stretch'i Kaldırın:** `self.ana_layout.addStretch()` satırını yorum satırı yapın. Butonların yeni konumuna bakın.
3.  **Sıralamayı Değiştirin:** `self.ana_layout.addStretch()` satırını `self.ana_layout.addWidget(self.etiket)` satırının *üstüne* taşıyın. Artık her şey nerede?

## 🧠 Kendini Test Et

1. **Soru:** `QGridLayout` ile `QVBoxLayout` arasındaki temel fark nedir?
2. **Soru:** Bir layout içine `addStretch()` eklendiğinde, bu stretch ne yapar?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** `QVBoxLayout` bileşenleri sadece tek bir sütun halinde (yukarıdan aşağıya) dizer. `QGridLayout` ise bileşenleri satır ve sütunlardan oluşan bir tablo yapısında (ızgara) herhangi bir hücreye yerleştirmenize olanak tanır.
2. **Cevap:** Mevcut boş alanı tamamen doldurur. Eğer üstüne bir stretch eklerseniz elemanları alta iter, altına eklerseniz elemanları üste iter.

</details>

## 🚀 Kendin Dene

**Görev:**
`QFormLayout` kullanarak basit bir kullanıcı kayıt formu oluşturun.
1.  Yeni bir sınıf veya mevcut sınıf üzerinde çalışın.
2.  Bir `QFormLayout` oluşturun ve bunu ana layout'a ekleyin.
3.  Formun içine şu iki satırı ekleyin:
    - **Ad Soyad:** (Bir `QLineEdit` ile)
    - **E-posta:** (Bir `QLineEdit` ile)
4.  Formun altına, yine yatay bir layout içinde "Kaydet" ve "Vazgeç" butonları yerleştirin.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: Layout Management](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QLayout.html)

---
[⬅️ Önceki Ders](03-Signals-And-Slots.md) | [Sonraki Ders ➡️](05-Qt-Designer.md)
