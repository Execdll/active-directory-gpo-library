# Windows İstemciler İçin Hızlı Başlatma (Fast Startup) Devre Dışı Bırakma Kılavuzu

| Özellik | Değer |
| :--- | :--- |
| **Politika Türü** | Group Policy Preferences (GPP) — Registry Settings |
| **Uygulama Alanı** | Bilgisayar Yapılandırması (`Computer Configuration` / `HKLM`) |
| **Etki Seviyesi** | GPO Tutarlılığı, Güncelleme Yönetimi & Sistem Kararlılığı |
| **Hedef Ortam** | Windows 10 / 11 Pro & Enterprise |
| **Önerilen GPO Adı** | `Computer_Disable_Fast_Startup` |

---

## 1. Yönetici Özeti ve Politikanın Amacı

Windows 10 ve 11 işletim sistemlerinde varsayılan olarak etkin gelen **Hızlı Başlatma (Fast Startup / Hiberboot)** özelliği, bilgisayar kapatıldığında (*Shut down*) sistemi tam olarak kapatmak yerine çekirdek (kernel) oturumunu `hiberfil.sys` dosyasına kaydederek karma bir hazırda bekletme moduna alır.

Kurumsal Active Directory ortamlarında bu davranış şu kritik sorunlara yol açmaktadır:

1. **Computer GPO Politikalarının Uygulanmaması:** Bilgisayar yapılandırması (`Computer Configuration`) politikalarının birçoğu yalnızca **tam soğuk önyükleme (Cold Boot)** sırasında işlenir. Fast Startup açık olduğunda kullanıcı bilgisayarı kapatıp açsa bile çekirdek oturumu yeniden başlatılmadığı için GPO değişiklikleri devreye girmez.
2. **Windows Güncellemelerinin Beklemede Kalması:** Güncelleme sonrası yeniden başlatma gerektiren yamalar, kullanıcı "Bilgisayarı Kapat" seçeneğini kullandığında tamamlanamaz.
3. **Ağ Sürücüleri ve Login Script Sorunları:** Önceki oturumun ağ durumu önbellekten yüklendiği için etki alanı denetleyicisiyle (Domain Controller) erken oturum iletişiminde gecikmeler yaşanabilir.

Bu prosedür, kayıt defteri üzerinden `HiberbootEnabled` değerini `0` olarak sabitleyerek tüm istemcilerin her kapatma işleminde tam ve temiz bir sistem kapanışı gerçekleştirmesini sağlar.

---

## 2. Yapılandırılan Kayıt Defteri (Registry) Matrisi

| No | Kayıt Defteri Yolu (Hive & Key Path) | Değer Adı | Tür | Atanan Değer |
| :--- | :--- | :--- | :--- | :--- |
| **01** | `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Power` | `HiberbootEnabled` | `REG_DWORD` | `0` (`0x0`) |

---

## 3. Teknik Parametre Analizi

* **`HiberbootEnabled = 1` (Varsayılan):** Hızlı Başlatma etkindir. Sistem kapatıldığında kullanıcı oturumu kapatılır ancak Windows çekirdeği ve sürücü durumu diske kaydedilir.
* **`HiberbootEnabled = 0` (Kurumsal Standart):** Hızlı Başlatma devre dışıdır. **Bilgisayarı Kapat** komutu verildiğinde tüm servisler, sürücüler ve çekirdek oturumu tamamen sonlandırılır; bir sonraki açılışta sıfırdan başlatılır.

> **Not:** Bu ayar yalnızca *Hızlı Başlatma* (Fast Startup) özelliğini kapatır; dizüstü bilgisayarlardaki normal **Hazırda Beklet (Hibernate)** veya **Uyku (Sleep)** modlarını devre dışı bırakmaz.

---

## 4. GPO (Group Policy Preferences) Dağıtım Prosedürü

Bu ayar makine genelinde (`HKEY_LOCAL_MACHINE`) geçerli olduğu için **Computer Configuration** altında yapılandırılmalıdır:

1. **`gpmc.msc` Konsolunu Açın:** Hedef Bilgisayar Organizasyon Birimi (OU) üzerine gelin ve yeni bir GPO oluşturup bağlayın.
2. **İlgili Menüye Gidin:** GPO'yu düzenleyin ve şu yolu izleyin:  
   `Computer Configuration > Preferences > Windows Settings > Registry`
3. **Yeni Kayıt Ekleyin:** Sağ tıklayıp **New > Registry Item** seçeneğini seçin.
4. **Parametreleri Tanımlayın:** Açılan pencerede aşağıdaki ayarları yapın:
   * **Action:** `Update` *(Anahtar yoksa oluşturur, varsa günceller)*
   * **Hive:** `HKEY_LOCAL_MACHINE`
   * **Key Path:** `SYSTEM\CurrentControlSet\Control\Session Manager\Power`
   * **Value name:** `HiberbootEnabled`
   * **Value type:** `REG_DWORD`
   * **Value data:** `0` (`Hexadecimal` / `Decimal`)
5. **Kaydedin:** **OK** butonuna tıklayarak yapılandırmayı tamamlayın.

---

## 5. PowerShell ile Hızlı Kontrol Komutu

İstemci bilgisayarda politikanın uygulandığını doğrulamak için yönetici PowerShell oturumunda şu komutu çalıştırın:

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Power" -Name "HiberbootEnabled"
```

**Beklenen Çıktı:** `HiberbootEnabled : 0`

---

## 6. Test ve Doğrulama Kontrol Listesi

| Sıra | Kontrol Adımı / Metot | Beklenen Çıktı / Değer | Durum |
| :--- | :--- | :--- | :--- |
| **1** | `gpupdate /force` komutunun çalıştırılması | `Computer Policy update has completed successfully` mesajı | `[ ]` Doğrulandı |
| **2** | Kayıt defteri (`regedit.exe` / PowerShell) kontrolü | `HiberbootEnabled` değerinin `0` olarak görülmesi | `[ ]` Doğrulandı |
| **3** | Denetim Masası (`powercfg.cpl`) kontrolü | **Güç düğmelerinin yapacaklarını seçin** menüsünde *Hızlı başlatmayı aç* seçeneğinin kapalı/pasif olması | `[ ]` Doğrulandı |
