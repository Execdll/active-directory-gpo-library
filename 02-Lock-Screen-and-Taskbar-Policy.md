# GPO Kilit Ekranı, Görev Çubuğu ve Widget Yapılandırma Kılavuzu

| Özellik | Değer |
| :--- | :--- |
| **Politika Türü** | Administrative Templates (ADMX / ADML) |
| **Uygulama Alanı** | Bilgisayar Yapılandırması (`Computer Configuration`) |
| **Etki Seviyesi** | Güvenlik, Bant Genişliği & Kullanıcı Arayüzü |
| **Hedef Ortam** | Windows 10 / 11 Pro & Enterprise |
| **Önerilen GPO Adı** | `Computer_Lock_Screen` |

---

## 1. Giriş ve Amaç

Bu dokümantasyon, Active Directory ortamında Grup Politikaları (GPO) kullanılarak istemci bilgisayarların kilit ekranı (*Lock Screen*), çevrimdışı harita güncellemeleri, görev çubuğu haberler/ilgi alanları (*News and Interests*) ve Windows 11 widget (*Araçlar*) özelliklerinin merkezi olarak yönetilmesi ve kısıtlanması amacıyla hazırlanan özel ADMX/ADML şablonlarının SYSVOL içerisine entegrasyonu ile `Computer_Lock_Screen` GPO nesnesinin yapılandırma adımlarını kapsamaktadır.

---

## 2. Özel ADMX/ADML Şablonlarının Hazırlanması ve SYSVOL Konumlandırılması

Özel şablon dosyaları olan `Dsh.admx` ve `Dsh.adml` dosyaları, Active Directory altyapısında **Merkezi Depo (Central Store)** yapısına uygun olarak SYSVOL dizinine kopyalanmalıdır:

* **`Dsh.admx`:** Şablon ve politika tanımları
* **`Dsh.adml`:** Dil ve arayüz kaynak dosyası

**SYSVOL Üzerindeki Hedef Yollar:**

* **ADMX Dosyası Konumu:**  
  `\\<DomainName>\sysvol\<DomainName>\Policies\PolicyDefinitions\Dsh.admx`
* **ADML Dosyası Konumu (Türkçe):**  
  `\\<DomainName>\sysvol\<DomainName>\Policies\PolicyDefinitions\tr-TR\Dsh.adml`
* **ADML Dosyası Konumu (İngilizce):**  
  `\\<DomainName>\sysvol\<DomainName>\Policies\PolicyDefinitions\en-US\Dsh.adml`

> **Not:** Özel `.admx` şablonunun **Group Policy Management Editor** arayüzünde hatasız görüntülenebilmesi için, konsolun açıldığı sunucu veya yönetim bilgisayarının işletim sistemi diliyle eşleşen dil klasöründe (`tr-TR` veya `en-US`) karşılık gelen `.adml` dosyasının bulunması zorunludur.

---

## 3. GPO Nesnesinin Oluşturulması

Active Directory ortamında **Group Policy Management (`gpmc.msc`)** konsolu üzerinden yeni ilke nesnesi oluşturulur ve ilgili bilgisayar Organizasyon Birimi'ne (OU) bağlanır:

* **GPO Adı:** `Computer_Lock_Screen`
* **Hedef Kapsam:** İlgili Bilgisayar OU'su (`Computer Configuration`)

---

## 4. Politika Ayarları ve Yapılandırma Adımları

Oluşturulan `Computer_Lock_Screen` adlı GPO nesnesi düzenlenerek **Computer Configuration > Policies > Administrative Templates** altında aşağıdaki politika ayarları yapılandırılır:

| Kategori / Yol | Politika Adı (Policy) | Durum (Setting) |
| :--- | :--- | :--- |
| `Control Panel/Personalization` | Prevent enabling lock screen slide show | **Enabled** |
| `Control Panel/Personalization` | Prevent lock screen background motion | **Enabled** |
| `Windows Components/Maps` | Turn off Automatic Download and Update of Map Data | **Enabled** |
| `Windows Components/Maps` | Turn off unsolicited network traffic on the Offline Maps settings page | **Enabled** |
| `Windows Components/News and interests` | Enable news and interests on the taskbar | **Disabled** |
| `Windows Components/Widgets` | Allow widgets | **Disabled** |
| `Windows Components/Widgets` | Disable Widgets Board | **Enabled** |
| `Windows Components/Widgets` | Disable Widgets On Lock Screen | **Enabled** |

---

## 5. Doğrulama ve Kontrol

Yapılandırma sonrasında istemci bilgisayarlarda yönetici komut isteminde politikaların zorla uygulanması ve sonuçların raporlanması için:

```
gpupdate /force
gpresult /r /scope computer
```

**Beklenen Sonuç:**
* Kilit ekranı slayt gösterisi ve arka plan hareketleri devre dışı kalır.
* Çevrimdışı harita veri güncellemeleri ve arka plan ağ trafiği engellenir.
* Görev çubuğundaki *Haberler ve İlgi Alanları* ile Windows 11 *Widgets* paneli (kilit ekranı dahil) tamamen kapatılır.
