# Capstone Proje ve Dağıtım (PyInstaller)

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Modüler proje yapısını kurabilmek (ui/, database/, models/ klasörleri).
- PyInstaller kullanarak Python uygulamasını tek bir çalıştırılabilir dosyaya (.exe) dönüştürebilmek.
- `--onefile` ve `--windowed` bayraklarının ne yaptığını açıklayabilmek.
- Uygulamaya özel bir simge ekleyebilmek.
- Bağımlılıkları yapılandırarak dağıtım için hazır hale getirebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Geliştirdiğimiz uygulamayı başkalarının kullanabilmesi için, Python ve gerekli tüm kütüphanelerin yüklü olmayan bir sistemde çalıştırılabilir hale getirmemiz gerekir. PyInstaller, bu işlemi tek bir komutla yapmamızı sağlar. Böylece uygulama başkaları tarafından kurulmadan veya Python bilmeden kullanılabilir hale gelir.

## 🔍 Birlikte Keşfedelim

Bir Python betiğini (örneğin `main.py`) bir çalıştırılabilir dosyaya dönüştürmek, bir kitabı derleme işlemine benzer: tüm bağımlılıklar (kütüphane dosyaları, yorumlayıcı vb.) tek bir pakette toplanır. Bu sayede kullanıcı sadece bu tek dosyayı çalıştırarak uygulamanın tüm özelliklerinden yararlanabilir.

## 🛠️ Uygulama

Bu derste, daha önce geliştirdiğimiz uygulamalarımızı (veritabanı, arayüz, stil, diyalog vb.) birleştiren bir "Kişisel Not ve Görev Yöneticisi" capstone projesini modüler bir yapıyla oluşturacağız ve ardından PyInstaller ile tek bir çalıştırılabilir dosyaya paketleyeceğiz.

Not: Bu derste, capstone projesinin tamamını kodlayacağız; ancak kodun uzunluğu ve dersin akışı göz önüne alındığında, modüler yapının oluşturulması, ana dosyanın (main.py) hazırlanması ve PyInstaller komutunun çalıştırılması üzerine odaklanacağız. Modüllerin içeriği daha önceki derslerde öğrendiklerimizin bir kombinasyonu olacaktır.

### Adım 1: Proje Yapısını Oluşturma

Proje kök dizininde aşağıdaki klasör yapısını oluşturalım:

```
capstone_projesi/
├── main.py
├── ui/
│   ├── main_window.py
│   └── styles.qss
├── database/
│   └── db_manager.py
├── models/
│   └── task_model.py
└── requirements.txt
```

### Adım 2: Modülleri Hazırlama

Her modülü daha önceki derslerdeki bilgilerle dolduracağız. Örneğin:

- **`ui/main_window.py`**: Ana pencere sınıfı (QMainWindow), menüler, araç çubukları, durum çubuğu, form elemanları, liste widget'ları, sinyal-slot bağlantıları.
- **`ui/styles.qss`**: Uygulama stilleri (QSS).
- **`database/db_manager.py`**: SQLite veritabanı bağlantısı ve CRUD işlemleri.
- **`models/task_model.py`**: Görev veri modeli (sınıf).
- **`main.py`**: Uygulamanın giriş noktası. QApplication örneği oluşturur, ana pencereyi gösterir ve eventi başlatır.

### Adım 3: PyInstaller ile Paketleme

1.  Sanal ortamı aktifleştirin ve proje dizinine gidin.
2.  Gereksinimleri kaydedin (eğer yapmadıysanız): `pip freeze > requirements.txt`
3.  PyInstaller'ı yükleyin (eğer yüklü değilse): `pip install pyinstaller`
4.  Tek bir çalıştırılabilir dosya oluşturun:
    ```
    pyinstaller --onefile --windowed --icon=app.ico main.py
    ```
    - `--onefile`: Tüm dosyaları tek bir yürütülebilir dosyada toplar.
    - `--windowed`: Konsol penceresini göstermez (GUI uygulamaları için).
    - `--icon=app.ico`: Uygulamaya bir simge ekler (isteğe bağlı).
