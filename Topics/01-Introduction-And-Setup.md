# Giriş ve Çevre Kurulumu

## 🎯 Bu Derste Ne Öğreneceğiz?

Bu dersin sonunda aşağıdaki yetkinliklere sahip olacaksınız:

- Grafiksel Kullanıcı Arayüzü (GUI) ile Komut Satırı Arayüzü (CLI) arasındaki temel farkları açıklayabilmek.
- Python için izole bir geliştirme ortamı (virtual environment) oluşturabilmek.
- PySide6 kütüphanesini başarıyla kurmak.
- PySide6 kullanarak ilk boş pencerenizi oluşturup ekranda gösterebilmek.

## 💡 Neden Bu Konuyu Öğreniyoruz?

Çoğu modern yazılım, kullanıcıların karmaşık komutlar yazması yerine butonlar, menüler ve pencereler aracılığıyla etkileşime girmesini sağlar. Bu etkileşim biçimine **Grafiksel Kullanıcı Arayüzü (GUI)** denir. Python ve PySide6 kullanarak, sadece kod yazan değil, gerçek kullanıcıların kullanabileceği profesyonel masaüstü uygulamaları geliştirmeyi öğreneceksiniz.

## 🔍 Birlikte Keşfedelim

Bir bilgisayar programıyla etkileşime geçmenin iki ana yolu vardır:

1.  **Komut Satırı Arayüzü (CLI - Command Line Interface):** Programla sadece metin yazarak ve komutlar göndererek konuşursunuz. Örneğin, bir dosyayı silmek için terminale `rm dosya.txt` yazarsınız. Hızlıdır ancak yeni başlayanlar için öğrenmesi zordur.
2.  **Grafiksel Kullanıcı Arayüzü (GUI - Graphical User Interface):** Programla görsel öğeler (butonlar, ikonlar, pencereler) aracılığıyla konuşursunuz. Bir dosyayı silmek için çöp kutusu ikonuna tıklarsınız. Daha sezgiseldir ve görsel geri bildirim sağlar.

> [!NOTE]
> Profesyonel bir yazılım geliştirici, ihtiyaca göre hem CLI hem de GUI araçlarını kullanmayı bilmelidir. Ancak "Görsel Programlama" dersinde biz, kullanıcıların en çok tercih ettiği GUI dünyasına odaklanacağız.

## 🛠️ Uygulama

### Adım 1: Geliştirme Ortamını Hazırlama

Her Python projesinde olduğu gibi, projelerimizin diğer sistem dosyalarından izole olması için bir **Sanal Ortam (virtual environment)** kullanmalıyız. Bu, kütüphane versiyonlarının birbirine karışmasını önler.

Terminalinizi (Linux kullanıyorsanız `zsh` veya `bash`) açın ve şu komutları sırayla uygulayın:

```bash
# Proje klasörüne gidin (eğer henüz orada değilseniz)
cd /home/abdullatif/Masaüstü/Pyside6

# Sanal ortamı oluşturun
python3 -m venv venv

# Sanal ortamı aktifleştirin
source venv/bin/activate
```

Aktifleştirme başarılı olduğunda, terminal satırınızın başında `(venv)` ibaresini görmelisiniz.

### Adım 2: PySide6 Kurulumu

Şimdi, arayüzlerimizi oluşturmamızı sağlayacak olan PySide6 kütüphanesini sanal ortamımıza kuralım:

```bash
pip install PySide6
```

### Adım 3: İlk PySide6 Penceresi

Şimdi VS Code'u açın ve `main.py` adında yeni bir dosya oluşturun. Aşağıdaki kodu bu dosyaya yapıştırın:

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget

def main():
    # 1. Uygulama nesnesini oluşturma
    app = QApplication(sys.argv)

    # 2. Bir pencere (widget) oluşturma
    pencere = QWidget()

    # 3. Pencereye özellikler ekleme
    pencere.setWindowTitle("Merhaba PySide6!")
    pencere.resize(400, 300)

    # 4. Pencereyi ekranda gösterme
    pencere.show()

    # 5. Uygulama döngüsünü başlatma
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
```

**Kodun Açıklaması:**
- `QApplication`: Her PySide6 uygulamasının en az bir tane olması gereken, uygulamanın kontrolünü ve olay döngüsünü (event loop) yöneten ana nesnedir.
- `QWidget`: En temel görsel bileşendir. Bir pencere veya bir buton aslında bir widget'tır.
- `app.exec()`: Uygulamanın kapanana kadar sürekli çalışmasını (kullanıcı bir butona basana kadar beklemesini) sağlar.

**Kodu Çalıştırın:**
Terminalde şu komutu verin:
```bash
python main.py
```

## 🧪 Deneyelim ve Gözlemleyelim

Kodunuz çalıştıktan sonra şunları yaparak uygulamanın nasıl tepki verdiğini gözlemleyin:

1.  `pencere.resize(400, 300)` satırındaki sayıları değiştirin (örneğin `800, 600`) ve programı tekrar çalıştırın. Pencere boyutu değişti mi?
2.  `pencere.setWindowTitle(...)` kısmındaki metni değiştirin. Pencerenin üst kısmındaki yazı güncellendi mi?
3.  `pencere.show()` satırını yorum satırı yapın (`# pencere.show()`) ve tekrar çalıştırın. Bir şey oluyor mu? (İpucu: Program çalışıyor gibi görünecek ama ekranda hiçbir şey göremeyeceksiniz!)

## 🧠 Kendini Test Et

1. **Soru:** `QApplication` nesnesi olmadan bir PySide6 uygulaması çalışabilir mi?
2. **Soru:** `QWidget` sınıfı ile `QMainWindow` arasındaki temel fark nedir? (İpucu: Bir sonraki derse göz atın!)

<details>
<summary>Cevapları göster</summary>

1. **Cevap:** Hayır, her PySide6 uygulaması mutlaka bir `QApplication` nesnesine ihtiyaç duyar; çünkü uygulamanın tüm yönetimini o sağlar.
2. **Cevap:** `QWidget` en temel, boş bir bileşendir. `QMainWindow` ise menü çubukları, araç çubukları ve durum çubuğu gibi özelliklere sahip, "ana pencere" olarak kullanılmaya hazır daha gelişmiş bir yapıdır.

</details>

## 🚀 Kendin Dene

**Görev:**
Öğrendiklerinizi kullanarak aşağıdaki kriterlere uyan bir Python dosyası oluşturun:
1. Pencere boyutu **800x600** piksel olsun.
2. Pencerenin başlığı (title) **"[Adınız Soyadınız] - İlk Uygulamam"** şeklinde olsun.
3. Kodu çalıştırdığınızda pencere ekranda başarıyla görünmelidir.

## 📚 Daha Fazlasını Öğrenmek

- [PySide6 Resmi Dökümantasyonu](https://doc.qt.io/qtforpython-6/) (İngilizce)

## 📝 Kısa Özet

- **GUI**, kullanıcı etkileşimi için görsel öğeler sunar.
- **venv**, kütüphanelerin projeye özel kalmasını sağlar.
- **QApplication**, uygulamanın beynidir.
- **QWidget**, görsel bileşenlerin temel yapı taşıdır.

---
[⬅️ Önceki Ders](Topics/00-None.md) | [Sonraki Ders ➡️](Topics/02-Oop-And-Structure.md)
