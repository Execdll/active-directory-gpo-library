# Windows Sunucular İçin Güncelleştirme ve Çalışma Saatleri (Active Hours) Kılavuzu

| Özellik | Değer |
| :--- | :--- |
| **Politika Türü** | Group Policy Object (GPO) — Administrative Templates (ADMX) |
| **Uygulama Alanı** | Bilgisayar Yapılandırması (`Computer Configuration` / `HKLM`) |
| **Etki Seviyesi** | İş Sürekliliği, Planlı Bakım Yönetimi & Sistem Kararlılığı |
| **Hedef Ortam** | Windows Server 2016 / 2019 / 2022 / 2025 (Domain Üyesi) |
| **Önerilen GPO Adı** | `Computer_Servers_Windows_Update_Active_Hours` |

---

## 1. Yönetici Özeti ve Politikanın Amacı

Windows Server işletim sistemlerinde varsayılan güncelleştirme davranışları yapılandırılmadığında, sistem kritik güvenlik yamalarını indirdikten sonra mesai saatleri içerisinde otomatik olarak yeniden başlatma (*Auto-Restart*) tetikleyebilmektedir.

Kurumsal Active Directory sunucu altyapısında bu davranış şu kritik sorunlara yol açmaktadır:

1. **Mesai Saatlerinde İş Sürekliliği Kesintisi:** Üretim, veritabanı, dosya paylaşımı ve uygulama sunucularının çalışma saatleri içerisinde plansız yeniden başlaması aktif kullanıcı oturumlarının ve servislerin kesilmesine neden olur.
2. **Kontrolsüz Yama Kurulumu:** Yöneticinin bilgisi ve bakım penceresi (*Maintenance Window*) dışında yüklenen güncellemeler, bağımlı servislerin sıralı başlatılmasını bozabilir.
3. **WSUS ve Zaman Aşımı Çakışmaları:** Varsayılan ayarlarla bırakılan sunucularda zaman aşımı (*Deadline*) veya varsayılan etkin saatlerin yetersiz kalması zorunlu yeniden başlatmaları devreye sokar.

Bu prosedür, Group Policy üzerinden çalışma saatlerini (`08:00` - `20:00`) koruma altına alarak otomatik yeniden başlatmayı engeller ve güncelleme kurulumunu sistem yöneticisi kontrolüne sabitler.

---

## 2. Yapılandırılan Group Policy (ADMX) Matrisi

| No | İlke Adı (Policy Setting) | GPO Dizini (`Computer Configuration > Policies > Administrative Templates > ...`) | Atanan Değer |
| :--- | :--- | :--- | :--- |
| **01** | **Turn off auto-restart for updates during active hours** | `Windows Components > Windows Update > Manage end user experience` | `Enabled` (`08:00` - `20:00`) |
| **02** | **Configure Automatic Updates** | `Windows Components > Windows Update > Manage end user experience` | `Enabled` (`3 - Auto download and notify for install`) |
| **03** | **No auto-restart with logged on users for scheduled automatic updates installations** | `Windows Components > Windows Update > Manage end user experience` | `Enabled` |
| **04** | **Specify deadlines for automatic updates and restarts** | `Windows Components > Windows Update > Manage updates offered from WSUS` | `Not Configured` / `Disabled` |

---

## 3. Teknik Parametre ve Kayıt Defteri (Registry) Analizi

GPO uygulandığında hedef sunucuların kayıt defterinde (`HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate`) aşağıdaki parametreler oluşturulur:

- **`SetActiveHours = 1` (`ActiveHoursStart = 8`, `ActiveHoursEnd = 20`):** Belirlenen mesai saatleri aralığında Windows Update kaynaklı otomatik yeniden başlatmayı engeller.
- **`AUOptions = 3` (Kurumsal Standart):** Güncellemeleri otomatik indirir ancak kurulum için sistem yöneticisi onayını bekler. *(Zamanlanmış kurulum tercih ediliyorsa `4` olarak yapılandırılabilir.)*
- **`NoAutoRebootWithLoggedOnUsers = 1`:** Sunucuda aktif bir kullanıcı/RDP oturumu açıkken zamanlanmış güncelleme sonrası otomatik yeniden başlatmayı durdurur.

