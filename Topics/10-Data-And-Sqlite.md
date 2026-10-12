# Veri Saklama ve SQLite

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Uygulamada verileri kalıcı hale getirme yöntemlerini (JSON, CSV, SQLite) açıklayabilmek.
- SQLite veritabanı bağlantısı kurabilmek ve temel CRUD işlemlerini (Create, Read, Update, Delete) uygulayabilmek.
- Veritabanı verilerini QTableWidget ile senkronize edebilmek.
- Veritabanı kayıtlarını CSV dosyasına aktarabilmek.
- Parametreli sorgular kullanarak SQL Injection saldırılarından korunabilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Bir masaüstü uygulaması kapatıldığında, kullanıcının girdiği tüm bilgiler kaybolmamalıdır. Düşünün bir not defteri uygulaması; her açtığınızda eski notlarınızın yoğolutulması kabul edilemez olur. 

Veri kalıcılığı (persistence), uygulamanın yeniden başlatıldığında önceki durumunu hatırlayabilmesini sağlar. Bunun için çeşitli yöntemler vardır:
- **JSON/CSV Dosyaları:** Basit ve insan tarafından okunabilir, ancak büyük veri kümeleri ve karmaşık sorgular için yetersiz.
- **SQLite:** Hafif, dosya tabanlı ve sıfır yapılandırma gerektiren bir veritabanıdır. Python'un standart kütüphanesi (`sqlite3`) ile sorunsuz çalışır. Karmaşık veri ilişkileri, sorgulamalar (filtreleme, sıralama, gruplama) ve güncelleme işlemleri için ideal bir seçenektir.

> [!NOTE]
> SQLite, mobil uygulamalar (Android/iOS),ウェブ 브라우저 ve birçok masaüstü programında yerel veri depolama için yaygın olarak kullanılır. Tek bir `.db` dosyasında tüm veritabanınızı taşıyabilirsiniz.

## 🔍 Birlikte Keşfedelim

Bir uygulamada veriyi geçici bellekte (RAM) tutmanın sınırlılığını bir örnekle görelim:

1.  Bir form doldurup "Kaydet" butonuna basarsınız.
2.  Veri bir listeye (örneğin `self.kayitlar = []`) eklenir ve bir `QTableWidget` görüntülenir.
3.  Uygulamayı kapatıp yeniden açarsınız.
4.  Liste boştur çünkü program belleği temizlendi; tüm veriniz kayboldu!

Bu durumda, veriyi **disk üzerinde kalıcı bir yerde** saklamamız gerekiyor. İşte SQLite tam bu noktada devreye girer: Verilerinizi bir dosyada (`veritabani.db`) tutarak, uygulama kapansada bile verilerinizi korursunuz.

## 🛠️ Uygulama

Bu derste, öğrenci kayıtlarını tutan, formdan veri alıp veritabanına kaydeden ve veritabanındaki verileri bir tabloda gösteren bir uygulama oluşturacağız. Ayrıca, verileri CSV dosyasına aktaracak bir özellik de ekleyeceğiz.

### Adım 1: Veritabanı Bağlantısı ve Tablo Oluşturma

Uygulama her başlatıldığında, `ogrenciler.db` adında bir veritabanı dosyası oluşturur (eğer yoksa) ve içinde `ogrenciler` tablosunu hazırlar.

### Adım 2: Formdan Veri Alma ve Veritabanına Kaydetme

Kullanıcı formu doldurduğunda, veriler parametreli bir `INSERT INTO` sorgusu ile veritabanına eklenir. Bu yöntem, SQL Injection saldırılarını önler.

### Adım 3: Veritabanı Verilerini QTableWidget'te Gösterme

Veritabanındaki tüm kayıtlar `SELECT * FROM ogrenciler` sorgusu ile çekilir ve `QTableWidget`에 satır satır eklenir.

### Adım 4: CSV Dışa Aktarımı

Veritabanındaki tüm veriler `SELECT` ile çekilir ve Python'ın `csv` modülü kullanarak `ogrenciler.csv` dosyasına yazılır.

