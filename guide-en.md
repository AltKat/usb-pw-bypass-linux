⚠️ **CRITICAL:** You must replace `usb-USB_ID` and `username` placeholders in this guide with your actual values!

---

## 📋 Initial Setup - Find Your USB ID

**1. Plug your USB key into the computer**

**2. Find your USB ID:**
```bash
ls -l /dev/disk/by-id/ | grep usb
```

**Example output:**
```
usb-SanDisk_Cruzer_Blade_4C530001071218115134 -> ../../sdb1
usb-Kingston_DataTraveler_G3_001CC0EC3410B971E9780E1B -> ../../sdc1
```

**3. Copy your USB's complete ID:**
- Example: `usb-SanDisk_Cruzer_Blade_4C530001071218115134`
- This will be your **actual USB ID**

**4. Find your username:**
```bash
whoami
```
**Example output:**
```
altkat
```
- This will be your **username**

---

## 📁 Files Used

```
/usr/local/bin/check-usb-key.sh         # for sudo and login
/etc/pam.d/sudo                         # sudo PAM configuration
/etc/pam.d/gdm-password                 # GNOME (GDM) login PAM configuration
/etc/pam.d/sddm                         # KDE (SDDM) login PAM configuration (Skip this file if you use GNOME)
```

---

## ⚙️ INSTALLATION

### 1️⃣ Main Script (for sudo and login)

```bash
sudo nano /usr/local/bin/check-usb-key.sh
```

**Content (ATTENTION: Replace USB_ID!):**
```bash
#!/bin/bash
# Replace USB_ID with your actual value from above!
# Example: usb-SanDisk_Cruzer_Blade_4C530001071218115134
if [ -e "/dev/disk/by-id/usb-USB_ID" ]; then
  exit 0  # USB found, skip password
else
  exit 1  # No USB, require password
fi
```

**REAL EXAMPLE (yours will be different):**
```bash
#!/bin/bash
if [ -e "/dev/disk/by-id/usb-SanDisk_Cruzer_Blade_4C530001071218115134" ]; then
  exit 0
else
  exit 1
fi
```

**Set permissions:**
```bash
sudo chmod 700 /usr/local/bin/check-usb-key.sh
sudo chown root:root /usr/local/bin/check-usb-key.sh
```

### 2️⃣ PAM Configuration

**For sudo:**
```bash
sudo nano /etc/pam.d/sudo
```
**For GNOME (GDM) login:**
```bash
sudo nano /etc/pam.d/gdm-password
```

**For KDE (SDDM) login (SKIP if you use GNOME):**
```bash
sudo nano /etc/pam.d/sddm
```

**Add this line AT THE TOP of the required file(s):**
```bash
# USB key check - this line must be at the top!
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
```

**File example (/etc/pam.d/sudo):**
```bash
# USB key check - this line must be at the top!
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
#%PAM-1.0
auth       include      system-auth
account    include      system-auth
password   include      system-auth
session    include      system-auth
```
---

## 🧪 TESTING

### ✅ Sudo Test
```bash
# Clear cache
sudo -k

# Test command (should work without password when USB is plugged)
sudo whoami
# Result: "root" (without asking for password)
```

### ✅ Login Test  
1. Logout (`logout` or menu exit)
2. Try to login with USB plugged in
3. Should login without asking for password

---

## 🗑️ ROLLBACK (Uninstall)

### Remove Script Files
```bash
sudo rm -f /usr/local/bin/check-usb-key.sh
```

### Clean PAM Configuration
```bash
sudo nano /etc/pam.d/sudo      # Remove the line
sudo nano /etc/pam.d/gdm-password # Remove the line
sudo nano /etc/pam.d/sddm      # Remove the line (If you added it)
```

**Line to remove:**
```bash
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
```
---

## ⚠️ SECURITY WARNINGS

🔴 **CRITICAL WARNING:** If you lose your USB = passwordless access!

🔒 **Script permissions must be 700** 

🏠 **Use only in secure environments**

🔐 **Never share your USB ID**

---

## 🐛 Troubleshooting

### "Permission denied" Error
```bash
# Check script permissions
ls -la /usr/local/bin/check-usb-key.sh
# Output: -rwx------ 1 root root ... (700 permissions)

# Fix if different
sudo chmod 700 /usr/local/bin/check-usb-key.sh
sudo chown root:root /usr/local/bin/check-usb-key.sh
```

### "USB ID not found" Problem
```bash
# Check while USB is plugged in
ls -l /dev/disk/by-id/ | grep usb

# Compare with USB_ID in script
cat /usr/local/bin/check-usb-key.sh
```

---

## 📚 How It Works

**PAM Integration:** Uses PAM (Pluggable Authentication Modules) to intercept authentication requests

**USB Detection:** Monitors `/dev/disk/by-id/` for your specific USB device

**Session Management:** Uses `loginctl` to manage user sessions and screen locks

---

Now sudo, and login are tied to your USB key! 🎉

**Final check:** Make sure you've replaced all `usb-USB_ID` and `username` placeholders with your actual values!
