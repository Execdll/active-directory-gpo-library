# 04 - Windows Server Güncelleştirme ve Çalışma Saatleri (Active Hours) Standardı

Kurumsal Active Directory ortamlarında yer alan Windows Server işletim sistemlerinin, mesai saatleri içerisinde Windows Update kaynaklı otomatik olarak yeniden başlatılmasını önlemek ve güncelleştirme davranışlarını standartlaştırmak amacıyla hazırlanan **Group Policy Object (GPO)** uygulama rehberi.

- **Önerilen GPO Adı:** `Computer_Servers_Windows_Update_Active_Hours`
- **Kapsam:** Bilgisayar (`HKLM` / `Computer Configuration`)
- **Hedef Sistemler:** Windows Server (2016 / 2019 / 2022 / 2025 - Domain Üyesi)

---

## 1. Amaç ve Kapsam

Üretim, veritabanı, dosya paylaşımı ve uygulama sunucularında mesai saatleri içinde gerçekleşen plansız yeniden başlatmalar iş sürekliliğini kesintiye uğratmaktadır. Bu politika ile tüm sunucu bilgisayar objeleri merkezi bir GPO altına alınarak çalışma saatleri (`Active Hours`) koruma altına alınır ve güncelleme kurulum/yeniden başlatma kuralları kontrol edilir.

---

## 2. GPO Yapılandırma Parametreleri

Sunucu Organizational Unit (OU) yapısına bağlanacak GPO içerisinde aşağıdaki 4 temel ilke yapılandırılmalıdır:

| İlke Adı (Policy Setting) | GPO Dizini | Değer / Durum | İşlev ve Etki |
| :--- | :--- | :--- | :--- |
| **Turn off auto-restart for updates during active hours** | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update > Manage end user experience` | `Enabled` (`08:00` - `20:00`) | Belirlenen mesai saatleri aralığında güncelleme kaynaklı otomatik yeniden başlatmayı engeller. |
| **Configure Automatic Updates** | `Windows Update > Manage end user experience` | `Enabled` (`3 - Auto download and notify for install`) | Güncellemeleri otomatik indirir ancak kurulum için sistem yöneticisi onayı bekler; plansız kurulumu önler. |
| **No auto-restart with logged on users for scheduled automatic updates installations** | `Windows Update > Manage end user experience` | `Enabled` | Sunucuda aktif bir kullanıcı/RDP oturumu açıkken zamanlanmış güncelleme sonrası otomatik yeniden başlatmayı durdurur. |
| **Specify deadlines for automatic updates and restarts** | `Windows Update > Manage updates offered from WSUS` | `Not Configured` / `Disabled` | Zaman aşımı (deadline) zorlamasıyla Active Hours kuralının ezilmesini önler. |

---

## 3. Uygulama Adımları

### 3.1. Sunucu Bilgisayar Objelerinin OU Kontrolü

Group Policy nesneleri varsayılan `Computers` konteynerine doğrudan bağlanamaz. Kuralın sunuculara etki etmesi için tüm sunucu bilgisayar hesaplarının (*Computer Objects*) ilgili **Sunucu OU'su (Organizational Unit)** altında yer aldığından emin olunmalıdır.

1. **Active Directory Users and Computers** (`dsa.msc`) konsolunu açın.
2. Hedef sunucuların ilgili Sunucu OU'su içinde bulunduğunu doğrulayın; varsayılan `Computers` altında kalan sunucuları ilgili OU içerisine taşıyın.

### 3.2. Group Policy Management (GPMC) Yapılandırması

1. Domain Controller üzerinde **Group Policy Management** (`gpmc.msc`) konsolunu başlatın.
2. Sunucuların bulunduğu OU'ya sağ tıklayın ve **Create a GPO in this domain, and Link it here...** seçeneğine tıklayın.
3. GPO adı olarak `Computer_Servers_Windows_Update_Active_Hours` bilgisini girin ve kaydedin.
4. Oluşturulan GPO'ya sağ tıklayıp **Edit** seçeneğini açın ve Bölüm 2'deki tabloda belirtilen 4 ilkeyi ilgili dizinler altında yapılandırın.
5. GPO'nun **Scope** sekmesinde *Security Filtering* alanında `Authenticated Users` grubunun ekli olduğunu ve **Link Enabled** seçeneğinin aktif olduğunu doğrulayın.

### 3.3. Politikayı Sunucularda Tetikleme ve Servis Yenileme

Standart 90 dakikalık yenileme döngüsünü beklemeden kuralı anında devreye almak için hedef sunucuda yönetici yetkili komut istemi (CMD) ile aşağıdaki komutları çalıştırın:

```cmd
gpupdate /force
net stop wuauserv && net start wuauserv
```

---

## 4. Doğrulama ve Kayıt Defteri (Registry) Kontrolü

Politikanın hedef sunuculara eksiksiz uygulandığını teyit etmek için iki aşamalı teknik kontrol gerçekleştirin:

### 4.1. GPO Uygulama Raporu (`gpresult`)

Yönetici yetkili komut isteminde aşağıdaki komutu çalıştırarak **Applied Group Policy Objects** başlığı altında `Computer_Servers_Windows_Update_Active_Hours` politikasının listelendiğini doğrulayın:

```cmd
gpresult /r /scope computer
```

### 4.2. Kayıt Defteri (Registry) Değerlerinin Doğrulanması

Windows Update politika parametrelerinin kayıt defterine işlendiğini kontrol etmek için **PowerShell** üzerinden aşağıdaki iki dizini sorgulayın:

```powershell
# 1. Otomatik Güncelleme ve Oturum Koruma Kontrolü:
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"

# 2. Active Hours (Etkin Saatler) Kontrolü:
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"
```

**Beklenen Referans Çıktı Değerleri:**

- **Otomatik Güncelleme Modu:** `AUOptions : 3` *(Otomatik indir, kurulum için bildir)* veya zamanlanmış kurulum tercih ediliyorsa `4`
- **Oturum Koruması:** `NoAutoRebootWithLoggedOnUsers : 1` *(Açık oturum varken otomatik yeniden başlatma kapalı)*
- **Etkin Saatler (Active Hours):** `SetActiveHours : 1` | `ActiveHoursStart : 8` | `ActiveHoursEnd : 20`

---

## 5. Kritik Operasyonel Notlar

> **WSUS Deadline (Son Tarih) Uyarısı:**
> Ortamınızda bir **WSUS (Windows Server Update Services)** sunucusu kullanılıyorsa ve WSUS üzerinden onaylanan (*Approve*) güncellemelere herhangi bir **Deadline (Son Tarih)** tanımlanırsa; Windows işletim sistemi yerel GPO üzerindeki `Active Hours` ve `NoAutoRebootWithLoggedOnUsers` kurallarını devre dışı bırakarak belirlenen saatte sunucuyu zorla yeniden başlatır. WSUS üzerinde otomatik veya manuel onaylama yapılırken **Deadline tanımlanmamalıdır**.

- **İsimlendirme Standardı:** GPO adında sabit saat aralığı (örn. `08-20`) kullanılmamalıdır. Çalışma saatlerinde değişiklik gerektiğinde yalnızca GPO içindeki *Active Hours* parametreleri güncellenmeli ve GPO **Comment** alanına revizyon notu düşülmelidir.
- **Bakım Penceresi Disiplini:** Güncellemeler sunuculara otomatik indirildikten sonra kurulum ve yeniden başlatma işlemleri yalnızca sistem yöneticileri tarafından belirlenen hafta sonu veya mesai dışı bakım pencerelerinde (*Maintenance Window*) kontrollü olarak gerçekleştirilmelidir.