```python
import sys
import csv
import sqlite3
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QLineEdit, QPushButton, QTableWidget, 
                             QTableWidgetItem, QLabel, QMessageBox, QHeaderView)
from PySide6.QtCore import Qt

class VeriSaklamaUygulamasi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initVeritabani()  # Veritabanını başlat
        self.initUI()

    def initVeritabani(self):
        """Veritabanı bağlantısını kurar ve tabloyu oluşturur (eğer yoksa)."""
        self.baglanti = sqlite3.connect("ogrenciler.db")
        self.imlec = self.baglanti.cursor()
        # Tabloyu oluştur (eğer yoksa)
        self.imlec.execute("""
            CREATE TABLE IF NOT EXISTS ogrenciler (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                isim TEXT NOT NULL,
                soyisim TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                yas INTEGER
            )
        """)
        self.baglanti.commit()

    def initUI(self):
        self.setWindowTitle("Öğrenci Kayıt Sistemi (SQLite)")
        self.resize(800, 500)

        # Merkezi alan ve Ana Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.ana_layout = QVBoxLayout(self.merkezi_alan)

        # --- ÜST KISIM: GİRİŞ FORMU ---
        self.form_layout = QHBoxLayout()
        
        self.input_isim = QLineEdit()
        self.input_isim.setPlaceholderText("İsim")
        self.form_layout.addWidget(self.input_isim)

        self.input_soyisim = QLineEdit()
        self.input_soyisim.setPlaceholderText("Soyisim")
        self.form_layout.addWidget(self.input_soyisim)

        self.input_email = QLineEdit()
        self.input_email.setPlaceholderText("E-posta")
        self.form_layout.addWidget(self.input_email)

        self.input_yas = QLineEdit()
        self.input_yas.setPlaceholderText("Yaş")
        self.form_layout.addWidget(self.input_yas)

        self.btn_kaydet = QPushButton("Kaydet")
        self.btn_kaydet.clicked.connect(self.kayit_ekle)
        self.form_layout.addWidget(self.btn_kaydet)

        self.btn_csv_aktar = QPushButton("CSV Dışa Aktar")
        self.btn_csv_aktar.clicked.connect(self.csv_disa_aktar)
        self.form_layout.addWidget(self.btn_csv_aktar)

        self.ana_layout.addLayout(self.form_layout)

        # --- ALT KISIM: TABLO ---
        self.tablo = QTableWidget()
        self.tablo.setColumnCount(5) # ID, İsim, Soyisim, E-posta, Yaş
        self.tablo.setHorizontalHeaderLabels(["ID", "İsim", "Soyisim", "E-posta", "Yaş"])
        
        # Sütun genişliklerini içeriğe göre ayarla
        self.tablo.horizontalHeader().setSectionResizeMode(QHeaderView.Stretch)
        # Satır seçimi modu: Tüm satırı seç
        self.tablo.setSelectionBehavior(QTableWidget.SelectRows)
        # Tek seferde tek satır seçilebilir
        self.tablo.setSelectionMode(QTableWidget.SingleSelection)
        
        # Veritabanındaki verileri tabloya yükle
        self.verileri_yukle_ve_goster()

        self.ana_layout.addWidget(self.tablo)

        # --- ALT BUTONLAR (Silme/Güncelleme için ileride eklenebilir) ---
        # Şimdilik sadece boş bir layout bırakıyoruz
        self.ana_layout.addStretch()

    def verileri_yukle_ve_goster(self):
        """Veritabanındaki verileri çekip QTableWidget'te gösterir."""
        # Önce tabloyu temizle
        self.tablo.setRowCount(0)
        
        # Veritabanından tüm kayıtları çek
        self.imlec.execute("SELECT id, isim, soyisim, email, yas FROM ogrenciler")
        kayitlar = self.imlec.fetchall()
        
        # Her kaydı tabloya ekle
        for satir_numarasi, kayit in enumerate(kayitlar):
            self.tablo.insertRow(satir_numarasi)
            for sutun_sayisi, deger in enumerate(kayit):
                item = QTableWidgetItem(str(deger))
                # ID sütunu düzenlenmesin
                if sutun_sayisi == 0:
                    item.setFlags(item.flags() & ~Qt.ItemIsEditable)
                self.tablo.setItem(satir_numarasi, sutun_sayisi, item)

    def kayit_ekle(self):
        isim = self.input_isim.text().strip()
        soyisim = self.input_soyisim.text().strip()
        email = self.input_email.text().strip()
        yas_str = self.input_yas.text().strip()

        # Temel doğrulama
        if not isim or not soyisim or not email:
            QMessageBox.warning(self, "Eksik Bilgi", "İsim, soyisim ve e-posta alanları zorunludur.")
            return

        # Yaş doğrulama (sayı ve makul bir aralıkta)
        try:
            yas = int(yas_str) if yas_str else None
            if yas is not None and (yas < 0 or yas > 120):
                raise ValueError
        except ValueError:
            QMessageBox.warning(self, "Geçersiz Yaş", "Yaş 0 ile 120 arasında bir tam sayı olmalıdır.")
            return

        try:
            # PARAMETRELİ SORGU KULLANIMI (SQL Injection önleme)
            self.imlec.execute("""
                INSERT INTO ogrenciler (isim, soyisim, email, yas)
                VALUES (?, ?, ?, ?)
            """, (isim, soyisim, email, yas))
            self.baglanti.commit()
            
            # Başarılıysa formu temizle ve tabloyu güncelle
            self.formu_temizle()
            self.verileri_yukle_ve_goster()
            QMessageBox.information(self, "Başarılı", "Kayıt başarıyla eklendi!")
        except sqlite3.IntegrityError:
            QMessageBox.warning(self, "Tekrar Eden E-posta", "Bu e-posta adresi zaten kayıtlı!")
        except Exception as e:
            QMessageBox.critical(self, "Veritabanı Hatası", f"Bir hata oluştu: {str(e)}")

    def csv_disa_aktar(self):
        """Veritabanındaki tüm verileri CSV dosyasına aktarır."""
        try:
            self.imlec.execute("SELECT isim, soyisim, email, yas FROM ogrenciler")
            kayitlar = self.imlec.fetchall()
            
            with open("ogrenciler.csv", "w", newline="", encoding="utf-8") as dosya:
                yazici = csv.writer(dosya)
                # Başlık satırı
                yazici.writerow(["İsim", "Soyisim", "E-posta", "Yaş"])
                # Veri satırları
                yazici.writerows(kayitlar)
            
            QMessageBox.information(self, "Başarılı", 
                f"{len(kayitlar)} kayıt 'ogrenciler.csv' dosyasına aktarildi.")
        except Exception as e:
            QMessageBox.critical(self, "CSV Hatası", f"Dosya yazılırken bir hata oluştu: {str(e)}")

    def formu_temizle(self):
        self.input_isim.clear()
        self.input_soyisim.clear()
        self.input_email.clear()
        self.input_yas.clear()

    def closeEvent(self, event):
        """Uygulama kapatılırken veritabanı bağlantısını güvenli şekilde kapatır."""
        self.baglanti.close()
        event.accept()

def main():
    app = QApplication(sys.argv)
    pencere = VeriSaklamaUygulamasi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Bağlantı Kontrolü:** Uygulamayı çalıştırın, birkaç kayıt ekleyin ve kapatın. Daha sonra yeniden açın. Kayıtlarınız hâlâ tabloDA mı?
2.  **Dosya Kontrolü:** Proje klasörünüzde `ogrenciler.db` adında bir dosya oluştu mu? Bu dosyayı bir SQLite tarayıcı programı (örneğin DB Browser for SQLite) ile açarak içeriğini görüntüleyebilirsiniz.
3.  **CSV Aktarımı:** "CSV Dışa Aktar" butonuna basın. `ogrenciler.csv` dosyası oluştu mu? İçeriği doğru mu?
4.  **SQL Injection Denemesi:** E-posta alanına `'; DROP TABLE ogrenciler; --` gibi bir değer girip kaydetmeyi deneyin. Uygulama bu saldırıyı engelledi mi? (İpucu: Parametreli sorgu kullanmamız bu tür saldırıları önler.)

