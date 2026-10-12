# Pencere Anatomisi ve Diyaloglar

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Bir ana pencerenin temel bileşenlerini (`QMenuBar`, `QToolBar`, `QStatusBar`) kullanabilmek.
- Uygulama içinde kullanıcıya bilgi, uyarı veya hata iletmek için `QMessageBox` kullanabilmek.
- Dosya seçme, dosya açma ve dosya kaydetme işlemleri için `QFileDialog` entegrasyonunu yapabilmek.
- Modalllık (Modal vs Modeless) kavramını anlayarak kullanıcı etkileşimini yönetebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Şu ana kadar oluşturduğumuz uygulamalar, sadece birer "kutu" gibiydi. Kullanıcıya bir işlem yapması gerektiğini söyleyen bir menü yoktu, bir dosya seçmesini isteyen bir pencere açılmıyordu veya bir hata oluştuğunda kullanıcıya "Dikkat! Dosya bulunamadı" diyen bir uyarı kutusu gelmiyordu.

Bir yazılımın "tam bir uygulama" gibi hissettirmesi için, kullanıcının işletim sistemiyle ve uygulama akışıyla etkileşime girmesini sağlayan bu yapısal bileşenlere ve diyaloglara ihtiyacı vardır.

## 🔍 Birlikte Keşfedelim

Bir restoran deneyimini düşünelim:

1.  **Menü (QMenuBar):** Restoranın girişindeki yemek listesidir. Hangi kategoride (Başlangıçlar, Ana Yemekler, İçecekler) ne olduğunu gösterir.
2.  **Araç Çubuğu (QToolBar):** Garsonun belindeki hızlı erişim araçları gibidir (peçete, çatal, bıçak). En çok kullanılan işlere hızlıca ulaşmanızı sağlar.
3.  **Durum Çubuğu (QStatusBar):** Garsonun size "Siparişiniz hazırlanıyor..." veya "Hesap hazır" demesi gibidir. Uygulamanın o anki durumu hakkında sessiz ama önemli bilgi verir.
4.  **Diyalog (QMessageBox):** Garsonun yanınıza gelip "Bir hata oldu, siparişiniz bitmedi" veya "Tatlı ister misiniz?" (Evet/Hayır) diye sormasıdır.

> [!IMPORTANT]
> **Modal Diyalog:** Kullanıcı diyaloğu kapatmadan ana pencereyle etkileşime giremez (Örn: "Kaydetmek istiyor musunuz?" sorusu).
> **Modeless Diyalog:** Kullanıcı diyaloğu açık tutarken arka plandaki ana pencerede çalışmaya devam edebilir (Örn: "Bul ve Değiştir" penceresi).

## 🛠️ Uygulama

Bu derste, bir metin editörü taslağı oluşturacağız. Menüden dosya seçeceğiz ve seçilen dosyanın içeriğini ekranda göstereceğiz.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QTextEdit, 
                             QFileDialog, QMessageBox, QStatusBar)
from PySide6.QtGui import QAction

