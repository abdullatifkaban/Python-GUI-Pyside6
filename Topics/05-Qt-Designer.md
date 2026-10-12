# Qt Designer ile Tasarım

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- **Qt Designer** görsel tasarım aracını kullanarak sürükle-bırak yöntemiyle arayüz tasarlayabilmek.
- `.ui` (XML tabanlı) dosya formatının yapısını ve önemini kavramak.
- Tasarlanan `.ui` dosyalarını Python koduna iki farklı yöntemle (dinamik yükleme ve dönüştürme) entegre edebilmek.
- Tasarım dosyasındaki bileşenlere (widget) Python tarafında erişebilmek.
- Görsel tasarım ile iş mantığını (business logic) birbirinden ayırabilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Bir arayüzü tamamen kod yazarak oluşturmak (hard-coding), özellikle karmaşık ve çok fazla bileşen içeren uygulamalarda çok zaman alır ve hata yapma olasılığını artırır. 

**Qt Designer**, bir "tuval" üzerinde butonları, etiketleri ve kutuları sürükleyip bırakarak, koordinatlarla veya layoutlarla uğraşmadan hızlıca arayüz oluşturmanızı sağlayan görsel bir araçtır. Bu yöntem sayesinde:
1.  **Hız:** Tasarım süreci çok daha hızlıdır.
2.  **Ayrıştırma (Separation of Concerns):** Arayüz tasarımı (görsellik) ile Python kodunuz (mantık) birbirinden ayrı tutulur. Tasarımı değiştirmek için Python kodunu bozmanıza gerek kalmaz.
3.  **Görsel Geri Bildirim:** Kod yazmadan önce uygulamanın nasıl görüneceğini anında görebilirsiniz.

## 🔍 Birlikte Keşfedelim

Bir web sitesi tasarladığınızı düşünün. Bir web tasarımcısı, HTML kodlarını elle yazmak yerine, hazır kutuları ve resimleri bir sayfa üzerinde yerleştirerek tasarımı oluşturur. Ardından bu tasarım, arka plandaki kodlarla birleştirilir.

Qt Designer tam olarak bunu yapar. Siz bir "tasarımcı" gibi davranarak görsel yapıyı kurarsınız, biz ise bir "yazılımcı" gibi o yapıyı canlandıran kodları yazarız.

> [!TIP]
> Profesyonel projelerde genellikle arayüz tasarımı (UI) ve uygulama mantığı (Logic) farklı dosyalarda tutulur. Bu, bir tasarımcının kod yazmadan arayüzü güncellemesine olanak tanır.

## 🛠️ Uygulama

Bu derste, Qt Designer ile hazırladığımız bir tasarımı Python içinde **dinamik olarak yükleme** yöntemini öğreneceğiz.

### Adım 1: Qt Designer ile İlk Tasarımınızı Yapın

