# Windows İstemciler İçin Görsel Performans & UX Standardizasyonu (GPO Rehberi)

| Özellik | Değer |
| :--- | :--- |
| **Doküman Kodu** | `GPO-SOP-2026-084` (v1.2) |
| **Politika Türü** | Group Policy Preferences (GPP) — Registry Settings |
| **Uygulama Alanı** | Domain Genelinde Kullanıcılar (`HKCU`) |
| **Etki Seviyesi** | Performans & Kullanıcı Arayüzü |
| **Hedef Ortam** | Windows 10 / 11 Pro & Enterprise, RDS / VDI |
| **Önerilen GPO Adı** | `GPO_USR_VisualPerformance_CustomStandard` |

---

## 1. Yönetici Özeti ve Politikanın Amacı

Kurumsal çalışma ortamlarında istemci uç nokta (endpoint) ve VDI/RDS terminal sunucularında donanım kaynak tüketimini düşürmek adına standart Windows ayarlarından **"En iyi performans için ayarla"** profili sıklıkla tercih edilmektedir. Ancak bu profil agresif bir şekilde devreye girdiğinde, günlük operasyonda çalışanların verimliliğini doğrudan etkileyen iki kritik kullanıcı arayüzü bileşenini zorla devre dışı bırakmaktadır:

1. Dosya simgeleri yerine görsel/doküman önizlemelerinin (**Thumbnails**) gizlenmesi.
2. Ekran yazı tipi kenar yumuşatmasının (**Font Smoothing / ClearType**) kapatılarak metinlerin pikselli ve okunaksız hale gelmesi.

Bu standart işletim prosedürü; arka plandaki tüm gereksiz pencere/menü animasyonlarını ve görsel ağırlıkları kapatırken, kullanıcı deneyimi ve doküman tespit hızı için elzem olan önizleme ve font netliğini garanti altına alan kurumsal standardı belirler.

---

## 2. Yapılandırılan Kayıt Defteri (Registry) Matrisi

| No | Kayıt Defteri Yolu (Hive & Key Path) | Değer Adı | Tür | Atanan Değer |
| :--- | :--- | :--- | :--- | :--- |
| **01** | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced` | `IconsOnly` | `REG_DWORD` | `0` |
| **02** | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects` | `VisualFXSetting` | `REG_DWORD` | `2` |
| **03** | `HKCU\Control Panel\Desktop` | `FontSmoothing` | `REG_SZ` | `2` |

---

## 3. Mimari Davranış ve Teknik Parametre Analizi

### 3.1. `IconsOnly` (`REG_DWORD = 0`) — Dosya Küçük Resim Önizlemeleri
Windows Gezgini'nin içerik oluşturma motorunu doğrudan kontrol eder. Değer `1` yapıldığında sistem görsel önizleme oluşturmayı tamamen durdurur ve genel dosya simgelerini basar. `0` değeri verilerek; ağ paylaşımlarında veya yerel disklerde yer alan tasarım dosyaları, ürün katalogları, teknik çizimler, PDF ve görsel belgelerin önizleme kareleri aktif tutulur. Bu yapılandırma, kullanıcıların ilgili dosyayı açmadan içeriği tanımasını sağlayarak dosya arama sürelerini minimize eder.

### 3.2. `VisualFXSetting` (`REG_DWORD = 2`) — Görsel Efekt Profil Modu
`Sistem Özellikleri > Performans Seçenekleri` altındaki profil seçicisini temsil eder. Windows mimarisinde bu anahtar 4 farklı duruma sahiptir:

* **Değer `0`:** Windows sizin için en iyisini seçsin (Varsayılan dinamik profil)
* **Değer `1`:** En iyi görünüm için ayarla (Tüm görsel efektler, gölgeler ve şeffaflıklar açık)
* **Değer `2`:** **Özel / Custom** (Kullanıcı veya GPO tarafından belirlenmiş özel görsel bayraklar)
* **Değer `3`:** En iyi performans için ayarla (Tüm efektler, önizlemeler ve font düzeltmeleri kapalı)

