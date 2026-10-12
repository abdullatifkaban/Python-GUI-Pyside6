# OOP ve Temel Yapı

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Grafiksel arayüz geliştirmede Nesne Yönelimli Programlama (OOP) mantığının neden kritik olduğunu açıklayabilmek.
- `QWidget` ve `QMainWindow` sınıfları arasındaki yapısal farkları ayırt edebilmek.
- Python'da sınıf (class) yapısı kullanarak arayüz bileşenlerini bir bütün halinde organize edebilmek.
- Kalıtım (Inheritance) ve `super().__init__()` kavramlarını bir arayüz projesinde uygulayabilmek.
- Sınıf içi değişkenleri (`self.`) kullanarak widget'lara farklı metotlardan erişebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Geçen hafta yazdığımız kod, basit bir pencere açmak için yeterliydi. Ancak gerçek bir uygulama; onlarca buton, yüzlerce metin kutusu, veritabanı bağlantıları ve karmaşık iş mantığı içerir. 

Eğer her şeyi tek bir dosya içinde, değişkenleri tek tek tanımlayarak yazmaya kalkarsanız, kodunuz bir süre sonra yönetilemez bir "spagetti koda" dönüşür. **Nesne Yönelimli Programlama (OOP)**, arayüzünüzü ve uygulamanızın özelliklerini mantıklı "nesneler" (sınıflar) altında toplamanıza olanak tanır. Bu sayede kodunuz düzenli, tekrar kullanılabilir ve geliştirilebilir olur.

## 🔍 Birlikte Keşfedelim

Bir mimar olduğunuzu hayal edin. Bir ev inşa ederken sadece tuğlaları üst üste dizmezsiniz; önce bir **plan (blueprint)** hazırlarsınız. Bu plan; odaların nerede olacağını, kapıların nereye açılacağını ve evin genel yapısını belirler.

- **Plan (Class/Sınıf):** Evin nasıl olacağını tarif eden çizimdir. Henüz bir ev değildir, sadece bir tariftir.
- **Ev (Instance/Nesne):** Bu plana bakılarak inşa edilmiş, içinde yaşayabileceğiniz gerçek yapıdır.

GUI dünyasında da durum aynıdır. `AnaPencere` adında bir plan (sınıf) hazırlarız, sonra bu plandan gerçek bir pencere (nesne) üretiriz.

> [!IMPORTANT]
> Eğer bir arayüzü sınıf yapısı kullanmadan yazarsanız, bir butona basıldığında o butonun içindeki veriye başka bir fonksiyonun ulaşması çok zorlaşır. Sınıf yapısı, tüm bu bileşenleri tek bir "çatı" altında toplar.

## 🛠️ Uygulama

Bu derste, bir önceki haftaki kodumuzu daha profesyonel bir yapıya, yani bir sınıf yapısına dönüştüreceğiz.

### Adım 1: Sınıf Yapısını Kurma ve Kalıtım

Öncelikle `QMainWindow` sınıfından miras alan bir `AnaPencere` sınıfı tanımlayacağız. `super().__init__()` komutu, miras aldığımız `QMainWindow` sınıfının tüm özelliklerini bizim sınıfımıza aktarır.

### Adım 2: Yardımcı Metot (`initUI`) Kullanımı

Tüm arayüz kodlarını doğrudan `__init__` içine yazmak yerine, düzenli olması için `initUI` adında bir yardımcı metot oluşturacağız.

### Adım 3: Widget'ları Sınıf İçinde Tanımlama

Widget'larımızı (etiketler, butonlar vb.) `self.etiket` şeklinde tanımlayacağız. Buradaki `self`, bu nesnenin "kendisine" ait olduğunu ve sınıfın her yerinden erişilebileceğini belirtir.

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow, QLabel, QWidget

