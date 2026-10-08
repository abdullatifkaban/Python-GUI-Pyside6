# 01 - Giriş ve Çevre Kurulumu

## 0. Çalışma Ortamının (VS Code) Hazırlanması

Öğrenme sürecinizi sorunsuz bir şekilde başlatmak için VS Code ortamını adım adım hazırlayacağız:

### VS Code ve Python Eklentisinin Yüklenmesi

1. **VS Code İndirme ve Kurulumu**
   - https://code.visualstudio.com/ adresinden işletim sisteminize uygun sürümü indirin.
   - İndirilen dosyayı çalıştırarak VS Code'u kurun.

2. **Python Eklentisinin Ekleme**
   - VS Code'u açın, sol taraftaki **Eklentiler** (Extensions) simgesine tıklayın (Ctrl+Shift+X).
   - Arama kutusuna "**Python**" yazın, Microsoft tarafından yayınlanan eklentiyi seçip **"Yükle"** butonuna basın.
   - Yükleme tamamlandıktan sonra VS Code'u yeniden başlatmanız önerilir (istemediğinde de genellikle sorunsuz çalışır).

### Proje Klasörünün Açılması ve main.py Dosyası Oluşturma

1. **Proje Klasörünü VS Code'da Açma**
   - Menüden **Dosya > Klasör Aç** (File > Open Folder) seçeneğine gidin.
   - Proje klasörünüzü (örneğin `Python-GUI-Pyside6`) seçip **"Klasör Seç"** butonuna tıklayın.
   - VS Code sol gezgininde proje yapınız görünecektir.

2. **main.py Dosyasının Oluşturulması**
   - Sol gezgin içinde proje klasörüne sağ tıklayın → **Yeni Dosya** (New File).
   - Dosya adını `main.py` olarak girin ve Enter tuşuna basın.
   - Boş bir `main.py` dosyası oluşturulur ve düzenleyicide açılır.

### VS Code Entegre Terminalini Açma

- Klavyenizde **Ctrl + `** (backtick) tuşlarına basın veya menüden **Terminal > Yeni Terminal** (Terminal > New Terminal) seçeneğine gidin.
- Alt panelete bir terminal penceri açılır; buradan komutlar çalıştırabilirsiniz.

### Sanal Ortamın (venv) VS Code Python Yorumlayıcısı Olarak Seçilmesi

> [!IMPORTANT]
> Sanal ortamı (venv) henüz oluşturmadıysanız, önce terminal üzerinden oluşturmalı ve aktif etmelisiniz.

1. **Sanal Ortam Oluşturma ve Aktifleştirme (Terminal üzerinden)**
   ```bash
   # Proje klasörünün içindeyken:
   python -m venv venv
   # Linux/macOS:
   source venv/bin/activate
   # Windows:
   venv\Scripts\activate
   ```

2. **VS Code'da Yorumlayıcıyı Seçme**
   - Alt çubukta (Status Bar) sağda bulunan Python sürümü gösterimi (örneğin "Python 3.12.0")na tıklayın.
   - Açılan paletten "**Python Yorumlayıcı Seç**" (Select Python Interpreter) komutunu çalıştırın.
   - Listelenen seçeneklerden, proje klasörünüz içindeki `venv` klasörü altında bulunan yorumlayıcıyı seçin (örneğin "./venv/bin/python").
   - Seçtikten sonra alt çubukta artık bu yorumlayıcı aktif olarak görünür.

> [!TIP]
> Eğer henüz sanal ortam oluşturmadıysanız, VS Code terminalinden `python -m venv venv` komutunu çalıştırabilir, ardından yukarıdaki adımlarla aktif edip yorumlayıcı olarak seçebilirsiniz.

Artık VS Code ortamınız hazır! `main.py` dosyasına kod yazmaya başlayabilir, terminal üzerinden `python main.py` komutuyla çalıştırabilirsiniz.

---

## 1. PySide6 ile Görsel Programlamanın Temeli

### GUI Nedir? CLI'dan Farkı Nedir?

Komut satırında `print()` komutlarını çalıştırdığında, kullanıcının gördüğü tek şey metin tabanı bir çıktıdır. GUI (Grafiksel Kullanıcı Arayüzü) ise, pencereler, butonlar ve metin kutuları gibi **görsel bileşenler**den oluşur.

> [!TIP]
> **CLI** → garsona veriri yazıkla, **GUI** → menüde oturup kendin seçersin. CLI metinle konuşur; GUI ise görsel öğelerle etkileşir.

| CLI | GUI |
|-----|-----|
| Metin komutlarıyla çalışır (`cd`, `python app.py`) | Fare tıklamaları ve klavye girişleriyle çalışır |
| Çıktı yalın metin | Pencereler, menüler, butonlar |
| Başlangıç eğrisi düşük | Başlangıç eğrisi biraz daha yüksek |
| Otomasyon için ideal | Kullanıcı dostu, görsel geri bildirim sunar |

### Qt Framework ve PySide6

**Qt**, C++ ile geliştirilmiş, milyonlarca profesyonel masaüstü uygulamanın temelini oluşturan bir çerçevedir. **PySide6**, Qt'nin resmi Python bağlayıcısıdır — yani C++ kodunu Python sözdizimine çevirmiş bir arayüzdür.

> [!NOTE]
> PySide6, Qt Company (eski The Qt Company) tarafından resmi olarak desteklenir. PyQt6 ise üçüncü taraf bir proje olarak geliştirilir. İkisi de hemen hemen aynı widget'ları sunar, ancak PySide6'in Qt altındaki resmi lisansı daha esnektir.

### Python Sanal Ortamı — Neden ve Nasıl Kullanılır?

Python projelerinde, tüm paketleri sistem Python'unun içine kurmak yerine, projeye özel bir **sanal ortam** oluşturmak daha güvenli ve temizdir:

```bash
# 1. Sanal ortam oluşturma
python -m venv proje_ortami

