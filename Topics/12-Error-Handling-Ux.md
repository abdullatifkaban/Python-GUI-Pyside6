# Hata Yönetimi ve Kullanıcı Deneyimi (UX)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Uygulama çökmesini (crash) engellemek için **`try-except` bloklarını** GUI kodlarıyla entegre edebilmek.
- Beklenmeyen (yakalanmamış) hataları **`sys.excepthook`** ile global olarak yakalayabilmek.
- Kullanıcıya teknik hata detayları yerine, anlaşılır ve yönlendirici **`QMessageBox`** mesajları gösterebilmek.
- Hataları arka planda bir dosyaya (**logging**) kaydederek hata takibi ve analizi yapabilmek.
- Kullanıcı dostu bir deneyim (UX) için hata anında uygulamanın kullanıma açık kalmasını sağlayabilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Şu ana kadar yazdığımız uygulamalarda, bir hata oluştuğunda (örneğin veritabanı bağlantısı koparsa veya bir dosya bulunamazsa) program genellikle ani bir şekilde kapanır ("crash" olur) veya kullanıcıyı yutan karmaşık bir Python "Traceback" hata mesajı ekrana basar. Bu durum:
1.  Kullanıcıyı korkutur ve güvenini zedeler.
2.  Geliştiriciye neyin yanlış gittiği hakkında **yeterli bilgi vermez** (kullanıcı "hata aldı" der ama ne olduğunu bilmez).
3.  Veri kaybına yol açabilir (kullanıcı formu doldurmuş, kaydetmeden önce hata olmuş olabilir).

Profesyonel bir yazılım; hataları **önlemeli**, **yakalamalı**, **kaydetmeli** ve kullanıcıya **"Ne oldu?"** ve **"Ne yapmalıyım?"** sorularına cevap veren bir şekilde bildirmelidir.

## 🔍 Birlikte Keşfedelim

Bir asansörü düşünün. Asansör bir arıza yaptığında (örneğin bir kablo koptuğunda):
1.  **Kontrolsüz düşmez** (güvenlik frenleri tutar - `try-except` / `excepthook`).
2.  **Yolcuları bilgilendirir:** "Arıza tespit edildi, acil duraklanma devreye girdi" (`QMessageBox.critical`).
3.  **Arızayı kaydeder:** Bakım ekibinin daha sonra inceleyebileceği bir log dosyasına yazılır (`logging` modülü).
4.  **Güvenli bir duruma getirir:** En yakın kata gelir ve kapılarını açar (uygulama kapanmaz, kullanıcı verisini kurtarabilir).

İşte biz de kodlarımızda bu "güvenlik frenlerini" ve "yolcu bilgilendirme sistemlerini" kuracağız.

## 🛠️ Uygulama

Bu derste, bir dosya okuyan ve içeriğini gösteren basit bir editörde; olası hataları (dosya bulunamama, okuma izni yok, beklenmedik hata) yakalayıp loglayan ve kullanıcıya nazikçe bildiren bir yapı kuracağız.

### Adım 1: Loglama (Logging) Altyapısını Kurma

Uygulama başlangıcında, hataların yazılacağı bir `app.log` dosyasını ve formatı yapılandırıyoruz.

### Adım 2: Global Hata Yakalayıcı (`sys.excepthook`)

Programın herhangi bir yerinde yakalanmamış bir hata olursa, program çökmeden hemen önce bu fonksiyon çalıştırılır. Burada hatayı loglarız ve kullanıcıya mesaj gösteririz.

### Adım 3: Yerel Hata Yönetimi (`try-except`)

Belli riskli işlemler (dosya okuma, veritabanı sorgusu) için özel `try-except` blokları yazarak kullanıcıya spesifik çözümler sunarız.

