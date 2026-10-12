# Arka Plan İşlemleri (QThread)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- **Arayüz Kilitlenmesi (UI Freeze)** nedir ve neden gerçekleştiğini açıklayabilmek.
- Ana iş parçacığı (GUI Thread) ve arka plan iş parçacığı (Worker Thread) arasındaki farkı kavrayabilmek.
- `QThread` ve `QObject` (Worker mantığı) kullanarak uzun süren işlemleri arka planda çalıştırabilmek.
- PySide6 sinyallerini (`pyqtSignal`) kullanarak arka plandan arayüze veri ve ilerleme durumu aktarabilmek.
- `QProgressBar` ile ilerleme çubuğunu canlı güncelleyebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Bir uygulama içinde dosya indirme, büyük bir veri setini işleme veya karmaşık bir hesaplama yapmak gibi işlemler zaman alabilir. Eğer bu işlemleri **ana arayüz iş parçacığında (GUI Thread)** çalıştırırsanız, uygulamanız kullanıcı girişlerini (fare hareketleri, tuş vuruşları) işleyemez ve "donmuş" gibi görünür. Kullanıcı bu durumda uygulamanın çöktüğünü sanabilir ve pencereyi kapatmaya çalışabilir.

Bu deneyimi **Arayüz Kilitlenmesi (UI Freeze)** denir ve kullanıcı deneyimi için büyük bir sorundur. İşte bu sorunu çözmek için **çoklu iş parçacığı (multithreading)** kullanırız. Ana iş parçacığı sadece arayüzü güncellemekle, zaman alan işler ayrı bir arka plan iş parçacığına (Worker Thread) bırakılır. Bu sayede arayüz her zaman yanıt vermeye (responsive) devam eder.

> [!WARNING]
> PySide6 (ve Qt genellikle) **kurallar**: **Sadece ana iş parçacığı (GUI Thread)** arayüz widget'larını değiştirebilir veya ona erişebilir. Arka plan iş parçacığını doğrudan bir butonun metnini değiştirmeye veya bir pencereyi kapatmaya çalıştırmayın! Bunun yerine, arka plandan **sinyal** gönderip ana iş parçacığında bu sinyali yakalayıp gerekli güncellemeyi yapmalısınız.

## 🔍 Birlikte Keşfedelim

Bir restoranın mutfağını ve salonu hayal edin:

1.  **Ana İş Parçacığı (GUI Thread):** Salondaki garson gibi. Müşteriyi karşılar, sipariş alır, yemeği sunar ve masaları temizler. Sürekli hareket halinde ve kullanıcı ile etkileşimde olur.
2.  **Arka Plan İş Parçacığı (Worker Thread):** Mutfağa çalışan aşçılar gibi. Sebzeleri soyur, etleri pişirir ve sosları hazırlar. Bu işler zaman alır ve garsonla doğrudan etkileşimde değildir. İşinin bir kısmı tamamlandığında (örneğin hafif bir salata hazır) garsona bildirim yapar: "Salata hazır!" Garson bu bilgi alıp sunumu yapar.
3.  **Sinyal (Signal):** Mutfağın garsona verdiği bildirimdir (örneğin bir fısıltı veya bir çağrı).
4.  **Slot (Yuva):** Garsonun bu bildirim alıp yapacağı eylemdir (örneğin salatayi plaka üzerine koyma).

Eğer aşçı (arka plan) doğrudan garsonun görevini yapmaya (örneğin menüyü alma) kalkarsa, karmaşaya yol açar ve mutfak çalışmaz. Aynı şekilde, bir PySide6 uygulamasında arka plan iş parçacığı doğrudan bir butonu değiştirmeye kalkarsa uygulama çökebilir veya öngörülemez şekilde davranabilir.

## 🛠️ Uygulama

Bu derste, bir sayacı 0'dan 100'e kadar sayıp her adımda ilerleme durumunu bildiren ve bu bilgiyi arayüzde bir `QProgressBar` ile gösteren bir uygulama oluşturacağız.

### Adım 1: Worker Sınıfı Tanımlama

İlk olarak, uzun süren işlemi yapacak olan `Worker` sınıfını tanımlayacağız. Bu sınıf `QObject` dan miras alır ve `QThread` içinde çalışacaktır.

### Adım 2: Sinyaller Tanımlama

Worker sınıfı içinde, ilerleme durumunu (`progress`) ve işin bittiğini (`finished`) bildiren iki sinyal tanımlayacağız.

### Adım 3: QThread Kurulumu ve Bağlantılar

Ana pencerede bir `QThread` örneği oluşturacağız, `Worker` nesnesini bu iş parçacığına taşıyacağız ve sinyallerimizi uygun slotlara (fonksiyonlara) bağlayacağız.

