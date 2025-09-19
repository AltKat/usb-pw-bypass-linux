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
/etc/pam.d/sddm                         # login PAM configuration
/usr/local/bin/unlock-if-usb.sh         # screen lock daemon
/etc/systemd/system/usb-unlock.service  # systemd service
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

**For SDDM login:**
```bash
sudo nano /etc/pam.d/sddm
```

**Add this line AT THE TOP of both files:**
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

### 3️⃣ Screen Lock Daemon

**Create script:**
```bash
sudo nano /usr/local/bin/unlock-if-usb.sh
```

**Content (ATTENTION: Replace USB_ID and USER!):**
```bash
#!/bin/bash
# MUST CHANGE: Write your USB ID here!
USB_ID="usb-USB_ID"  # Example: usb-SanDisk_Cruzer_Blade_4C530001071218115134

# MUST CHANGE: Write your username here!  
USER="username"      # Example: john, mary, etc...

while true; do
    # Check if USB is plugged in
    if [ -e "/dev/disk/by-id/$USB_ID" ]; then
        # Find user's active session
        SESSION=$(loginctl list-sessions | grep $USER | awk '{print $1}')
        if [ -n "$SESSION" ]; then
            # Unlock the session
            loginctl unlock-session $SESSION
        fi
    fi
    sleep 2  # Wait 2 seconds
done
```

**REAL EXAMPLE (yours will be different):**
```bash
#!/bin/bash
USB_ID="usb-SanDisk_Cruzer_Blade_4C530001071218115134"  # Real USB ID
USER="john"                                              # Real username

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

**Set permissions:**
```bash
sudo chmod 700 /usr/local/bin/unlock-if-usb.sh
sudo chown root:root /usr/local/bin/unlock-if-usb.sh
```

### 4️⃣ Systemd Service

**Create service file:**
```bash
sudo nano /etc/systemd/system/usb-unlock.service
```

**Content:**
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

**Start the service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now usb-unlock.service
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

### ✅ Screen Lock Test
```bash
# Lock the screen
Ctrl+Alt+L

# Lock should automatically unlock when USB is plugged (within 2-3 seconds)
```

---

## 🗑️ ROLLBACK (Uninstall)

### Remove Script Files
```bash
sudo rm -f /usr/local/bin/check-usb-key.sh
sudo rm -f /usr/local/bin/unlock-if-usb.sh
```

### Clean PAM Configuration
```bash
sudo nano /etc/pam.d/sudo      # Remove the line
sudo nano /etc/pam.d/sddm      # Remove the line
```

**Line to remove:**
```bash
auth sufficient pam_exec.so seteuid quiet /usr/local/bin/check-usb-key.sh
```

### Remove Systemd Service
```bash
sudo systemctl disable --now usb-unlock.service
sudo rm -f /etc/systemd/system/usb-unlock.service
sudo systemctl daemon-reload
```

---

## ⚠️ SECURITY WARNINGS

🔴 **CRITICAL WARNING:** If you lose your USB = passwordless access!

🔒 **Script permissions must be 700** 

💻 **Daemon checks every 2 seconds**

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

### Daemon Not Working
```bash
# Check service status
sudo systemctl status usb-unlock.service

# View logs  
sudo journalctl -u usb-unlock.service -f
```

---

## 📚 How It Works

**PAM Integration:** Uses PAM (Pluggable Authentication Modules) to intercept authentication requests

**USB Detection:** Monitors `/dev/disk/by-id/` for your specific USB device

**Session Management:** Uses `loginctl` to manage user sessions and screen locks

**Systemd Service:** Runs as a background daemon for continuous monitoring

---

Now sudo, login, and screen lock are tied to your USB key! 🎉

**Final check:** Make sure you've replaced all `usb-USB_ID` and `username` placeholders with your actual values!
