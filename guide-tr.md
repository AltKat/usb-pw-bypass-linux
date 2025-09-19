⚠️ **KRİTİK:** Bu rehberdeki `usb-USB_ID` ve `username` yer tutucularını mutlaka gerçek değerlerinizle değiştirmelisiniz!

---

## 📋 İlk Kurulum - USB ID'nizi Bulun

**1. USB anahtarınızı bilgisayara takın**

**2. USB ID'nizi bulun:**
```bash
ls -l /dev/disk/by-id/ | grep usb
```

**Örnek çıktı:**
```
usb-SanDisk_Cruzer_Blade_4C530001071218115134 -> ../../sdb1
usb-Kingston_DataTraveler_G3_001CC0EC3410B971E9780E1B -> ../../sdc1
```

**3. USB'nizin tam ID'sini kopyalayın:**
- Örnek: `usb-SanDisk_Cruzer_Blade_4C530001071218115134`
- Bu sizin **gerçek USB ID'niz** olacak

**4. Kullanıcı adınızı bulun:**
```bash
whoami
```
**Örnek çıktı:**
```
altkat
```
- Bu sizin **kullanıcı adınız** olacak

---

## 📁 Kullanılan Dosyalar

```
/usr/local/bin/check-usb-key.sh         # sudo ve login için
/etc/pam.d/sudo                         # sudo PAM konfigürasyonu  
/etc/pam.d/sddm                         # login PAM konfigürasyonu
/usr/local/bin/unlock-if-usb.sh         # ekran kilidi daemon'u
/etc/systemd/system/usb-unlock.service  # systemd servisi
```

---

## ⚙️ KURULUM

### 1️⃣ Ana Script (sudo ve login için)

```bash
sudo nano /usr/local/bin/check-usb-key.sh
```