1.  Terminalinize `designer` yazarak veya IDE'nizin araçları arasından Qt Designer'ı bularak programı açın.
2.  **Main Window** şablonunu seçin ve "Create" butonuna basın.
3.  Sol taraftaki **Widget Box** panelinden bir `Push Button` sürükleyip pencerenin içine bırakın.
4.  Butona tıklayın ve sağ taraftaki **Property Editor** panelinden `objectName` özelliğini bulun. Burayı `btn_selamla` olarak değiştirin (Bu isim, Python'dan bu butona ulaşmamızı sağlayacak olan kimliktir!).
5.  Bir de `Label` sürükleyip bırakın ve onun `objectName` değerini `lbl_mesaj` yapın.
6.  Dosyayı `tasarim.ui` adıyla projenizin bulunduğu klasöre kaydedin.

### Adım 2: Dinamik Yükleme (QUiLoader) ile Python Entegrasyonu

Tasarım dosyasını (`.ui`), Python kodumuzun içine "çalışma anında" (runtime) yükleyeceğiz. Bu yöntem, tasarımda yaptığınız değişikliklerin kodu değiştirmeden hemen yansımasını sağlar.

```python
import sys
from PySide6.QtWidgets import QApplication, QMainWindow
from PySide6.QtUiTools import QUiLoader
from PySide6.QtCore import QFile

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        
        # 1. Tasarım dosyasını yüklemek için QUiLoader nesnesi oluşturma
        self.loader = QUiLoader()
        
        # 2. .ui dosyasını açma
        self.ui_dosyasi = QFile("tasarim.ui")
        self.ui_dosyasi.open(QFile.ReadOnly)
        
        # 3. Tasarımı yükleme (Bu işlem bize bir widget nesnesi döndürür)
        self.ui = self.loader.load(self.ui_dosyasi, self)
        self.ui_dosyasi.close()

        # 4. Pencereyi ana pencere olarak ayarlama
        self.setCentralWidget(self.ui)
        self.setWindowTitle("Qt Designer Entegrasyonu")

        # 5. Tasarımdaki bileşenlere (widget) erişme
        # findChild metodu, nesne ismine (objectName) göre bileşeni bulur.
        self.buton = self.ui.findChild(QPushButton, "btn_selamla")
        self.etiket = self.ui.findChild(QLabel, "lbl_mesaj")

        # 6. Sinyal-Slot bağlantısı
        if self.buton:
            self.buton.clicked.connect(self.selamla)

    def selamla(self):
        if self.etiket:
            self.etiket.setText("👋 Tasarım başarıyla yüklendi!")

def main():
    app = QApplication(sys.argv)
    pencere = AnaPencere()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Önemli Detaylar:**
- **`QUiLoader`**: `.ui` dosyasını okuyup onu çalışan bir nesneye dönüştüren araçtır.
- **`findChild(Tip, "isim")`**: Tasarımda verdiğiniz `objectName` değerini kullanarak, Python tarafında o bileşene bir değişken ismiyle ulaşmanızı sağlar.
- **`setCentralWidget(self.ui)`**: Yüklenen tasarımın tüm pencereyi kaplamasını sağlar.

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Tasarımı Güncelleyin:** Qt Designer'ı tekrar açın. Butonun metnini değiştirin veya butonun rengini/boyutunu ayarlayın. Dosyayı kaydedin ve Python kodunu **tekrar çalıştırmadan** (sadece kapatıp açarak) sonucu görün. Değişiklikler anında yansıdı mı?
2.  **İsim Değişikliği:** Qt Designer'da butonun `objectName` değerini `btn_yeni` yapın. Kodunuzu çalıştırın. Program hata veriyor mu? (İpucu: `findChild` artık eski ismi bulamadığı için `None` dönecektir!)

## 🧠 Kendini Test Et

1. **Soru:** `QUiLoader` yöntemi ile `.ui` dosyasını Python'a yüklemenin en büyük avantajı nedir?
2. **Soru:** `findChild` metodunda neden ilk parametre olarak `QPushButton` gibi bir tip belirtiyoruz?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Dinamiktir. Tasarımda yapılan değişiklikler (renk, boyut, konum vb.) Python kodunda hiçbir satırı değiştirmeye gerek kalmadan, uygulama bir sonraki çalıştırıldığında doğrudan uygulanır.
2. **Cevap:** Bu, arama işlemini daha güvenli ve hızlı hale getirir. Python'a "Bu isimdeki nesneyi bul ama sadece bir buton ise getir" demiş olursunuz.

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Qt Designer ile yeni bir tasarım yapın.
2.  İçine bir `QLineEdit` (metin girişi) ve bir `QPushButton` ekleyin.
3.  `QLineEdit`'in adını `txt_giris`, butonun adını `btn_yazdir` yapın.
4.  Python kodunuzda bu bileşenlere erişin.
5.  Butona tıklandığında, `QLineEdit` içine yazılan metni bir `QLabel` içinde gösteren uygulamayı tamamlayın.

## 📚 Daha Fazlasını Öğrenmek

- [Qt Designer Official Documentation](https://doc.qt.io/qtforpython-6/PySide6/designer/index.html)
- [pyside6-uic: .ui dosyalarını .py dosyasına dönüştürme](https://doc.qt.io/qtforpython-6/PySide6/scripts/pyside6-uic.html) (Alternatif yöntem)

---
[⬅️ Önceki Ders](Topics/04-Layout-Management.md) | [Sonraki Ders ➡️](Topics/06-Form-Widgets.md)