class MetinEditoru(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Basit Metin Düzenleyici")
        self.resize(600, 400)

        # 1. Merkezi Alan (Metin Yazma Alanı)
        self.text_edit = QTextEdit()
        self.setCentralWidget(self.text_edit)

        # 2. Durum Çubuğu (StatusBar)
        self.setStatusBar(QStatusBar(self))
        self.statusBar().showMessage("Hazır")

        # 3. Menü Çubuğu (MenuBar) Oluşturma
        menubar = self.menuBar()
        dosya_menusu = menubar.addMenu("&Dosya") # '&' işareti Alt+F kısayolu sağlar

        # 4. Aksiyonlar (QAction) Tanımlama
        ac_aksiyonu = QAction("Dosya Aç", self)
        ac_aksiyonu.setShortcut("Ctrl+O")
        ac_aksiyonu.triggered.connect(self.dosya_ac)

        kaydet_aksiyonu = QAction("Kaydet", self)
        kaydet_aksiyonu.setShortcut("Ctrl+S")
        kaydet_aksiyonu.triggered.connect(self.dosya_kaydet)

        cikis_aksiyonu = QAction("Çıkış", self)
        cikis_aksiyonu.triggered.connect(self.close)

        # Aksiyonları menüye ekleme
        dosya_menusu.addAction(ac_aksiyonu)
        dosya_menusu.addAction(kaydet_aksiyonu)
        dosya_menusu.addSeparator() # Araya çizgi çeker
        dosya_menusu.addAction(cikis_aksiyonu)

    def dosya_ac(self):
        # 5. Dosya Seçme Diyaloğu (QFileDialog)
        dosya_yolu, _ = QFileDialog.getOpenFileName(
            self, "Dosya Seç", "", "Metin Dosyaları (*.txt);;Tüm Dosyalar (*)"
        )

        if dosya_yolu:
            try:
                with open(dosya_yolu, 'r', encoding='utf-8') as f:
                    icerik = f.read()
                    self.text_edit.setText(icerik)
                    self.statusBar().showMessage(f"Açıldı: {dosya_yolu}", 3000)
            except Exception as e:
                # 6. Hata Mesaj Kutusu (QMessageBox)
                QMessageBox.critical(self, "Hata", f"Dosya okunurken bir hata oluştu:\n{str(e)}")
        else:
            self.statusBar().showMessage("Dosya seçimi iptal edildi.")

    def dosya_kaydet(self):
        dosya_yolu, _ = QFileDialog.getSaveFileName(self, "Dosya Kaydet", "", "Metin Dosyaları (*.txt)")
        
        if dosya_yolu:
            try:
                with open(dosya_yolu, 'w', encoding='utf-8') as f:
                    f.write(self.text_edit.toPlainText())
                QMessageBox.information(self, "Başarılı", "Dosya başarıyla kaydedildi!")
                self.statusBar().showMessage("Kaydedildi.", 3000)
            except Exception as e:
                QMessageBox.critical(self, "Hata", f"Kaydedilemedi: {str(e)}")

def main():
    app = QApplication(sys.argv)
    pencere = MetinEditoru()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Kritik Noktaları:**
- **`QAction`**: Menüdeki bir öğeye tıklandığında ne olacağını belirleyen "komut" nesnesidir.
- **`getOpenFileName`**: Kullanıcıya dosya seçtiren standart Windows/Linux penceresini açar. Geriye `(dosya_yolu, seçilen_filtre)` şeklinde bir tuple döner.
- **`QMessageBox.critical`**: Kullanıcıya çok önemli bir hata olduğunu belirtmek için kırmızı bir ikon ve "Tamam" butonu içeren bir pencere açar.
- **`statusBar().showMessage(mesaj, süre)`**: Alt köşede geçici bir bilgi mesajı gösterir. `süre` milisaniye cinsindendir.

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Hatalı Dosya Deneyi:** Bir metin dosyası açmaya çalışın ama dosya okuma yetkiniz olmayan bir klasörü seçin. `QMessageBox.critical` nasıl bir tepki veriyor?
2.  **Kısayol Testi:** `Ctrl+O` tuşlarına basın. Menüye gitmeden dosya seçme penceresi açılıyor mu?
3.  **StatusBar Gözlemi:** Dosya açtığınızda alt taraftaki mesajın birkaç saniye sonra kaybolup kaybolmadığını kontrol edin.

## 🧠 Kendini Test Et

1. **Soru:** `QFileDialog.getOpenFileName` metodundan dönen ikinci değer (indeks 1) neyi temsil eder?
2. **Soru:** Bir uygulamanın "çökmesini" engellemek için dosya okuma işlemlerinde neden `try-except` kullanmalıyız?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Kullanıcının seçtiği dosya filtreleme kriterini (örneğin `"Metin Dosyaları (*.txt)"`) temsil eder.
2. **Cevap:** Dosyanın silinmiş olması, izin eksikliği veya dosyanın bozuk olması gibi durumlar Python'da bir `Exception` fırlatır. Eğer bunu yakalamazsak program doğrudan kapanır (crash). `try-except` sayesinde hatayı yakalayıp kullanıcıya nazik bir `QMessageBox` ile gösterebiliriz.

</details>

## 🚀 Kendin Dene

**Görev:**
Mevcut uygulamaya bir **"Temizle"** aksiyonu ekleyin.
1.  Menüye "Düzen" adında yeni bir menü ekleyin.
2.  Bu menünün altına "Tümünü Temizle" adında bir aksiyon ekleyin.
3.  Bu aksiyona tıklandığında `text_edit` içindeki tüm metnin silinmesini sağlayın.
4. Temizlikten sonra `statusBar` üzerinde "Metin temizlendi" mesajını gösterin.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: QMenuBar](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QMenuBar.html)
- [PySide6 Documentation: QFileDialog](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QFileDialog.html)

---
[⬅️ Önceki Ders](Topics/06-Form-Widgets.md) | [Sonraki Ders ➡️](Topics/08-Qss-And-Styling.md)
