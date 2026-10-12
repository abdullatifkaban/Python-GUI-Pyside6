# Veri Listeleme (List & Table)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- `QListWidget` kullanarak basit, tek sütunlu veri listeleri oluşturabilmek.
- `QTableWidget` kullanarak sütun ve satırlardan oluşan karmaşık veri tabloları inşa edebilmek.
- Tabloya veri ekleme, silme, güncelleme ve hücrelere erişme işlemlerini yapabilmek.
- Kullanıcının tablodan seçtiği satırı tespit edip işlem yapabilmek.
- Veri listeleme bileşenlerini Layout sistemine entegre edebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Gerçek dünyadaki veriler genellikle listeler veya tablolar halindedir: bir e-posta kutusu, bir kullanıcı listesi, bir stok takibi veya bir not defteri. 

Şu ana kadar öğrendiğimiz `QLineEdit` veya `QLabel`, tek bir veri parçasını göstermek için uygundur. Ancak "Tüm müşterileri listele" veya "Bu ay yapılan satışları göster" gibi senaryolarda; veriyi satırlara ve sütunlara ayırarak, kullanıcının sıralayabildiği, seçebildiği ve düzenleyebildiği yapılandırılmış bir görünüme ihtiyaç duyarız. İşte `QListWidget` ve `QTableWidget` tam bu noktada devreye girer.

## 🔍 Birlikte Keşfedelim

Bir elektronik tablo (Excel/Calc) uygulamasını düşünün:

1.  **Satırlar (Rows):** Her bir kayıt (örneğin bir müşteri, bir ürün).
2.  **Sütunlar (Columns):** O kaydın özellikleri (İsim, E-posta, Telefon, Adres).
3.  **Hücreler (Cells):** Satır ve sütunun kesişimindeki tek bir veri parçası.

`QTableWidget` tam bu mantıkla çalışır. Siz satır ve sütun sayısını belirlersiniz, ardından her hücreye bir `QTableWidgetItem` koyarsınız.

`QListWidget` ise bu tablonun basitleştirilmiş, sadece tek sütunlu bir versiyonudur; "Yapılacaklar Listesi" gibi basit listeler için idealdir.

> [!TIP]
> `QTableWidget` kullanırken veri miktarı çok büyükse (binlerce satır) performans sorunları yaşanabilir. Bu tür durumlarda ileri seviyede `QTableView` + `Model/View` mimarisi kullanılır. Bu derste öğreneceğimiz "Widget tabanlı" yaklaşım, orta ölçekli uygulamalar ve öğrenme aşaması için mükemmeldir.

## 🛠️ Uygulama

Bu derste, formdan girilen verileri bir tabloya ekleyen, tablodan seçilen veriyi silen ve seçilen veriyi formda gösteren bir **"Kullanıcı Yönetim Paneli"** inşa edeceğiz.