```python
import sys
import time
from PySide6.QtCore import QObject, QThread, Signal
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QVBoxLayout, QPushButton, QProgressBar, QLabel)

# --- 1. WORKER SINIFI ---
class Worker(QObject):
    # 2. SİNYAL TANIMLAMALARI
    # 'progress' sinyali, tam sayı bir değer (yüzde) taşır
    progress = Signal(int)
    # 'finished' sinyali, iş tamamlandığında tetiklenir
    finished = Signal()

    def __init__(self):
        super().__init__()

    # 3. UZUN SÜREN İŞLEM
    def run(self):
        """0'dan 100'e kadar sayar, her adımda 100 ms bekler."""
        for i in range(101):
            # İşlem gerçekten burada yapılır (örneğin veri indirme, işleme)
            time.sleep(0.1) # Gerçek bir işlem yerine bekleme simülasyonu
            # İlerlemeyi ana iş parçacığına bildir
            self.progress.emit(i)
        # İş bitti
        self.finished.emit()

# --- ANA PENCERE ---
class ArkaPlanIslemUygulamasi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("QThread ile Arka Plan İşlemleri")
        self.resize(300, 200)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        self.etiket = QLabel("İşlem Başlamadı")
        self.layout.addWidget(self.etiket)

        self.progress_bar = QProgressBar()
        self.progress_bar.setRange(0, 100)
        self.layout.addWidget(self.progress_bar)

        self.btn_baslat = QPushButton("İşlemi Başlat")
        self.btn_baslat.clicked.connect(self.islemi_baslat)
        self.layout.addWidget(self.btn_baslat)

        self.btn_iptal = QPushButton("İşlemi İptal Et")
        self.btn_iptal.clicked.connect(self.islemi_iptal)
        self.btn_iptal.setEnabled(False) # Başlangıçta pasif
        self.layout.addWidget(self.btn_iptal)

        # Thread ve Worker nesnelerini None ile başlatıyoruz
        self.thread = None
        self.worker = None

    def islemi_baslat(self):
        """İş parçacığını ve worker'i oluşturup başlatır."""
        # 1. QThread nesnesi oluştur
        self.thread = QThread()
        # 2. Worker nesnesi oluştur
        self.worker = Worker()
        # 3. Worker'i thread'e taşımama
        self.worker.moveToThread(self.thread)

        # 4. SİNYAL-SLOT BAĞLANTILARI
        # Thread başladığında worker.run metodu çalıştırılsın
        self.thread.started.connect(self.worker.run)
        # Worker'den gelen ilerleme sinyali, progress_bar'ı güncellesin
        self.worker.progress.connect(self.guncelle_progress)
        # Worker'den iş bitti sinyali alındığında
        self.worker.finished.connect(self.islemi_bitti)
        # İş bittiğinde thread'i güvenli şekilde kapat
        self.worker.finished.connect(self.thread.quit)
        # Thread bittiğinde kaynakları serbest bırak (memory leak önleme)
        self.thread.finished.connect(self.worker.deleteLater)
        self.thread.finished.connect(self.thread.deleteLater)

        # 5. ARAYÜZ DURUMUNU GÜNCELLE
        self.btn_baslat.setEnabled(False)
        self.btn_iptal.setEnabled(True)
        self.etiket.setText("İşlem Devam Ediyor...")
        self.progress_bar.setValue(0)

        # 6. THREAD'I BAŞLAT
        self.thread.start()

    def guncelle_progress(self, deger):
        """Worker'den gelen progres sinyalini işler."""
        self.progress_bar.setValue(deger)
        self.etiket.setText(f"İşlem % {deger} tamamlandı")

    def islemi_bitti(self):
        """Worker'in finished sinyali alındığında çalışır."""
        self.etiket.setText("🎉 İşlem Tamamlandı!")
        self.btn_baslat.setEnabled(True)
        self.btn_iptal.setEnabled(False)

    def islemi_iptal(self):
        """Kullanıcı iptal butonuna bastığında çalışır."""
        if self.thread and self.thread.isRunning():
            # Thread'i sonlandırmak için quit() yeterli olabilir,
            # ancak worker içinde bir bayrak kontrolü daha güvenlidir.
            # Bu örnekte basitçe quit() kullanıyoruz.
            self.thread.quit()
            self.thread.wait() # Thread'in gerçekten bitmesini bekle
            self.etiket.setText("⏹️ İşlem Kullanıcı Tarafından İptal Edildi")
            self.btn_baslat.setEnabled(True)
            self.btn_iptal.setEnabled(False)

def main():
    app = QApplication(sys.argv)
    pencere = ArkaPlanIslemUygulamasi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Kritik Noktaları:**
- **`Worker.moveToThread(self.thread)`**: Worker nesnesini belirtilen iş parçacığına taşır. Bunun ardından Worker'in tüm metotları o iş parçacığında çalışır.
- **`self.thread.started.connect(self.worker.run)`**: Thread başladığında `Worker.run()` metodu otomatik olarak çağrılır. Dikkat: `run()` metodu **parantezsiz** verilir!
- **`self.worker.progress.connect(self.guncelle_progress)`**: Worker'den gelen `progress` sinyali, ana iş parçacığındaki `guncelle_progress` fonksiyonuna bağlanır. Bu sayede güvenli bir şekilde ProgressBar güncellenir.
- **`self.worker.finished.connect(self.thread.quit)`**: İş bittiğinde thread'i durdurarak kaynakları serbest bırakırız.
- **`self.thread.finished.connect(self.worker.deleteLater)`**: Thread tamamlandığında Worker nesnesini bellekten silerek memory leak önleriz.
- **İptal İşlemi**: `thread.quit()` thread'in etkinlik döngüsünü (event loop) durdurur. Daha güvenli bir yöntem için Worker içinde bir `self._is_running` bayrağı kullanılabilir, ancak bu örnekte basit tutuyoruz.

## 🧪 Deneyelim ve Gözlemleyelim

1.  **İşlem Başlatma:** "İşlemi Başlat" butonuna basın. ProgressBar dolmaya başlayıp etiket güncelleniyor mu? Butonlar durumunda geändertı mi? (Başlat pasif, İptal aktif olmalı)
2.  **İptal Etme:** İşlem sırasında "İşlemi İptal Et" butonuna basın. İş durduruluyor mu ve etiket güncelliyor mu?
3.  **Arayüz Tepkililiği:** İşlem sırasında pencereyi taşıyabilir, yeniden boyutlandırabilir veya diğer butonlarla etkileşime geçebilir misiniz? (Evet, olmalı! UI donmamalı)
4.  **Çoklu Başlatma:** İşlem sırasında tekrar "İşlemi Başlat" butonuna basmayı deneyin. Buton pasif olduğu için ikinci bir thread başlamamalıdır.

## 🧠 Kendini Test Et

1. **Soru:** `QThread` içinde doğrudan `self.progress_bar.setValue(50)` gibi bir satır yazarsak ne olur?
2. **Soru:** Worker sınıfında `__init__` metodunda `self.progress = Signal(int)` tanımlamak yerine, `run` metodunun içinde tanımlamasaydık ne olur?
3. **Soru:** `self.thread.finished.connect(self.worker.deleteLater)` satırının anlamı nedir?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Uygulama muhtemelen çökecek (crash) veya öngörülemez şekilde davranacak çünkü **sadece ana iş parçacığı** GUI nesnelerine erişebilir. Arka plan iş parçacığından doğrudan erişim güvensizdir.
2. **Cevap:** Her `Worker.run()` çağrısında yeni bir sinyal nesnesi oluşturulurdu, bu da bellek sızıntısı ve tutarsız davranıma yol açardı. Sinyaller sınıf seviyesinde tanımlanmalıdır.
3. **Cevap:** Thread'in çalışması bittiğinde (quit() tarafından), Worker nesnesinin bellekten güvenli bir şekilde silinmesini sağlar. Bu, Python'ın çöp toplayıcısının nesneyi toplamamasını önler ve memory leak'i önler.

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Mevcut uygulamaya bir **"Durdur ve Sıfırla"** butonu ekleyin.
2.  Bu buton, işlem sırasında çalışıp, işi durdurup progress bar'ı sıfırlamalı ve etiketi "İşlem sıfırlandı" olarak güncellemeli.
3.  İpucu: Worker sınıfına bayrak bazlı bir durdurma mekanizması ekleyin. `run` metodu içinde bir `if not self._is_running: break` kontrolü yapın ve `stop` metodu ile bu bayrağı `False` yapın.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: QThread](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QThread.html)
- [PySide6 Documentation: QObject & Signals & Slots](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#signals-and-slots)
- [Threading in Qt (Official)](https://doc.qt.io/qt-6/threads-technologies.html)

## 📝 Kısa Özet

- **UI Freeze:** Uzun süren işlemler GUI threadinde çalıştığında arayüz donar.
- **QThread:** Zaman alan işleri arka planda çalıştırmak için kullanılır.
- **Worker Mantığı:** `QObject` miras alan bir sınıf, `QThread` içinde `moveToThread` ile taşınır.
- **Sinyaller:** `pyqtSignal` ile arka plan iş parçacığından ana iş parçacığına veri (ilerleme, sonuç) gönderilir.
- **Güvenli Güncelleme:** Sadece ana iş parçacığı GUI widget'larını değiştirebilir; arka plandan sinyal gönderilir.
- **Kaynak Yönetimi:** Thread bittiğinde `quit()`, `wait()` ve `deleteLater` ile temizlik yapılır.
- **İptal:** Güvenli iptal için Worker içinde bayrak kontrolü kullanılabilir.

---
[⬅️ Önceki Ders](10-Data-And-Sqlite.md) | [Sonraki Ders ➡️](12-Error-Handling-Ux.md)
