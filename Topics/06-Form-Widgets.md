# 06 - Temel Form Elemanları

## 1. Temel Form Elemanları — Kavramsal Temeller

Masaüstü uygulamalarında kullanıcıdan veri almak için **form elemanları** kullanılır. Günlük hayatta bir form doldururken adını yazmak, cinsiyetini seçmek, yaşını belirtmek gibi işlemler yaparız; PySide6'te de bu işlemler için özel widget'lar sağlanır. Bu widget'lar sayesinde kullanıcıyla etkileşim kurulur, girişler alınır ve işlenir.

### Metin Girdileri: QLineEdit ve QTextEdit

Metin girişi için iki temel widget vardır: tek satırlı giriş için `QLineEdit`, çok satırlı giriş için `QTextEdit`. `QLineEdit` tek bir satır alırken, `QTextEdit` birden fazla satır ve riche text desteği sunar.

| Özellik | `QLineEdit` | `QTextEdit` |
|---------|-------------|-------------|
| Satır sayısı | Tek satır | Çok satırlı |
| Kullanım alanı | İsim, e-posta, şifre gibi kısa metinler | Açıklama, not, mesaj gibi uzun metinler |
| Rich text desteği | Yok | Var (HTML desteği) |

> [!NOTE]
> `QLineEdit` genellikle formlarda tek satırlık alanlar için tercih edilirken, `QTextEdit` daha uzun açıklamalar veya notlar için kullanılır.

### Seçim Bileşenleri: QCheckBox ve QRadioButton (Group Box Mantığı)

Kullanıcıdan seçim almak için `QCheckBox` (onay kutusu) ve `QRadioButton` (seçenek düğmesi) kullanılır. `QCheckBox` tek başına veya bir grup içinde işaretlenebilirken, `QRadioButton` genellikle bir grup içinde yalnızca bir seçeneğin seçilebilmesini sağlar.

> [!TIP]
> `QRadioButton`'ları aynı grup içinde kullanmak için genellikle `QGroupBox` veya sadece aynı mantıksal gruba bağlayarak çalışırız; Qt'de radio buttonlar otomatik olarak aynı parent içinde tek seçim zorunluluğunu uygular.

### Açılır Listeler ve Sayısal Girdiler: QComboBox ve QSpinBox

Seçim yapmak için liste sunmak istersek `QComboBox` (açılır liste) kullanırız. Sayısal girişler için ise artırma ve azaltma düğmeleri olan `QSpinBox` ya da ondalıklar için `QDoubleSpinBox` kullanılır.

| Özellik | `QComboBox` | `QSpinBox` |
|---------|-------------|------------|
| Giriş tipi | Seçim listesi | Sayısal artırma/azaltma |
| Kullanım alanı | Ülke, şehir, kategori gibi önceden tanımlı seçenekler | Yaş, sayı, miktar gibi sayısal değerler |
| Özelleştirme | Metin ve simge eklenebilir | Minimum, maksimum, adım ayarlaması |

> [!IMPORTANT]
> `QSpinBox` ile minimum ve maksimum değerleri sınırlayarak geçersiz girişleri önleyebiliriz; bu da doğrulama işlemini kolaylaştırır.

### Kullanıcı Girdilerini Doğrulama (Validation)

Kullanıcıdan gelen verilerin boş olmaması, belirli bir formatta olması veya sayısal bir aralıkta olması gibi koşulları kontrol etmek **doğrulama** (validation) denir. Form gönderilmeden önce bu kontroller yapılır; geçersiz veri girişinde kullanıcıya uygun uyarı mesajları gösterilir.

> [!WARNING]
> Boş bırakılamayan alanlar için `isEmpty()` kontrolü, e-posta formatı için düzenli ifadeler, sayısal alanlar için `value()` ile minimum/maksimum karşılaştırmaları yaygın olarak kullanılır.

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Basit bir kullanıcı formu oluşturacağız. Bu formda ad soyad için `QLineEdit`, cinsiyet için `QRadioButton` grubu, Ülke seçimi için `QComboBox` ve bir not alanı için `QTextEdit` yer alacak. Kullanıcı "Kaydet" butonuna bastığında girilen bilgiler okuyup konsola yazdırılacak. Ayrıca zorunlu alanların boş bırakılmaması kontrol edilecek; boş bırakılırsa uyarı penceresi gösterilecek. Son olarak, formun kullanımını göstermek adına `QSpinBox` ile yaş seçimi eklenecek ve yaş bilgisi de özetle dahil edilecektir.