5.  Oluşan `dist/` klasörü içinde `main.exe` (Windows) veya `main` (Linux/macOS) dosyasını bulun. Bu dosyayı başka bir Python kurulu olmayan bir makinede test edin.

## 🧪 Deneyelim ve Gözlemleyelim

1.  `--onefile` bayrağını kaldırıp yeniden derleyin. Ne olur? (Çıktı: Birçok dosya ve bir `main` çalıştırılabilir dosyası oluşturur.)
2.  `--windowed` bayrağını kaldırın. Ne olur? (Çıktı: Uygulama çalıştırıldığında arka planda bir konsol penceresi de açılır.)
3.  Simge dosyası yolunu yanlış yazın. Ne olur? (Çıktı: PyInstaller bir uyarı verir ama simgesiz bir exe üretir.)

## 🧠 Kendini Test Et
1. **Soru:** `--onefile` ve `--windowed` bayraklarının ne işe yaradığını açıklayın.
2. **Soru:** PyInstaller ile oluşturulan çalıştırılabilir dosyayı başka bir bilgisayarda test ederken "Python not found" hatası alıyorum. Ne yapmalıyım?

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** `--onefile` tüm bağımlılıkları ve yorumlayıcısını tek bir dosyada toplar, `--windowed` ise GUI uygulamalarında konsol penceresinin görünmesini engeller.

2. **Cevap:** PyInstaller, Python yorumlayıcısını ve tüm gerekli kütüphaneleri çalıştırılabilir dosyasının içine yerleştirir. Bu hatayı alıyorsanız, olası iki durum vardır: a) `--onefile` kullanmadınız ve bağımlılıkları yanlış yerleştirdiniz, b) antivirüs yazılımı çalıştırılabilir dosyayı engelledi. Güvenli bir kaynaktan indirdiğinizden ve gerekli izinlere sahip olduğunuzdan emin olun.

</details>

## 🚀 Kendin Dene

**Görev:**
1.  Bu derslerde geliştirdiğiniz bir uygulamayı (örneğin Hafta 10-11-12'deki not ve görev yöneticisi) modüler bir yapıya dönüştürün.
2.  `main.py` dosyasını hazırlayın.
3.  Projenizi PyInstaller ile `--onefile` ve `--windowed` bayrakları kullanarak paketleyin.
4.  Oluşan çalıştırılabilir dosyayı çalıştırarak uygulamanın düzgün çalıştığından emin olun.
5.  İsteğe bağlı olarak, bir `.ico` simgesi ekleyin ve `--icon` parametresi ile uygulamanıza simge verin.

## 📚 Daha Fazlasını Öğrenmek

- [PyInstaller Resmi Dokümantasyonu](https://pyinstaller.org/en/stable/)
- [PyInstaller Kullanım Kılavuzu](https://pyinstaller.org/en/stable/usage.html)

## 📝 Kısa Özet

- **Modüler Mimari:** Kodunuzu mantıksal klasörlere (ui, database, models) bölerak yeniden kullanılabilirliği ve okunabilirliği artırır.
- **PyInstaller:** Python uygulamalarını bağımsız çalıştırılabilir dosyalara dönüştürür.
- **Tek Dosya (`--onefile`):** Dağıtımı tek bir dosya ile yapmayı sağlar.
- **Pencere Modu (`--windowed`):** Konsol penceresini gizleyerek sadece GUI gösterir.
- **Simge (`--icon`):** Uygulamaya profesyonel bir görünüm kazandırır.
- **Dağıtım:** Oluşan `dist/` klasöründeki çalıştırılabilir dosya, Python kurulu olmayan sistemlerde çalıştırılabilir.

---
[⬅️ Önceki Ders](12-Error-Handling-Ux.md) | [Ana Sayfa ➡️](../README.md)
