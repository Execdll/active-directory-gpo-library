# Active Directory Group Policy (GPO) Yapılandırma Standartları

Kurumsal Windows istemci ve sunucu ortamları için test edilmiş, performans, güvenlik ve kullanıcı deneyimi odaklı Group Policy (GPO) uygulama rehberleri.

---

## İçindekiler

| No | Politika / Doküman Adı | Kapsam | Dosya |
| :--- | :--- | :--- | :--- |
| **01** | **Görsel Performans & UX Standardizasyonu** | Kullanıcı (`HKCU`) | [01-Visual-Performance-UX-Standard.md](./01-Visual-Performance-UX-Standard.md) |
| **02** | **Kilit Ekranı, Görev Çubuğu ve Widget Yapılandırması** | Bilgisayar (`HKLM`) | [02-Lock-Screen-and-Taskbar-Policy.md](./02-Lock-Screen-and-Taskbar-Policy.md) |

---

## Genel Uygulama Notları

* Tüm politikalar üretim ortamına (Production) alınmadan önce pilot bir Organizasyon Birimi (OU) üzerinde test edilmelidir.
* Özel ADMX/ADML şablonu gerektiren politikalarda dosyaların SYSVOL `PolicyDefinitions` merkez deposuna aktarıldığından emin olunmalıdır.
* Değişikliklerin istemcilere anında yansıması için `gpupdate /force` komutu kullanılabilir.