```python
import sys
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QLineEdit, QPushButton, QTableWidget, 
                             QTableWidgetItem, QLabel, QMessageBox, QHeaderView)
from PySide6.QtCore import Qt

class KullaniciYonetimi(QMainWindow):
    def __init__(self):
        super().__init__()
        self.secili_satir = -1 # Hangi satır seçili olduğunu takip etmek için
        self.initUI()

    def initUI(self):
        self.setWindowTitle("Kullanıcı Yönetim Paneli")
        self.resize(700, 400)

        # Merkezi alan ve Ana Layout
        self.merkezi_alan = QWidget()
        self.setCentralWidget(self.merkezi_alan)
        self.ana_layout = QVBoxLayout(self.merkezi_alan)

        # --- ÜST KISIM: GİRİŞ FORMU ---
        self.form_layout = QHBoxLayout()
        
        self.input_isim = QLineEdit()
        self.input_isim.setPlaceholderText("İsim Soyisim")
        self.form_layout.addWidget(self.input_isim)

        self.input_email = QLineEdit()
        self.input_email.setPlaceholderText("E-posta")
        self.form_layout.addWidget(self.input_email)

        self.btn_ekle = QPushButton("Ekle")
        self.btn_ekle.clicked.connect(self.kayit_ekle)
        self.form_layout.addWidget(self.btn_ekle)

        self.btn_guncelle = QPushButton("Güncelle")
        self.btn_guncelle.clicked.connect(self.kayit_guncelle)
        self.btn_guncelle.setEnabled(False) # Başlangıçta pasif
        self.form_layout.addWidget(self.btn_guncelle)

        self.ana_layout.addLayout(self.form_layout)

        # --- ALT KISIM: TABLO ---
        self.tablo = QTableWidget()
        self.tablo.setColumnCount(3) # 3 Sütun
        self.tablo.setHorizontalHeaderLabels(["ID", "İsim Soyisim", "E-posta"])
        
        # Sütun genişliklerini içeriğe göre ayarla
        self.tablo.horizontalHeader().setSectionResizeMode(QHeaderView.Stretch)
        # Satır seçimi modu: Tüm satırı seç
        self.tablo.setSelectionBehavior(QTableWidget.SelectRows)
        # Tek seferde tek satır seçilebilir
        self.tablo.setSelectionMode(QTableWidget.SingleSelection)
        
        # Hücre tıklandığında sinyal
        self.tablo.itemClicked.connect(self.satir_secildi)

        self.ana_layout.addWidget(self.tablo)

        # --- ALT BUTONLAR ---
        self.alt_butonlar = QHBoxLayout()
        self.btn_sil = QPushButton("Seçiliyi Sil")
        self.btn_sil.clicked.connect(self.kayit_sil)
        self.btn_sil.setEnabled(False)
        self.alt_butonlar.addWidget(self.btn_sil)
        
        self.ana_layout.addLayout(self.alt_butonlar)

        # Örnek veri
        self.ornek_veri_yukle()

    def ornek_veri_yukle(self):
        ornekler = [
            (1, "Ahmet Yılmaz", "ahmet@ornek.com"),
            (2, "Ayşe Demir", "ayse@ornek.com"),
        ]
        for id_, isim, email in ornekler:
            self.tabloya_ekle(id_, isim, email)

    def tabloya_ekle(self, id_, isim, email):
        """Yardımcı metot: Tabloya yeni satır ekler."""
        satir = self.tablo.rowCount()
        self.tablo.insertRow(satir)
        
        # QTableWidgetItem oluşturup tabloya yerleştir
        item_id = QTableWidgetItem(str(id_))
        item_id.setFlags(item_id.flags() & ~Qt.ItemIsEditable) # ID düzenlenmesin
        self.tablo.setItem(satir, 0, item_id)
        
        self.tablo.setItem(satir, 1, QTableWidgetItem(isim))
        self.tablo.setItem(satir, 2, QTableWidgetItem(email))

    def kayit_ekle(self):
        isim = self.input_isim.text().strip()
        email = self.input_email.text().strip()

        if not isim or not email:
            QMessageBox.warning(self, "Eksik Bilgi", "Lütfen tüm alanları doldurun.")
            return

        # Basit ID üretimi: Mevcut max ID + 1
        yeni_id = self.tablo.rowCount() + 1
        self.tabloya_ekle(yeni_id, isim, email)
        
        self.formu_temizle()

    def satir_secildi(self, item):
        """Tablodan bir satır seçildiğinde formu doldur."""
        self.secili_satir = item.row()
        
        # Seçili satırdan verileri al
        id_item = self.tablo.item(self.secili_satir, 0)
        isim_item = self.tablo.item(self.secili_satir, 1)
        email_item = self.tablo.item(self.secili_satir, 2)

        self.input_isim.setText(isim_item.text())
        self.input_email.setText(email_item.text())

        # Güncelle ve Sil butonlarını aktif et
        self.btn_guncelle.setEnabled(True)
        self.btn_sil.setEnabled(True)
        self.btn_ekle.setEnabled(False) # Seçiliyken eklemeyi engelle (opsiyonel)

    def kayit_guncelle(self):
        if self.secili_satir == -1:
            return

        yeni_isim = self.input_isim.text().strip()
        yeni_email = self.input_email.text().strip()

        if not yeni_isim or not yeni_email:
            QMessageBox.warning(self, "Eksik Bilgi", "Alanlar boş olamaz.")
            return

        # Hücreleri güncelle
        self.tablo.item(self.secili_satir, 1).setText(yeni_isim)
        self.tablo.item(self.secili_satir, 2).setText(yeni_email)
        
        self.formu_temizle()

    def kayit_sil(self):
        if self.secili_satir == -1:
            return

        cevap = QMessageBox.question(
            self, "Onay", "Seçili kaydı silmek istediğinizden emin misiniz?",
            QMessageBox.Yes | QMessageBox.No
        )
        
        if cevap == QMessageBox.Yes:
            self.tablo.removeRow(self.secili_satir)
            self.formu_temizle()

    def formu_temizle(self):
        self.input_isim.clear()
        self.input_email.clear()
        self.secili_satir = -1
        self.btn_guncelle.setEnabled(False)
        self.btn_sil.setEnabled(False)
        self.btn_ekle.setEnabled(True)

def main():
    app = QApplication(sys.argv)
    pencere = KullaniciYonetimi()
    pencere.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

## 🧪 Deneyelim ve Gözlemleyelim

1.  **Ekleme:** Formu doldurup "Ekle"ye basın. Veri tabloya eklendi mi?
2.  **Seçme:** Tablodaki bir satıra tıklayın. Formun otomatik doldurulduğunu ve "Güncelle/Sil" butonlarının aktif olduğunu, "Ekle" butonunun pasif olduğunu kontrol edin.
3.  **Güncelleme:** Formdaki veriyi değiştirip "Güncelle"ye basın. Tablodaki verinin değiştiğini görün.
4.  **Silme:** "Sil" butonuna basın. Onay penceresi (QMessageBox) geldi mi? "Evet" dediğinizde satır silindi mi?
5.  **ID Koruması:** Tablodaki ID sütununu (ilk sütun) düzenlemeye çalışın. Düzenlenebiliyor mu? (Kodda `setFlags` ile engelledik).

## 🧠 Kendini Test Et

1. **Soru:** `QTableWidget` ile veri eklerken neden `QTableWidgetItem` nesnesi oluşturmak zorundayız?
2. **Soru:** `setSelectionBehavior(QTableWidget.SelectRows)` ne işe yarar?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** `QTableWidget` hücreleri doğrudan string kabul etmez. Her hücre bir `QTableWidgetItem` nesnesidir. Bu nesne; metin, renk, ikon, tooltip ve düzenlenebilirlik (flags) gibi özellikleri barındırır.
2. **Cevap:** Kullanıcı bir hücreye tıkladığında, sadece o hücre değil, o hücreye ait **tüm satırın** seçilmesini sağlar. Bu, "Kaydı sil" veya "Kaydı düzenle" gibi işlemler için çok daha kullanışlıdır.

</details>

## 🚀 Kendin Dene

**Görev:**
Uygulamaya bir **"Arama"** özelliği ekleyin.
1.  Üst kısma bir `QLineEdit` (placeholder: "İsim veya e-posta ara...") ve yanına bir "Ara" butonu ekleyin.
2.  "Ara" butonuna basıldığında, tablodaki tüm satırları dolaşın ve aranan metni (büyük/küçük harf duyarsız) içeren satırları **gösterin**, içermeyenleri **gizleyin** (`setRowHidden(True/False)`).
3.  Arama kutusu boşken "Ara"ya basılırsa tüm satırları tekrar görünür yapın.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Documentation: QTableWidget](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QTableWidget.html)
- [PySide6 Documentation: QListWidget](https://doc.qt.io/qtforpython-6/PySide6/QtWidgets/QListWidget.html)
- [Model/View Programming (İleri Seviye)](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QAbstractItemModel.html)

## 📝 Kısa Özet

- **`QListWidget`**: Basit, tek boyutlu listeler için uygundur (`addItem`, `takeItem`).
- **`QTableWidget`**: Sütunlu, yapılandırılmış veriler içindir. `setItem(row, col, item)` ile veri girilir.
- **`QTableWidgetItem`**: Hücre verilerini taşıyan nesnedir. `flags()` ile düzenlenebilirlik kontrol edilir.
- **Sinyaller**: `itemClicked` (hücre tıklandığında), `cellChanged` (hücre değeri değiştiğinde).
- **Yardımcı Metotlar**: `insertRow`, `removeRow`, `rowCount`, `setRowHidden`.

---
[⬅️ Önceki Ders](Topics/08-Qss-And-Styling.md) | [Sonraki Ders ➡️](Topics/10-Data-And-Sqlite.md)