class AnaPencere(QMainWindow):
    def __init__(self):
        # 1. Miras alınan sınıfın (QMainWindow) başlatılması
        super().__init__()
        
        # 2. Arayüz kurulum metodu çağrılıyor
        self.initUI()

    def initUI(self):
        # Pencere ayarları
        self.setWindowTitle("OOP ile İlk Pencere")
        self.resize(400, 300)

        # QMainWindow için merkezi bir widget (merkezi alan) gereklidir.
        # Tüm bileşenler bu merkezi widget üzerine yerleştirilir.
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)

        # 3. Bir etiket (QLabel) oluşturma ve sınıf değişkeni olarak tanımlama
        # self.etiket sayesinde bu değişkene sınıfın her yerinden ulaşabiliriz.
        self.etiket = QLabel("OOP Dünyasına Hoş Geldiniz!", self.merkezi_alan)
        
        # Etiketi pencerenin ortasına yakın bir yere konumlandıralım
        self.etiket.move(100, 130)

def main():
    app = QApplication(sys.argv)
    
    # Sınıftan bir nesne (instance) oluşturuyoruz
    pencere = AnaPencere()
    pencere.show()
    
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Kritik Noktaları:**
- **`super().__init__()`**: "Ben bir `QMainWindow`'ım ama kendime has özelliklerim de olacak, önce temel özelliklerimi yükle" demektir.
- **`self.merkezi_alan`**: `QMainWindow` yapısında, butonların ve etiketlerin duracağı bir "ana platform" (central widget) belirlemek zorunludur.
- **`self.etiket`**: Eğer sadece `etiket = QLabel(...)` yazsaydık, bu değişken sadece `initUI` metodu içinde yaşardı. `self.` kullanarak onu tüm sınıfın bir parçası yaptık.

## 🧪 Deneyelim ve Gözlemleyelim

Kodunuz çalıştıktan sonra şu değişiklikleri yapın:

1.  `self.etiket.move(100, 130)` satırındaki koordinatları değiştirin (örneğin `50, 50`). Etiket nasıl hareket ediyor?
2.  `self.setWindowTitle(...)` kısmını değiştirin.
3.  `initUI` metodu içine `print("Arayüz hazırlandı!")` satırını ekleyin. Programı çalıştırdığınızda terminalde ne görüyorsunuz?

## 🧠 Kendini Test Et

1. **Soru:** `QMainWindow` kullanırken neden `setCentralWidget` metodunu çağırmalıyız?
2. **Soru:** `self` anahtar kelimesi ne işe yarar?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** `QMainWindow` karmaşık bir yapıya sahiptir (menüler, araç çubukları vb.). Uygulamanın ana içeriğinin nerede duracağını belirtmek için merkezi bir bileşen (widget) tanımlamak zorunludur.
2. **Cevap:** `self`, oluşturulan nesnenin kendisine referans verir. Sınıf içindeki değişkenlerin ve metotların nesneye özel olarak saklanmasını ve sınıfın her yerinden erişilmesini sağlar.

</details>

## 🚀 Kendin Dene

**Görev:**
`AnaPencere` sınıfı içinde, `initUI` metoduna ikinci bir `QLabel` daha ekleyin. 
- Bu yeni etiketin metni **"Bu ikinci etikettir."** olsun.
- Bu etiketi, ilk etiketin hemen altına (örneğin `100, 170` koordinatlarına) yerleştirin.
- Programı çalıştırdığınızda iki etiketin de pencerede göründüğünden emin olun.

## 📚 Daha Fazlasını Öğrenmek

- [Python Docs: Classes](https://docs.python.org/3/tutorial/classes.html)
- [PySide6 Documentation: QMainWindow](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QMainWindow.html)

## 📝 Kısa Özet

- **OOP**, karmaşık arayüzleri yönetilebilir parçalara böler.
- **Kalıtım (Inheritance)**, mevcut sınıfların (örneğin `QMainWindow`) yeteneklerini kullanarak yeni sınıflar türetmemizi sağlar.
- **`self`**, sınıf içindeki verilerin (widget'lar gibi) kalıcı olmasını ve her yerden erişilmesini sağlar.
- **`QMainWindow`**, menüler ve araç çubukları gibi özellikler sunan hazır bir pencere yapısıdır.

---
[⬅️ Önceki Ders](Topics/01-Introduction-And-Setup.md) | [Sonraki Ders ➡️](Topics/03-Signals-And-Slots.md)