```python
import sys
import traceback
import logging
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, 
                             QVBoxLayout, QPushButton, QTextEdit, QFileDialog, 
                             QMessageBox, QLabel)
from PySide6.QtCore import Qt

# --- 1. LOGGING YAPILANDIRMASI ---
# Uygulama genelinde kullanılacak logger ayarları
logging.basicConfig(
    filename="app.log",           # Log dosyasının adı
    level=logging.ERROR,          # Sadece ERROR ve üstü seviyeleri kaydet
    format="%(asctime)s - %(levelname)s - %(message)s", # Zaman, Seviye, Mesaj
    encoding="utf-8"              # Türkçe karakter desteği
)

# Global hata yakalayıcı fonksiyonu
def global_hata_yakalayici(exc_type, exc_value, exc_traceback):
    """
    Yakalanmamış (unhandled) hatalar için son çare güvenlik ağı.
    Uygulama çökmeden hemen önce bu fonksiyon çalışır.
    """
    # Hata detaylarını string olarak al
    hata_detaylari = "".join(traceback.format_exception(exc_type, exc_value, exc_traceback))
    
    # 1. Hatayı log dosyasına yaz
    logging.error(f"YAKALANAN HATA:\n{hata_detaylari}")
    
    # 2. Kullanıcıya nazik bir mesaj göster (Eğer QApplication henüz başlatıldıysa)
    # Not: Bu fonksiyon çok erken çağrılırsa QMessageBox çalışmayabilir.
    # Güvenli bir yol: QApplication.instance() kontrolü.
    app_instance = QApplication.instance()
    if app_instance:
        QMessageBox.critical(
            None, 
            "Beklenmeyen Bir Hata Oluştu", 
            "Uygulamada beklenmeyen bir sorun oluştu.\n"
            "Detaylar 'app.log' dosyasına kaydedildi.\n\n"
            "Lütfen uygulamayı yeniden başlatın."
        )
    
    # Standart hata çıktısını de bastır (konsol için)
    sys.__excepthook__(exc_type, exc_value, exc_traceback)

# Global hook'u aktif et
sys.excepthook = global_hata_yakalayici

# --- ANA PENCERE SINIFI ---
class GuvenliEditor(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Güvenli Dosya Okuyucu (Hata Yönetimi)")
        self.resize(600, 400)

        # Merkezi alan ve Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.layout = QVBoxLayout(self.merkezi_alan)

        self.etiket = QLabel("Bir metin dosyası seçin:")
        self.layout.addWidget(self.etiket)

        self.btn_dosya_sec = QPushButton("Dosya Seç ve Oku")
        self.btn_dosya_sec.clicked.connect(self.dosya_oku)
        self.layout.addWidget(self.btn_dosya_sec)

        self.btn_hata_uret = QPushButton("🧪 Test: Beklenmeyen Hata Üret (Sıfıra Bölme)")
        self.btn_hata_uret.clicked.connect(self.beklenmeyen_hata_uret)
        self.layout.addWidget(self.btn_hata_uret)

        self.text_alani = QTextEdit()
        self.text_alani.setReadOnly(True)
        self.text_alani.setPlaceholderText("Dosya içeriği burada görünecek...")
        self.layout.addWidget(self.text_alani)

    def dosya_oku(self):
        """Kullanıcıdan dosya seçmesini isteyip, hataları yöneterek okur."""
        dosya_yolu, _ = QFileDialog.getOpenFileName(
            self, "Metin Dosyası Seç", "", "Metin Dosyaları (*.txt);;Tüm Dosyalar (*)"
        )

        if not dosya_yolu:
            return # Kullanıcı iptal etti

        # --- YEREL HATA YÖNETİMİ (TRY-EXCEPT) ---
        try:
            # Riskli işlem: Dosya açma ve okuma
            with open(dosya_yolu, 'r', encoding='utf-8') as f:
                icerik = f.read()
            
            self.text_alani.setText(icerik)
            self.etiket.setText(f"✅ Başarıyla okundu: {dosya_yolu}")
            
        except FileNotFoundError:
            # Spesifik hata: Dosya yok
            logging.error(f"Dosya bulunamadı: {dosya_yolu}")
            QMessageBox.warning(self, "Dosya Bulunamadı", 
                f"Seçilen dosya sistemde bulunamadı:\n{dosya_yolu}\n\n"
                "Dosya taşınmış veya silinmiş olabilir.")
            
        except PermissionError:
            # Spesifik hata: İzin yok
            logging.error(f"Okuma izni yok: {dosya_yolu}")
            QMessageBox.critical(self, "Erişim Engellendi", 
                "Bu dosyayı okuma izniniz yok.\n"
                "Yönetici olarak çalıştırmayı deneyin veya dosya izinlerini kontrol edin.")
            
        except UnicodeDecodeError:
            # Spesifik hata: Kodlama hatası (binary dosya seçilmiş olabilir)
            logging.error(f"Kodlama hatası (UTF-8 değil): {dosya_yolu}")
            QMessageBox.warning(self, "Uyumsuz Dosya Formatı", 
                "Seçilen dosya metin formatında değil veya UTF-8 kodlaması kullanmıyor.\n"
                "Lütfen bir .txt dosyası seçin.")
            
        except Exception as e:
            # Genel hata: Öngörmediğimiz herhangi bir hata
            logging.exception(f"Dosya okunurken beklenmeyen hata: {dosya_yolu}")
            QMessageBox.critical(self, "Bilinmeyen Hata", 
                f"Dosya okunurken bir hata oluştu:\n{str(e)}\n\n"
                "Detaylar log dosyasına kaydedildi.")

    def beklenmeyen_hata_uret(self):
        """Global excepthook'un çalışmasını test etmek için kasıtlı hata üretir."""
        # Bu hata bir try-except içinde DEĞİL, doğrudan tetikleniyor.
        # Bu yüzden sys.excepthook devreye girecek.
        x = 10 / 0  # ZeroDivisionError

def main():
    app = QApplication(sys.argv)
    # High DPI scaling için (isteğe bağlı ama iyi pratik)
    # app.setAttribute(Qt.AA_EnableHighDpiScaling) # PySide6'da genellikle varsayılan
    pencere = GuvenliEditor()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Kritik Noktaları:**
- **`logging.basicConfig`**: Uygulama başlar başlamaz log dosyasını (`app.log`), formatı ve seviyeyi ayarlar.
- **`sys.excepthook = global_hata_yakalayici`**: Bu satır, **yakalanmamış tüm hatalar** için son bir güvenlik ağı kurar. Uygulama çökmez, loglar ve kullanıcıyı uyarır.
- **`logging.exception(...)`**: `except` bloğu içinde kullanıldığında, hatanın **tam traceback'ini (izini)** log dosyasına otomatik ekler. `logging.error` sadece mesajı yazar.
- **Hata Türlerine Özel Yakalama**: `FileNotFoundError`, `PermissionError`, `UnicodeDecodeError` gibi spesifik hataları ayrı ayrı yakalayıp kullanıcıya **farklı, işlem yapılabilir** mesajlar veriyoruz.
- **`QMessageBox` Türleri**: 
    - `warning`: Kullanıcı düzeltebileceği bir durum (dosya yok).
    - `critical`: Ciddi bir sorun, işlem yapılamıyor (izin yok, beklenmedik hata).

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Normal Akış:** Geçerli bir `.txt` dosyası seçin. İçerik gelmeli.
2.  **Dosya Bulunamadı:** Bir dosya seçin, ardından o dosyayı silin veya taşıyın ve tekrar aynı dosyayı seçmeyi deneyin (veya elle yazın). "Dosya Bulunamadı" uyarısı gelmeli.
3.  **İzin Hatası:** (Linux/Mac) `chmod 000 test.txt` yaparak bir dosyanın okuma iznini kaldırın, ardından o dosyayı seçin. "Erişim Engellendi" uyarısı gelmeli. (Test sonrası `chmod 644 test.txt` ile düzeltmeyi unutmayın).
4.  **Global Hata Testi:** "🧪 Test: Beklenmeyen Hata Üret" butonuna basın. Uygulama **çökmeden**, bir "Beklenmeyen Bir Hata Oluştu" mesajı göstermeli ve `app.log` dosyasına `ZeroDivisionError` yazmalı.
5.  **Log Dosyasını Kontrol Edin:** Proje klasöründeki `app.log` dosyasını açın. Tüm hataların zaman damgalı olarak kaydedildiğini görün.

## 🧠 Kendini Test Et

1. **Soru:** `try-except` bloğu ile `sys.excepthook` arasındaki fark nedir? Hangi durumda hangisi kullanılır?
2. **Soru:** `logging.error` ve `logging.exception` farkı nedir?
3. **Soru:** Kullanıcıya `traceback` (hata izi) göstermemek neden önemlidir?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** 
   - `try-except`: **Bildiğimiz riskli** kod blokları için (dosya açma, veritabanı). Hatanın *nerede* olabileceğini biliyorsak ve kullanıcıya *spesifik* bir çözüm sunmak istiyorsak kullanırız.
   - `sys.excepthook`: **Bilmediğimiz, beklenmedik** hatalar için (programın herhangi bir yerinde). Kodun herhangi bir yerinde beklenmedik bir hata olursa uygulamanın çökmesini engellemek ve loglamak için "son çare" ağıdır.

2. **Cevap:** `logging.error` sadece verdiğiniz mesajı yazar. `logging.exception` ise **o anki hata traceback'ini (stack trace)** de otomatik olarak mesaja ekler. `except` bloğu içinde her zaman `logging.exception` kullanmak daha faydalıdır.

3. **Cevap:** 
   - Güvenlik: Hata izi dosya yolları, kod yapısı, kütüphane versiyonları gibi hassas bilgiler içerebilir.
   - Kullanıcı Deneyimi: Kullanıcı "TypeError at line 45 in module..." gördüğünde ne yapacağını bilemez. "Dosya bulunamadı, lütfen kontrol edin" anlar.

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Uygulamaya bir **"Logları Göster"** butonu ekleyin.
2.  Bu butona basıldığında `app.log` dosyasının içeriğini okuyan ve bir `QMessageBox` (veya yeni bir `QDialog`) içinde gösteren bir fonksiyon yazın.
3.  Eğer log dosyası yoksa veya boşsa "Henüz hata kaydı yok" mesajı verin.

## 📚 Daha Fazlasını Öğrenmek

- [Python Logging HOWTO](https://docs.python.org/3/howto/logging.html)
- [Python sys.excepthook Documentation](https://docs.python.org/3/library/sys.html#sys.excepthook)
- [PySide6 QMessageBox Documentation](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QMessageBox.html)

## 📝 Kısa Özet

- **Proaktif Yönetim:** `try-except` ile bilinen riskli alanları sarın.
- **Reaktif Güvenlik Ağı:** `sys.excepthook` ile bilinmeyen hataları yakalayın.
- **Loglama:** `logging` modülü ile hataları zaman damgalı, kategorili ve traceback'li bir şekilde `app.log` dosyasına yazın.
- **Kullanıcı İletişimi:** Teknik detayları loga yazın, kullanıcıya **"Ne oldu?"** ve **"Ne yapmalı?"** sorularına cevap veren sade mesajlar (`QMessageBox`) gösterin.
- **Uygulama Hayat Döngüsü:** `closeEvent` veya `sys.excepthook` içinde kaynakları (veritabanı bağlantısı, thread'ler) temizleyin.

---
[⬅️ Önceki Ders](11-Multithreading-Qthread.md) | [Sonraki Ders ➡️](13-Capstone-And-Pyinstaller.md)