> [!TIP]
> Bu adımları VS Code'daki `main.py` dosyasında uygulayarak, her adımda dosyayı kaydedip `python main.py` komutuyla terminalde anlık sonuçları görebilirsiniz.

---

### Adım 1: Form elemanlarını (QLineEdit, QCheckBox, QComboBox) içeren bir arayüz düzeni kurma

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import (
    QApplication, QMainWindow, QWidget,
    QLabel, QLineEdit, QRadioButton,
    QComboBox, QTextEdit, QPushButton,
    QVBoxLayout, QHBoxLayout, QGroupBox,
    QMessageBox, QSpinBox
)

class AnaPencere(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Kullanıcı Formu")
        self.resize(500, 400)
        self.init_ui()

    def init_ui(self):
        # Merkez widget ve ana dikey layout
        merkez = QWidget()
        self.setCentralWidget(merkez)
        ana_layout = QVBoxLayout(merkez)

        # ---- Ad Soyad alanı ----
        ad_soyad_layout = QHBoxLayout()
        ad_soyad_layout.addWidget(QLabel("Ad Soyad:"))
        self.ad_soyad_input = QLineEdit()
        ad_soyad_layout.addWidget(self.ad_soyad_input)
        ana_layout.addLayout(ad_soyad_layout)

        # ---- Cinsiyet seçimi (QRadioButton) ----
        cinsiyet_grup = QGroupBox("Cinsiyet")
        cinsiyet_layout = QHBoxLayout()
        self.erkek_rb = QRadioButton("Erkek")
        self.kadin_rb = QRadioButton("Kadın")
        self.erkek_rb.setChecked(True)  # varsayılan
        cinsiyet_layout.addWidget(self.erkek_rb)
        cinsiyet_layout.addWidget(self.kadin_rb)
        cinsiyet_grup.setLayout(cinsiyet_layout)
        ana_layout.addWidget(cinsiyet_grup)

        # ---- Ülke seçimi (QComboBox) ----
        ulke_layout = QHBoxLayout()
        ulke_layout.addWidget(QLabel("Ülke:"))
        self.ulke_combo = QComboBox()
        self.ulke_combo.addItems(["Türkiye", "Almanya", "Fransa", "İtalya", "İspanya"])
        ulke_layout.addWidget(self.ulke_combo)
        ana_layout.addLayout(ulke_layout)

        # ---- Not alanı (QTextEdit) ----
        not_layout = QHBoxLayout()
        not_layout.addWidget(QLabel("Not:"))
        self.not_input = QTextEdit()
        self.not_input.setFixedHeight(80)
        not_layout.addWidget(self.not_input)
        ana_layout.addLayout(not_layout)

        # ---- Yaş seçimi (QSpinBox) ----
        yas_layout = QHBoxLayout()
        yas_layout.addWidget(QLabel("Yaş:"))
        self.yas_spin = QSpinBox()
        self.yas_spin.setRange(0, 120)
        self.yas_spin.setValue(25)
        yas_layout.addWidget(self.yas_spin)
        ana_layout.addLayout(yas_layout)

        # ---- Kaydet butonu ----
        self.kaydet_btn = QPushButton("Kaydet")
        self.kaydet_btn.clicked.connect(self.formu_kaydet)
        ana_layout.addWidget(self.kaydet_btn)

    def formu_kaydet(self):
        # Form verilerini oku
        ad_soyad = self.ad_soyad_input.text().strip()
        cinsiyet = "Erkek" if self.erkek_rb.isChecked() else "Kadın"
        ulke = self.ulke_combo.currentText()
        not_text = self.not_input.toPlainText().strip()
        yas = self.yas_spin.value()

        # Basit doğrulama: zorunlu alanlar
        if not ad_soyad:
            QMessageBox.warning(self, "Eksik Bilgi", "Ad Soyad alanı boş bırakılamaz!")
            return
        if not ulke:
            QMessageBox.warning(self, "Eksik Bilgi", "Ülke seçimi yapmalısınız!")
            return

        # Form özetini konsola yazdır
        print("--- Form Bilgileri ---")
        print(f"Ad Soyad: {ad_soyad}")
        print(f"Cinsiyet: {cinsiyet}")
        print(f"Ülke: {ulke}")
        print(f"Yaş: {yas}")
        print(f"Not: {not_text}")
        print("----------------------")

        # Bilgilendirme penceresi
        QMessageBox.information(
            self,
            "Başarılı",
            f"Form bilgileri alındı:\nAd Soyad: {ad_soyad}\nÜlke: {ulke}\nYaş: {yas}"
        )
```

#### Kod Analizi

- **İçe aktarmeler:** PySide6'nın gerekli widget sınıflarını alırız; `QLineEdit`, `QRadioButton`, `QComboBox`, `QTextEdit`, `QSpinBox` ve layout yöneticileri.
- **AnaPencere sınıfı:** `QMainWindow`'dan miras alır; pencereyi kurmak için `init_ui()` metodu çağırılır.
- **`init_ui()` metodu:** 
  - Merkez widget (`QWidget`) ve dikey bir `QVBoxLayout` oluşturur.
  - Her bir form alanı için yatay layout (`QHBoxLayout`) kullanılır; etiket (`QLabel`) ve giriş widget'ı yan yana eklenir.
  - Cinsiyet için `QGroupBox` içinde iki `QRadioButton` yer alır; varsayılan olarak "Erkek" işaretli.
  - Ülke seçimi için `QComboBox` kullanılır; örnek bir ülke listesi eklenir.
  - Not alanı için çok satırlı `QTextEdit` yüksekliği 80 piksele sabitlenir.
  - Yaş seçimi için `QSpinBox` kullanılır; minimum 0, maksimum 120, varsayılan 25.
  - "Kaydet" butonu oluşturulur ve `clicked` sinyali `formu_kaydet` slotuna bağlanır.
- **`formu_kaydet()` metodu:** 
  - Form elemanlarından metinleri alır (`text()`, `currentText()`, `toPlainText()`, `value()`).
  - `strip()` ile baş ve sondaki bozukluklar temizlenir.
  - Zorunlu alanlar (`ad_soyad`, `ulke`) boş ise `QMessageBox.warning` ile kullanıcı uyarılır ve fonksiyon erken dönüş yapar.
  - Geçerli veri varsa konsola neatly biçimlendirilen özet yazdırılır.
  - Başarılı girişte `QMessageBox.information` ile onay mesajı gösterilir.

#### Görsel / İşlevsel Sonuç

```
Ekranda "Kullanıcı Formu" başlıklı bir pencere görünür.
Pencerenin içinde:
- Üstte "Ad Soyad:" etiketi ve tek satırlık giriş kutusu (QLineEdit)
- Ardından "Cinsiyet:" grubu içinde iki radyo düğmesi (Erkek, Kadın)
- Sonra "Ülke:" etiketi ve açılır liste (QComboBox)
- Daha sonra "Not:" etiketi ve çok satırlık metin kutusu (QTextEdit)
- Son olarak "Yaş:" etiketi ve artırma/azaltma düğmeleri olan sayaç kutusu (QSpinBox)
- En altta "Kaydet" butonu bulunur.

Kullanıcı Ad Soyad girer, bir cinsiyet seçer, bir ülke seçer, isteğe bağlı not yazar ve yaşı ayarlar.
"Kaydet" butonuna tıklandığında:
- Eğer Ad Soyad veya Ülke boş bırakılmışsa, uyarı penceresi gösterilir.
- Aksi takdirde, girilen bilgiler konsola şu şekilde basılır:
  --- Form Bilgileri ---
  Ad Soyad: Ahmed Yılmaz
  Cinsiyet: Kadın
  Ülke: Türkiye
  Yaş: 30
  Not: Bu bir test notudur.
  ----------------------
- Ayrıca bilgilendirme penceresi ile girişin alındığına dair onay verilir.
```

---

### Adım 2: Formdaki verileri okuyan bir "Kaydet" butonu ve okuma fonksiyonu yazma

Kullanıcı tarafından doldurulan form verilerini toklamak ve işlemek için bir buton ve bu butona bağlı bir slot (fonksiyon) gereklidir. Daha önceki kodda zaten `self.kaydet_btn` butonu ve `self.formu_kaydet` metodu oluşturulmuştur; bu bölümde bu mekanizmanın işleyişini daha net bir şekilde açıklıyoruz.

Butonun `clicked` sinyali, kullanıcı farla butona bastığında tetiklenir. Bu sinyal, `self.formu_kaydet` metodunu çalıştırır; metod içindeki kodlar form elemanlarındaki mevcut değerleri okur, doğrular ve gerekli işlemleri yapar. Böylece arayüz ile mantık ayrı ayrı yönetilir, kodun okunabilirliği artar.

> [!TIP]
> Slot fonksiyonları (`self.formu_kaydet` gibi) genellikle `self.` ön ekiyle tanımlanır; böylece sınıfın diğer metotları ve özelliklerine (örneğin `self.ad_soyad_input`) kolayca erişebiliriz.

> [!WARNING]
> Slot fonksiyonu içinde uzun süren işlemler yapmamalıdır; arayüzü donar (bloklar). Uzun işler gerekirse `QThread` kullanılarak arka plana alınmalıdır.

#### Doğrulama Mantığı

`formu_kaydet` metodunda gördüğümüz gibi, `if not ad_soyad:` kontrolü boş bırakılamayan alani yakalar. Benzer şekilde `if not ulke:` kontrolü de zorunlu bir seçim yapmadığı durumu yakalar. Bu tür kontroller, kullanıcıya geri bildirim vermek için `QMessageBox.warning` veya `QMessageBox.critical` ile gösterilebilir.

> [!NOTE]
> Gerçek uygulamalarda e-posta doğrulama gibi daha karmaşık kontroller için düzenli ifadeler (`re` modülü) kullanılabilir; ancak bu ders seviyesinde temel boşluk ve seçim kontrolleri yeterlidir.

---

### Adım 3: QLineEdit boş bırakıldığında veya seçim yapılmadığında kullanıcıya uyarı gösteren doğrulama mantığı ekleme

Bir form doldururken zorunlu alanları boş bırakmaya çalıştığımızda, genellikle formun altına kırmızı bir mesaj pojawır veya bir uyarı penceresi çıkar. PySide6'te bu uyarı mekanizması `QMessageBox` ile taklit edilir.

Kullanıcı `Ad Soyad` kutusuna hiçbir şey yazmazsa veya `Ülke` açılır listesinde bir seçenek seçmezse, `QMessageBox.warning` ile bilgilendirici bir uyarı penceresi gösterilir ve işlem durdurulur (`return`). Böylece eksik verinin işlenmesi önlenir.

> [!IMPORTANT]
> Uyarı mesajlarının metni kısa ve net olmalı; kullanıcı neyin eksik olduğunu anlayabilmelidir. Örneğin "Ad Soyad alanı boş bırakılamaz!" gibi bir mesaj hem tanımlayıcı hem de yönlendiricidir.

## 3. Sıra Sende (Pekiştirme Görevi)

- **Görev:** Forma `QSpinBox` (Yaş seçimi) ekleyip gelen veriyi form özetine dahil etme. Yaş değerini 0‑120 arasında tutacak şekilde sınırlayıp, yaş kutusunun varsayılan değerini 18 yapın.
- **İpucu:** `self.yas_spin.setRange(0, 120)` ve `self.yas_spin.setValue(18)` satırlarını `init_ui()` metodunun `yas_layout` kısmına ekleyin.
- **Beklenen Çıktı:** Kodu çalıştırdığınızda, yaş kutusunun minimale 0, maksimum 120 ve başlangıç değeri 18 olan bir `QSpinBox` görürünüz. Yaşı değiştirip "Kaydet" butonuna bastığınızda, konsol çıktısında yaş değeri doğru şekilde görünmelidir.

> [!TIP]
> Yaş kutusunun değerini `self.yas_spin.value()` şeklinde alarak form özetine ekleyebilirsiniz; bu zaten `formu_kaydet` metodunda yapılmıştır.