## 🧠 Kendini Test Et

1. **Soru:** Veritabanı bağlantısını her işlemde açup kapatmak yerine, uygulama ömrü boyunca açık tutmak daha iyi mi?
2. **Soru:** `?` işaretleri neyi temsil eder?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Genellikle uygulama ömrü boyunca açık bir bağlantı tutmak daha performanslıdır. Ancak, çoklu iş parçacıklı (threading) kullanıyorsanız dikkatli olmak gerekir. Bu dersimizdeki gibi basit tek iş parçacıklı uygulamalar için uygulama başlangıcında açma ve kapanışta kapatma yeterlidir.
2. **Cevap:** Bu, **parametreli sorgu** (parameterized query) Placeholder'larıdır. Gerçek değerler (`isim`, `soyisim`, `email`, `yas`) sorgudan sonra ayrı bir tuple olarak verilir. Bu sayede kullanıcı girdisi doğrudan sorgunun içinde değil, güvenli bir şekilde işlenir ve SQL Injection önlenir.

</details>
<summary>Cevapları göster</summary>

1. **Soru:** Veritabanı bağlantısını her işlemde açup kapatmak yerine, uygulama ömrü boyunca açık tutmak daha iyi mı?
   **Cevap:** Genellikle uygulama ömrü boyunca açık bir bağlantı tutmak daha performanslıdır. Ancak, çoklu iş parçacığı (threading) kullanıyorsanız dikkatli olmak gerekir. Bu dersimizdeki gibi basit tek iş parçacıklı uygulamalar için uygulama başlangıcında açma ve kapanışta kapatma yeterlidir.

