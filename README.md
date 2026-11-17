# Active Directory Group Policy (GPO) Yapılandırma Standartları

Kurumsal Windows istemci ve sunucu ortamları için test edilmiş, performans ve kullanıcı deneyimi odaklı Group Policy (GPO) uygulama rehberleri.

---

## İçindekiler

| No | Politika / Doküman Adı | Kapsam | Dosya |
| :--- | :--- | :--- | :--- |
| **01** | **Görsel Performans & UX Standardizasyonu** | Kullanıcı (`HKCU`) | [01-Visual-Performance-UX-Standard.md](./01-Visual-Performance-UX-Standard.md) |

---

## Genel Uygulama Notları
* Tüm politikalar üretim ortamına (Production) alınmadan önce pilot bir Organizasyon Birimi (OU) üzerinde test edilmelidir.
* Değişikliklerin istemcilere anında yansıması için `gpupdate /force` komutu kullanılabilir.