# 2. Sanal ortamı aktifleştirme (Linux/macOS)
source proje_ortami/bin/activate

# 3. PySide6'yı kurma
pip install PySide6
```

> [!IMPORTANT]
> **Linux/macOS** için aktifleştirme: `source proje_ortami/bin/activate`.  
> **Windows** için aktifleştirme: `proje_ortami\Scripts\activate`.

### Kritik Sınıflar ve Metotlar

Bu bölümde kullanacağımız temel PySide6 sınıfları ve metotları:

| Sınıf / Metot | Açıklama |
|---------------|----------|
| `QApplication(sys.argv)` | Uygulamanın kalbi; pencere döngüsünü başlatır |
| `QWidget()` | Boş bir pencere (window) oluşturur |
| `.resize(w, h)` | Pencerenin piksel cinsinden boyutunu ayarlar |
| `.setWindowTitle("...")` | Pencerenin başlığındaki yazıyı değiştirir |
| `.show()` | Pencereyi ekranda görünür hale getirir |
| `.move(x, y)` | Pencerenin ekrandaki konumunu piksel bazlı ayarlar |
| `QScreen().availableGeometry()` | Kullanıcının ekranı için kullanılabilir alanı verir |

---

## 2. Adım Adım Kod Uygulaması

> **Senaryo:** Basit bir masaüstü penceresi oluşturmayacağız; önce uygulama döngüsünü kuruyoruz, sonra pencereyi boyutlandırıp başlık veriyor, en son ekran ortasında gösteriyoruz.

---

### Adım 1: QApplication Nesnesini Başlatma

#### Eklenecek Kod

```python
import sys
from PySide6.QtWidgets import QApplication

# 1) QApplication → uygulamanın kalbi, olay döngüsünü kurar
app = QApplication(sys.argv)

# 2) app.exec() → pencere kapatılana kadar döngüyü çalıştırır
# 3) sys.exit() → uygulama kapandığında doğru çıkış kodunu verir
sys.exit(app.exec())
```

#### Kod Analizi

- `import sys` → Python'un sistem modülünü alır; `sys.argv` ile komut satırı argümanlarına erişmemizi sağlar.
- `from PySide6.QtWidgets import QApplication` → Qt'nin görsel bileşenlerinden `QApplication` sınıfını getirir.
- `app = QApplication(sys.argv)` → Uygulama nesnesini oluşturur. Bu nesne olmadan, Qt hiçbir pencereyi yönetemez.
- `app.exec()` → **Olay döngüsünü (event loop)** başlatır. Yani pencere kapatılana kadar program burada kalır, kullanıcının tıklamaları, klavye girişleri gibi olayları bekler.
- `sys.exit(app.exec())` → Uygulama kapandığında çıkış kodunu terminalin döndürür.

#### Görsel / İşlevsel Çıktı

```
Terminalde hiçbir şey görünmez — sadece program çalışır, hiçbir pencere açılmaz.
Çünkü henüz bir QWidget oluşturmadık; sadece uygulama döngüsü kuruldu.
```

---

### Adım 2: Boş Pencere Oluşturma, Boyutlandırma ve Başlık Atama

#### Eklenecek Kod

```python
from PySide6.QtWidgets import QWidget

# QWidget → PySide6'nın her görsel pencerenin temeli.
pencere = QWidget()

# resize(600, 400) → pencerenin genişliğini 600, yüksekliğini 400 piksel yapar.
pencere.resize(600, 400)

# setWindowTitle → pencerenin üst çubuğunda görünen başlığı değiştirir.
pencere.setWindowTitle("Merhaba PySide6")
```

#### Kod Analizi

- `from PySide6.QtWidgets import QWidget` → `QWidget` sınıfını içeri aktarır.
- `pencere = QWidget()` → Görsel bir pencere nesnesi oluşturur; ama henüz ekranda görünmez.
- `pencere.resize(600, 400)` → Pencerenin boyutunu 600×400 piksel olarak ayarlar.
- `pencere.setWindowTitle("Merhaba PySide6")` → Pencerenin başlığını değiştirir.

#### Görsel / İşlevsel Çıktı

```
Halen ekranda bir şey görünmez — .show() çağrılmadı.
Ancak arka planda pencere hazır:
  • Boyutu: 600 × 400 piksel
  • Başlığı: "Merhaba PySide6"
