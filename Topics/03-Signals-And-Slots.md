# Etkileşim (Sinyal & Slot)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- **Olay Güdümlü Programlama (Event-Driven Programming)** mantığını kavrayabilmek.
- Qt ekosisteminin kalbi olan **Sinyal (Signal)** ve **Slot (Yuva)** kavramlarını açıklayabilmek.
- Bir bileşenin (örneğin bir butonun) olayını, bir fonksiyonla (metotla) başarıyla bağlayabilmek.
- En sık kullanılan sinyalleri (`clicked`, `textChanged`, `returnPressed`) uygulamalarda kullanabilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Şu ana kadar oluşturduğumuz pencereler sadece "statik" görsellerdi. Bir butona basıldığında hiçbir şey olmuyor, bir metin kutusuna yazı yazıldığında uygulama bunu fark etmiyordu. Bir yazılımı "canlı" ve "etkileşimli" kılan şey, kullanıcının eylemlerine karşı verdiği tepkilerdir.

Eğer bir butona basıldığında bir işlem yapılmasını istiyorsanız, butonun "tıklandım!" diye bağırmasını (Sinyal) ve bir fonksiyonun bu bağırma sesini duyup harekete geçmesini (Slot) sağlamalısınız.

## 🔍 Birlikte Keşfedelim

Bir kapı zilini hayal edin:

1.  **Olay (Event):** Birinin kapı ziline basması.
2.  **Sinyal (Signal):** Zil mekanizmasının elektrik devresini tamamlayıp "Dinnn!" diye bir sinyal göndermesi.
3.  **Slot (Yuva):** Evin içindeki hoparlörün bu sinyali alıp sesi dışarı vermesi veya sizin zil sesini duyup kapıya gitmeniz.

Eğer zil sinyal gönderiyor ama evde kimse o sinyali dinlemiyorsa (yani sinyal bir fonksiyona bağlanmamışsa), kapıya kim gelirse gelsin hiçbir şey olmaz.

> [!TIP]
> GUI programlamada her şey bir "olay" etrafında döner. Kullanıcı hareket eder, fare hareket eder, klavyeye basılır... Hepsi birer sinyaldir!

## 🛠️ Uygulama

Bu derste, bir butona tıklandığında ekrandaki metni değiştiren interaktif bir uygulama inşa edeceğiz.

### Adım 1: Gerekli Bileşenleri Tanımlama

Önce bir buton (`QPushButton`) ve bir etiket (`QLabel`) oluşturup bunları pencereye yerleştirmeliyiz.

### Adım 2: Slot (Fonksiyon) Yazma

Butona basıldığında ne olacağını belirleyen normal bir Python metodu yazacağız.

### Adım 3: Sinyal ve Slotu Bağlama (Connecting)

En kritik adım! `buton.clicked.connect(metod_ismi)` kalıbını kullanarak bu ikisini birbirine bağlayacağız.

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel, QVBoxLayout, QWidget

class EtkilesimliPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Sinyal ve Slot Dünyası")
        self.resize(300, 200)

        # Merkezi alan oluşturma
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        
        # Düzen (Layout) oluşturma
        self.layout = QVBoxLayout(self.merkezi_alan)

        # 1. Bileşenleri oluşturma
        self.etiket = QLabel("Henüz bir şeye basılmadı...")
        self.buton = QPushButton("Bana Tıkla!")

        # Bileşenleri layout'a ekleme
        self.layout.addWidget(self.etiket)
        self.layout.addWidget(self.buton)

        # 2. SİNYAL ve SLOT bağlantısını kurma
        # 'clicked' butonun sinyalidir, 'metni_degistir' ise bizim slotumuzdur.
        self.buton.clicked.connect(self.metni_degistir)

    # 3. SLOT (Fonksiyon) tanımı
    def metni_degistir(self):
        self.etiket.setText("🎉 Harika! Butona bastın.")

def main():
    app = QApplication(sys.argv)
    pencere = EtkilesimliPencere()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Dikkat Edilmesi Gerekenler:**
- `self.buton.clicked.connect(self.metni_degistir)` satırında, fonksiyonun yanına parantez `()` **koymuyoruz**. Eğer parantez koyarsanız, fonksiyon bağlantı kurulurken değil, program açılır açılmaz hemen çalışır. Biz sadece fonksiyonun *adını* veriyoruz ki, tıklandığında *çalıştırılabilsin*.

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Bağlantıyı Koparın:** `self.buton.clicked.connect(...)` satırının başına `#` koyarak yorum satırı yapın ve programı çalıştırın. Butona bastığınızda ne oluyor? (Cevap: Hiçbir şey, çünkü sinyal boşluğa gidiyor!)
2.  **Farklı Bir Sinyal Deneyin:** Eğer bir metin kutusu olsaydı, `textChanged` sinyalini kullanabilirdik. Bir `QLineEdit` ekleyip `self.input.textChanged.connect(self.metni_degistir)` yazarsanız, her harf yazdığınızda fonksiyon tetiklenir.

## 🧠 Kendini Test Et

1. **Soru:** Bir sinyali birden fazla slota bağlayabilir miyiz?
2. **Soru:** `.connect()` metodunda neden fonksiyon isminin yanına parantez `()` yazmıyoruz?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Evet! Bir butona tıkladığınızda hem bir etiketi değiştirebilir, hem de aynı anda bir dosyayı kaydeden başka bir fonksiyonu tetikleyebilirsiniz.
2. **Cevap:** Çünkü parantez koymak, fonksiyonu "hemen şimdi çalıştır" demektir. Biz ise fonksiyonun kendisini (referansını) bağlantıya veriyoruz ki, olay (tıklama) gerçekleştiğinde Qt o fonksiyonu bizim yerimize çalıştırsın.

</details>

## 🚀 Kendin Dene

**Görev:**
Mevcut kodunuza ikinci bir buton ekleyin.
1.  Butonun metni **"Sıfırla"** olsun.
2.  Bu butona tıklandığında, etiketteki metin tekrar eski haline (**"Henüz bir şeye basılmadı..."**) dönsün.
3.  İpucu: Yeni bir slot metodu (`self.metni_sifirla`) oluşturmalı ve bunu yeni butonun `clicked` sinyaline bağlamalısınız.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: Signals & Slots](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#signals-and-slots)

## 📝 Kısa Özet

- **Olay Güdümlü Programlama:** Programın akışının kullanıcı eylemlerine (sinyallere) bağlı olmasıdır.
- **Sinyal (Signal):** Bir nesnenin meydana gelen bir olayı bildirmesidir (Örn: `clicked`).
- **Slot (Yuva):** Sinyal geldiğinde çalıştırılan hedef fonksiyondur.
- **Bağlantı (Connection):** Sinyal ile slotu birbirine bağlayan köprüdür (`.connect()`).

---
[⬅️ Önceki Ders](02-Oop-And-Structure.md) | [Sonraki Ders ➡️](04-Layout-Management.md)