> **Mimari Çakışma Önleme Notu:** Eğer bu parametreye `3` (*En İyi Performans*) atanmış olsaydı, Windows oturum açılışında diğer iki ayar (`IconsOnly` ve `FontSmoothing`) aktif edilse bile Gezgin motoru bu ayarları ezecek ve kapatacaktır. Sistem genelinde görsel animasyon yükünü kısıtlarken belirli özellikleri açık bırakabilmenin işletim sistemi mimarisindeki tek yolu profili `2` (**Özel**) olarak sabitlemektir.

### 3.3. `FontSmoothing` (`REG_SZ = "2"`) — Ekran Yazı Tipi Kenar Yumuşatma
Windows ClearType ve alt piksel metin oluşturma motorunun (Sub-pixel font smoothing) anahtarıdır. Değer `0` veya `1` olduğunda işletim sistemi anti-aliasing mekanizmasını durdurur. Özellikle yüksek çözünürlüklü monitörlerde, ERP/muhasebe yazılımlarında, Excel tablolarında ve tarayıcılarda metin kenarları kırık piksellere dönüşür. Bu değerin `2` (`REG_SZ` / Dize Değeri) olarak yapılandırılması, donanımsal yük bindirmeden ekran okunabilirliğini ve çalışma konforunu korur.

---

## 4. GPO (Group Policy Preferences) Dağıtım Prosedürü

Bu yapılandırma kullanıcı kovanında (`HKCU`) yer aldığı için standart İdari Şablonlar yerine **Group Policy Preferences (GPP) Registry** yoluyla dağıtılmalıdır. Aşağıdaki adımlar Active Directory Domain Controller üzerinde izlenmelidir:

1. **`gpmc.msc` Konsolunu Açın:** Hedef Kullanıcı Organizasyon Birimi (OU) üzerine gelin ve yeni bir GPO oluşturup bağlayın.
2. **İlgili Menüye Gidin:** GPO'yu düzenleyin ve şu yolu izleyin:  
   `User Configuration > Preferences > Windows Settings > Registry`
3. **Yeni Kayıt Ekleyin:** Sağ tıklayıp **New > Registry Item** seçeneğini seçin.
4. **Parametreleri Tanımlayın:** Açılan pencerede aşağıdaki ayarları yapın:
   * **Action:** `Update` *(Anahtar yoksa oluşturur, varsa günceller; kullanıcı ayarı bozsa dahi tekrar düzeltir)*
   * **Hive:** `HKEY_CURRENT_USER`
   * **Key Path & Value Details:** Bölüm 2.0'daki matrise göre 3 anahtarı ayrı ayrı tanımlayın.
5. **İsteğe Bağlı Hedefleme (Item-Level Targeting):** RDS / Terminal Sunucuları için **Common** sekmesinden **Item-Level Targeting** kullanılarak politikanın yalnızca belirli sunucu gruplarında veya güvenlik gruplarında devreye girmesi sağlanabilir.

---

## 5. PowerShell ile Hızlı Kontrol Komutları

İstemci oturumunda kayıt defteri değerlerinin GPO üzerinden başarıyla uygulanıp uygulanmadığını doğrulamak için:

```powershell
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "IconsOnly"
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" -Name "VisualFXSetting"
Get-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "FontSmoothing"
```

---

## 6. Test, Kalibrasyon ve Doğrulama Kontrol Listesi

| Sıra | Kontrol Adımı / Metot | Beklenen Çıktı / Değer | Durum |
| :--- | :--- | :--- | :--- |
| **1** | `gpupdate /force` komutunun çalıştırılması | `User Policy update has completed successfully` mesajı | `[ ]` Doğrulandı |
| **2** | Oturumun yeniden başlatılması (`Logoff / Logon`) | Explorer kabuğunun yeni `HKCU` değerlerini belleğe alması | `[ ]` Doğrulandı |
| **3** | Kayıt defteri (`regedit.exe`) kontrolü | `IconsOnly=0`, `VisualFXSetting=2`, `FontSmoothing=2` teyidi | `[ ]` Doğrulandı |
| **4** | `sysdm.cpl` Arayüz doğrulaması | **Görsel Efektler** sekmesinde **Özel** radyo butonunun seçili olması | `[ ]` Doğrulandı |