**İçerik (DİKKAT: USB_ID'yi değiştirin!):**
```bash
#!/bin/bash
# USB_ID kısmını yukarıdaki gerçek değerinizle değiştirin!
# Örnek: usb-SanDisk_Cruzer_Blade_4C530001071218115134
if [ -e "/dev/disk/by-id/usb-USB_ID" ]; then
  exit 0  # USB bulundu, parolayı atla
else
  exit 1  # USB yok, parola iste
fi
```

**GERÇEK ÖRNEK (sizinki farklı olacak):**
```bash
#!/bin/bash
if [ -e "/dev/disk/by-id/usb-SanDisk_Cruzer_Blade_4C530001071218115134" ]; then
  exit 0
else
  exit 1
fi
```

**İzinleri ayarlayın:**
```bash
sudo chmod 700 /usr/local/bin/check-usb-key.sh
sudo chown root:root /usr/local/bin/check-usb-key.sh
```

### 2️⃣ PAM Konfigürasyonu

**Sudo için:**
```bash
sudo nano /etc/pam.d/sudo
```

**SDDM login için:**
```bash
sudo nano /etc/pam.d/sddm
```

**Her iki dosyanın da EN ÜSTÜNE bu satırı ekleyin:**
```bash
# USB anahtar kontrolü - bu satır en üstte olmalı!
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
```

**Dosya örneği (/etc/pam.d/sudo):**
```bash
# USB anahtar kontrolü - bu satır en üstte olmalı!
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
#%PAM-1.0
auth       include      system-auth
account    include      system-auth
password   include      system-auth
session    include      system-auth
```

### 3️⃣ Ekran Kilidi Daemon'u

**Script oluşturun:**
```bash
sudo nano /usr/local/bin/unlock-if-usb.sh
```

**İçerik (DİKKAT: USB_ID ve USER'ı değiştirin!):**
```bash
#!/bin/bash
# MUTLAKA DEĞİŞTİRİN: USB ID'nizi buraya yazın!
USB_ID="usb-USB_ID"  # Örnek: usb-SanDisk_Cruzer_Blade_4C530001071218115134

# MUTLAKA DEĞİŞTİRİN: Kullanıcı adınızı buraya yazın!  
USER="username"      # Örnek: ahmet, ayşe, vs...

while true; do
    # USB takılı mı kontrol et
    if [ -e "/dev/disk/by-id/$USB_ID" ]; then
        # Kullanıcının aktif oturumunu bul
        SESSION=$(loginctl list-sessions | grep $USER | awk '{print $1}')
        if [ -n "$SESSION" ]; then
            # Oturumu unlock et
            loginctl unlock-session $SESSION
        fi
    fi
    sleep 2  # 2 saniye bekle
done
```

**GERÇEK ÖRNEK (sizinki farklı olacak):**
```bash
#!/bin/bash
USB_ID="usb-SanDisk_Cruzer_Blade_4C530001071218115134"  # Gerçek USB ID
USER="ahmet"                                             # Gerçek kullanıcı adı

while true; do
    if [ -e "/dev/disk/by-id/$USB_ID" ]; then
        SESSION=$(loginctl list-sessions | grep $USER | awk '{print $1}')
        if [ -n "$SESSION" ]; then
            loginctl unlock-session $SESSION
        fi
    fi
    sleep 2
done
```

**İzinleri ayarlayın:**
```bash
sudo chmod 700 /usr/local/bin/unlock-if-usb.sh
sudo chown root:root /usr/local/bin/unlock-if-usb.sh
```

### 4️⃣ Systemd Servisi

**Servis dosyası oluşturun:**
```bash
sudo nano /etc/systemd/system/usb-unlock.service
```

**İçerik:**
```ini
[Unit]
Description=USB Unlock Daemon
After=graphical.target

[Service]
Type=simple
ExecStart=/usr/local/bin/unlock-if-usb.sh
Restart=always
User=root

[Install]
WantedBy=default.target
```

**Servisi başlatın:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now usb-unlock.service
```

---

## 🧪 TEST ETME

### ✅ Sudo Testi
```bash
# Önbelleği temizle
sudo -k

# Test komutu (USB takılıyken parolasız çalışmalı)
sudo whoami
# Sonuç: "root" (parola sormadan)
```

### ✅ Login Testi  
1. Oturumu kapatın (`logout` veya menüden çıkış)
2. USB takılıyken giriş yapmayı deneyin
3. Parola sormadan giriş yapabilmelisiniz

### ✅ Ekran Kilidi Testi
```bash
# Ekranı kilitleyin
Ctrl+Alt+L

# USB takılıyken kilit otomatik açılmalı (2-3 saniye içinde)
```

---

## 🗑️ GERİ ALMA (Kaldırma)

### Script Dosyalarını Kaldırın
```bash
sudo rm -f /usr/local/bin/check-usb-key.sh
sudo rm -f /usr/local/bin/unlock-if-usb.sh
```

### PAM Konfigürasyonunu Temizleyin
```bash
sudo nano /etc/pam.d/sudo      # Satırı silin
sudo nano /etc/pam.d/sddm      # Satırı silin
```

**Silinecek satır:**
```bash
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
```

### Systemd Servisini Kaldırın
```bash
sudo systemctl disable --now usb-unlock.service
sudo rm -f /etc/systemd/system/usb-unlock.service
sudo systemctl daemon-reload
```

---

## ⚠️ GÜVENLİK UYARILARI

🔴 **KRİTİK UYARI:** USB'nizi kaybederseniz = parolasız erişim!

🔒 **Script izinleri mutlaka 700 olmalı** 

💻 **Daemon her 2 saniyede kontrol eder**

🏠 **Sadece güvenli ortamlarda kullanın**

🔐 **USB ID'nizi asla paylaşmayın**

---

## 🐛 Sorun Giderme

### "Permission denied" Hatası
```bash
# Script izinlerini kontrol edin
ls -la /usr/local/bin/check-usb-key.sh
# Çıktı: -rwx------ 1 root root ... (700 izinleri)

# Farklıysa düzeltin
sudo chmod 700 /usr/local/bin/check-usb-key.sh
sudo chown root:root /usr/local/bin/check-usb-key.sh
```

### "USB ID bulunamadı" Sorunu
```bash
# USB takılıyken kontrol edin
ls -l /dev/disk/by-id/ | grep usb

# Script'teki USB_ID ile karşılaştırın
cat /usr/local/bin/check-usb-key.sh
```

### Daemon Çalışmıyor
```bash
# Servis durumunu kontrol edin
sudo systemctl status usb-unlock.service

# Logları görüntüleyin  
sudo journalctl -u usb-unlock.service -f
```

---


## 📚 Nasıl Çalışır

**PAM Entegrasyonu:** Kimlik doğrulama isteklerini yakalamak için PAM (Pluggable Authentication Modules) kullanır

**USB Algılama:** Spesifik USB cihazınız için `/dev/disk/by-id/` dizinini izler

**Oturum Yönetimi:** Kullanıcı oturumlarını ve ekran kilitlerini yönetmek için `loginctl` kullanır

**Systemd Servisi:** Sürekli izleme için arka planda daemon olarak çalışır

---

Artık sudo, login ve ekran kilidini USB anahtarınıza bağladınız! 🎉

**Son kontrol:** Tüm `usb-USB_ID` ve `username` yer tutucularını gerçek değerlerinizle değiştirdiğinizden emin olun!