> **WSUS Deadline Uyarısı:** Ortamınızda **WSUS** kullanılıyorsa ve onaylanan (*Approve*) güncellemelere **Deadline (Son Tarih)** tanımlanırsa; Windows işletim sistemi yerel GPO üzerindeki `Active Hours` ve `NoAutoRebootWithLoggedOnUsers` kurallarını devre dışı bırakarak belirlenen saatte sunucuyu zorla yeniden başlatır. WSUS onay kurallarında **Deadline tanımlanmamalıdır**.

---

## 4. GPO Dağıtım Prosedürü

Bu ayar sunucu genelinde (`HKEY_LOCAL_MACHINE`) geçerli olduğu için **Computer Configuration** altında yapılandırılmalıdır:

1. **Sunucu OU Kontrolü (`dsa.msc`):** Group Policy nesneleri varsayılan `Computers` konteynerine bağlanamaz. Tüm sunucu bilgisayar objelerinin ilgili **Sunucu OU'su** altında olduğundan emin olun.
2. **`gpmc.msc` Konsolunu Açın:** Hedef Sunucu Organizasyon Birimi (OU) üzerine sağ tıklayın ve **Create a GPO in this domain, and Link it here...** seçeneği ile `Computer_Servers_Windows_Update_Active_Hours` adında yeni bir GPO oluşturun.
3. **İlgili Menüye Gidin:** GPO'yu düzenleyin (**Edit**) ve şu yolu izleyin:  
   `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update`
4. **İlkeleri Yapılandırın:** Bölüm 2'deki matriste belirtilen 4 ilkeyi ilgili alt klasörler altında tanımlayın ve kaydedin.
5. **Kapsam (Scope) Doğrulaması:** GPO'nun **Scope** sekmesinde *Security Filtering* alanında `Authenticated Users` grubunun yer aldığını ve **Link Enabled** seçeneğinin aktif olduğunu doğrulayın.
6. **Politikayı Tetikleyin:** Hedef sunucuda yönetici komut isteminde politikayı güncelleyip Windows Update servisini yenileyin:
   ```cmd
   gpupdate /force && net stop wuauserv && net start wuauserv
   ```

---

## 5. PowerShell ile Hızlı Kontrol Komutu

Hedef sunucuda politikanın kayıt defterine işlendiğini doğrulamak için yönetici PowerShell oturumunda şu komutları çalıştırın:

```powershell
# Otomatik Güncelleme ve Oturum Koruma Kontrolü
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"

# Active Hours (Etkin Saatler) Kontrolü
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"
```

**Beklenen Çıktı:** `AUOptions : 3`, `NoAutoRebootWithLoggedOnUsers : 1`, `SetActiveHours : 1`, `ActiveHoursStart : 8`, `ActiveHoursEnd : 20`

---

## 6. Test ve Doğrulama Kontrol Listesi

| Sıra | Kontrol Adımı / Metot | Beklenen Çıktı / Değer | Durum |
| :--- | :--- | :--- | :--- |
| **1** | `gpresult /r /scope computer` komutunun çalıştırılması | **Applied Group Policy Objects** altında `Computer_Servers_Windows_Update_Active_Hours` politikasının görünmesi | `[ ]` Doğrulandı |
| **2** | Kayıt defteri (`...\WindowsUpdate\AU`) PowerShell kontrolü | `AUOptions : 3` ve `NoAutoRebootWithLoggedOnUsers : 1` değerlerinin görülmesi | `[ ]` Doğrulandı |
| **3** | Kayıt defteri (`...\WindowsUpdate`) PowerShell kontrolü | `SetActiveHours : 1`, `ActiveHoursStart : 8` ve `ActiveHoursEnd : 20` değerlerinin görülmesi | `[ ]` Doğrulandı |
| **4** | Windows Update Ayarları (`sconfig` / Settings) kontrolü | Etkin saatlerin `08:00 - 20:00` olarak kilitli/yapılandırılmış görünmesi | `[ ]` Doğrulandı |