2. **Soru:** `INSERT INTO ogrenciler (isim, soyisim, email, yas) VALUES (?, ?, ?, ?)` sorgusunda `?` işaretleri neyi temsil eder?
   **Cevap:** Bu, **parametreli sorgu** (parameterized query) Placeholder'larıdır. Gerçek değerler (`isim`, `soyisim`, `email`, `yas`) sorgudan sonra ayrı bir tuple olarak verilir. Bu sayede kullanıcı girdisi doğrudan sorgunun içinde değil, güvenli bir şekilde işlenir ve SQL Injection önlenir.

3. **Soru:** `CREATE TABLE IF NOT EXISTS` ifadesi neden kullanılır?
   **Cevap:** Eğer tablo zaten varsa, hata vermek yerine sadece geçer. Böylece uygulama her başlatıldığında tabloyu yeniden oluşturmaya çalışmaz; sadece yoksa oluşturur.

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Uygulamaya bir **"Sil"** butonu ekleyin. Seçili satırdaki kaydı veritabanından silin ve tabloyu güncelleyin.
2.  Silme işlemi öncesinde kullanıcıdan `QMessageBox.question` ile onay alın.
3.  İpucu: Silinen kaydın `id` değerini `self.tablo.item(seçili_satir, 0).text()` ile alın ve `DELETE FROM ogrenciler WHERE id = ?` sorgusunu çalıştırın.

## 📚 Daha Fazlasını Öğrenmek

- [Python SQLite3 Dokumentasyonu](https://docs.python.org/3/library/sqlite3.html)
- [SQLite Official Website](https://www.sqlite.org/index.html)
- [DB Browser for SQLite (GUI Tool)](https://sqlitebrowser.org/)

## 📝 Kısa Özet

- **Veri Kalıcılığı:** Uygulama kapatıldığında verilerin kaybolmaması için gereklidir.
- **SQLite:** Hafif, dosya tabanlı ve Python ile entegrasyonu kolay olan bir veritabanıdır.
- **Bağlantı:** `sqlite3.connect("dosya.db")` ile kurulur; `close()` ile kapatılır.
- **CRUD İşlemleri:** 
  - Create: `INSERT INTO tablo VALUES (...)`
  - Read: `SELECT * FROM tablo`
  - Update: `UPDATE tablo SET kolon = değer WHERE koşul`
  - Delete: `DELETE FROM tablo WHERE koşul`
- **Güvenlik:** Her zaman **parametreli sorgular** (`?` veya `:isim` placeholderları) kullanarak SQL Injection saldırılarından korunun.
- **Entegrasyon:** `QTableWidget`, `setRowCount()` ve `setItem()` metotları ile veritabanı sonuçlarıyla senkronize edilir.
- **CSV Dışa Aktarımı:** Python'ın `csv` modülü ile veriler `dosya.csv` olarak yazılabilir.

---
[⬅️ Önceki Ders](09-Data-Listing-Widgets.md) | [Sonraki Ders ➡️](11-Multithreading-Qthread.md)