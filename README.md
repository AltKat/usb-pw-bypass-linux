# 🔐 USB Key Authentication for Linux

A simple hack to skip sudo/login passwords when your USB key is plugged in. Got tired of typing passwords on my home workstation, so I made this USB-based auth system for Fedora/KDE. When your USB is connected = no passwords needed. When it's not = back to normal security. Uses PAM integration and systemd daemon to monitor USB status and auto-unlock screen locks. 

**Documentation available in [English](guide-en.md) and [Turkish](guide-tr.md).** 

⚠️ **Use at your own risk** - this is a personal convenience tool, not enterprise security! Physical access to your USB = access to your system.

---

*Developed with assistance from Claude AI. MIT License.*
