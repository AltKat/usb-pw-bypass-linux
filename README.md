# 🔐 USB Key Authentication for Linux (GDM/SDDM Display Managers)

A simple hack to skip sudo/login passwords when your USB key is plugged in. Got tired of typing passwords on my home workstation, so I made this USB-based auth system for KDE/GNOME. When your USB is connected = no passwords needed. When it's not = back to normal security. Uses PAM integration to monitor USB status and auto-unlock screen locks.

* This method is compatible with **GNOME (GDM)** and **KDE (SDDM)** display managers.

**Documentation available in [English](guide-en.md) and [Turkish](guide-tr.md).** 

***
### IMPORTANT NOTES:

**⚠️ Use at your own risk**: This is a personal convenience tool, not enterprise security! Physical access to your USB = access to your system.



**🔒 Keyring/Wallet Warning:** Due to the passwordless authentication, the GNOME Keyring or KDE Wallet will prompt you for your user password upon the first application access. This is because the PAM service does not pass a password to unlock the encrypted keyring. You only need to enter the password **once per session**.

***


*Developed with assistance from Claude AI. MIT License.*


---

# 🔐 Linux İçin USB Anahtarlı Kimlik Doğrulama (GDM/SDDM Oturum Yöneticileri)

USB anahtarınız takılı olduğunda sudo/oturum açma parolalarını atlamak için basit bir hile. Kendi iş istasyonumda sürekli parola yazmaktan yorulduğum için bu USB tabanlı kimlik doğrulama sistemini KDE/GNOME için yaptım. USB'niz bağlı olduğunda = parola gerekmez. Bağlı olmadığında = normal güvenliğe geri döner. USB durumunu izlemek ve ekran kilitlerini otomatik açmak için PAM entegrasyonu kullanır.

* Bu yöntem **GNOME (GDM)** ve **KDE (SDDM)** görüntü yöneticileriyle uyumludur.

**Dokümantasyon [İngilizce](guide-en.md) ve [Türkçe](guide-tr.md) olarak mevcuttur.**

***
### ÖNEMLİ UYARILAR:

**⚠️ Riski size aittir:** Bu, kurumsal güvenlik aracı değil, kişisel bir kolaylık aracıdır! USB'nize fiziksel erişim = sisteminize erişim demektir.



**🔒 Anahtar Zinciri/Cüzdan Uyarısı:** Parolasız kimlik doğrulama nedeniyle, GNOME Keyring veya KDE Cüzdanı (KDE Wallet), ilk uygulama erişiminde size kullanıcı parolanızı soracaktır. Bunun nedeni, PAM hizmetinin şifreli anahtar zincirinin kilidini açmak için parola iletmemesidir. Parolayı oturum başına **sadece bir kez** girmeniz gerekir.

***

*Claude AI yardımıyla geliştirilmiştir. MIT Lisansı.*

---