```

---

### Adım 3: Pencereyi Gösterme ve Ekran Ortalayarak Konumlandırma

#### Eklenecek Kod

```python
from PySide6.QtGui import QScreen

# show() → pencerenizi artık ekranda görür!
pencere.show()

# Ekran ortalaması:
# 1) QApplication.primaryScreen() ile aktif ekran nesnesini alırız
# 2) availableGeometry() ile ekranın kullanılabilir kısmının geometrisini alırız
# 3) center() ile bu geometrinin merkez noktasını buluruz
# 4) Pencerenin sol üst köşesini, merkez noktanın X/Y'sinden
#    penceremizin genişliğinin / 2'sini çıkarıp yerleştiririz
ekran = QApplication.primaryScreen()
ekran_geometrisi = ekran.availableGeometry()
pencere.move(
    ekran_geometrisi.center().x() - pencere.width() // 2,
    ekran_geometrisi.center().y() - pencere.height() // 2
)
```

#### Kod Analizi

- `pencere.show()` → **Bu, pencerenizi ekranda görünür kılan tek komuttur.** Hiçbir şey yapılmasa da pencere hiç görünmez.
- `app.primaryScreen()` → QApplication nesnesi aracılığıyla aktif (monitörde ön plan) ekran nesnesini döndürür.
- `ekran.availableGeometry()` → Bu ekranın işletim sisteminin ayırdığı kullanılabilir alanı (görev çubuğu, farepanosu vb. dışarıda kalan alanlar hariic) verir.
- `ekran_geometrisi.center()` → Bu alanın tam ortasındaki noktayı `(x, y)` koordinatları cinsinden döndürür.
- `pencere.move(...)` → Pencerenizi, merkez noktanın sol/yukarıya kaydırılmış hâline gönderir; böylece pencere ekranın tam ortasında durur.

#### Görsel / İşlevsel Çıktı

```
Ekranda 600 × 400 piksellik bir pencere belirir:
  • Başlığı: "Merhaba PySide6"
  • Konumu: Monitörün tam ortası
  • İçeriği: Boş (boş QWidget)
```

---

### Nihai Tam Kod Bloğu

Öğrencinin kopyalayıp doğrudan çalıştırabileceği tek dosyalık kod:

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget

# Adım 1: QApplication ile uygulama döngüsü kurulur
app = QApplication(sys.argv)

# Adım 2: Boş pencere, boyut ve başlık
pencere = QWidget()
pencere.resize(600, 400)
pencere.setWindowTitle("Merhaba PySide6")

# Adım 3: Göster ve ekranın ortasına yerleştir
pencere.show()

# Ekran ortalaması:
# 1) QApplication.primaryScreen() ile aktif ekran nesnesini alırız
# 2) availableGeometry() ile ekranın kullanılabilir kısmının geometrisini alırız
# 3) center() ile bu geometrinin merkez noktasını buluruz
# 4) Pencerenin sol üst köşesini, merkez noktanın X/Y'sinden
#    penceremizin genişliğinin / 2'sini çıkarıp yerleştiririz
ekran = app.primaryScreen()
ekran_geometrisi = ekran.availableGeometry()
pencere.move(
    ekran_geometrisi.center().x() - pencere.width() // 2,
    ekran_geometrisi.center().y() - pencere.height() // 2
)

# Döngüyü başlat
sys.exit(app.exec())
```

> [!WARNING]
> `QScreen` sınıfı **boş kurucu ile oluşturulamaz** (`QScreen()` çalışmaz!).
> Ekran nesnesini **`app.primaryScreen()`** (veya pencere için `pencere.screen()`) ile almalısınız.
> Ayrıca `QScreen` sınıfı, **PySide6.QtGui** modülünden gelir; burada ise ihtiyacımız olan `QApplication`'a zaten `app` üzerinden eriştiğimiz için ayrı import gerekmez.

---

## 3. Bireysel Öğrenme Görevi (Sıra Sende)

> **Görev:** Pencerenin boyutunu 800 × 600 yapın ve başlığına kendi adınızı ekleyin (örneğin; `"Ahmet'in PySide6 Penceresi"`).

- **Ipucu:** `resize()` ve `setWindowTitle()` metodlarının içindeki argümanları değiştirin.
- **Beklenen sonuç:** Kodu çalıştırdığınızda, ekranın ortasında 800 × 600 piksel boyutunda, başlığında kendi adınızın yazdığı bir pencere belirir.

Başarıyla tamamladığınızda, PySide6 ile **uygulama döngüsü**, **pencere nesnesi** ve **ekran ortalaması** gibi temel kavramları kavradığınızdan emin olun. Bir sonraki konuda, OOP prensipleriyle bu pencereyi daha profesyonel bir yapıya sokacağız!
