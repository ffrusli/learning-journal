# 🖥️ Reviving a Discarded PC: antiX Linux Install, Offline Driver Debugging & Network Recovery

> From a dustbin to a functional Linux workstation — a hands-on project in hardware 
> diagnostics, OS installation, kernel module troubleshooting, and manual network recovery.

---

## 📋 Project Overview

This project documents the full process of rescuing an old desktop PC and turning it into 
a working Linux machine — entirely from scratch, with no internet access during the 
critical troubleshooting phase.

Everything here is real: real hardware problems, real errors, real fixes.

---

## 🖥️ Hardware Specs

| Component | Details |
|-----------|---------|
| CPU | Intel i5 Core vPro |
| RAM | 4GB DDR3 (2 × 2GB) — 4 slots, supports up to 8GB |
| Storage | 256GB HDD |
| Network | No built-in WiFi | 
| Previous OS | Windows 7 (password protected) |

Since I had no ethernet, I use my Wifi USB Adapter: TP-Link Archer T3U (USB)
| Component | Details |
|-----------|---------|
| Chipset | Realtek RTL8812BU |
---

## 🎯 Goals

- Install a lightweight Linux distro on constrained hardware
- Solve real hardware/driver problems without a tutorial to follow
- Learn the Linux networking stack from hardware up to interface
- Build a documented, repeatable troubleshooting workflow

---

## 🐧 Why antiX Linux?

Considered: Ubuntu, Lubuntu, Xubuntu, Linux Mint XFCE

Chose **antiX** because:
- Extremely lightweight (critical for 4GB RAM + HDD)
- Ships with minimal pre-configuration → forces real learning
- Strong community support for older hardware

---

## ⚙️ Installation Process

**Tools used:**
- BalenaEtcher → create bootable USB
- BIOS boot menu → boot from USB
- antiX installer → format drive, install OS

**Skills learned:**
- BIOS/UEFI navigation and boot order configuration
- Partitioning and formatting a drive during OS install
- Bootable USB creation

---

## 🚨 The Real Challenge: No Internet nor WiFi After Install

### Diagnosis
```bash
lspci           # No network controller detected
lsusb           # USB WiFi adapter detected ✓
ip a            # Only lo and eth0 — no wlan0
```

**Root cause:** antiX Linux ships without proprietary drivers.
The Realtek RTL8812BU driver was missing entirely.

**Constraint:** No Ethernet cable available → had to solve 100% offline.

---

## 🔧 Offline Driver Installation

**On a second machine:**
1. Downloaded Realtek RTL8812BU driver
2. Used **Rufus** to format a second USB as storage
   *(Windows Disk Management kept corrupting it to RAW format)*
3. Copied driver to USB, transferred to antiX machine

**On the antiX machine:**
Initially I was trying to execute and run on the usb, but it didn't work. I guess I should copy the file to my machine instead. So I did:
```bash
cp -r /media/USB/88x2bu-20210702 ~/
cd ~/88x2bu-20210702
```

then 
```bash
chmod +x install-driver.sh
sudo ./install-driver.sh
```
Installed completed, it pop-up nano editor
```bash
/etc/modprobe.d/88x2bu.conf
```
I didn't know what to uncomment, left it as it is. ctrl x+y enter. Then I reboot.

```bash
sudo reboot
````

---

## 🐛 Debugging: Driver Loaded, Interface Missing

After install, driver was present but `wlan0` still didn't appear:
```bash
lsmod | grep 88x2bu     # Driver loaded ✓
ip a                    # Still no wlan0 ✗
```

**Root cause:** Realtek RTL8812BU power management bug.

**Fix — edited kernel module config:**
```bash
sudo nano /etc/modprobe.d/88x2bu.conf
```

Uncomment and change value:
```
options 88x2bu rtw_power_mgnt=0 rtw_enusbss=0
```

**Reloaded the module:**
```bash
sudo modprobe -r 88x2bu
sudo modprobe 88x2bu
```

`wlan0` appeared. ✅

---

## 🌐 Manual Network Bringup
Walking through every layer of the Linux networking stack manually:

Try to scan network:
```bash
sudo iw dev wlan0 scan | grep SSID
```

It gave:
```bash
command failed: Network is down (-100)
# Which means wlan0 is currently down.
````

Thus:
1. Activate the interface
```bash
sudo ip link set wlan0 up
ip a
```
output:
```bash
wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

2. Scan for networks
```bash
sudo iw dev wlan0 scan | grep SSID
```
It is now listing all WiFi surrounded me! Yay! 🤓🎉

3. Hand control to network manager
```bash
sudo service connman restart
cmst
```
Select WiFi → enter password → connect.

4. Verify connection
```bash
ping -c 3 google.com
```

To make sure wlan0 always comes up automatically at boot

5. Configure interface
```bash
sudo nano /etc/network/interfaces
```
Added:
```bash
auto wlan0
iface wlan0 inet manual
    pre-up /sbin/ip link set wlan0 up
```
Boot → wlan0 auto-up → ConnMan starts → WiFi auto-connect

---

## 🔄 Post-Connection: Firmware & Updates
```bash
sudo apt update && sudo apt upgrade
sudo apt install firmware-realtek
```

Ensures the Realtek adapter behaves correctly long-term.

---

## 🧠 Skills Demonstrated

| Skill Area | Specifics |
|------------|-----------|
| Linux Installation | Bare-metal install on legacy hardware |
| BIOS Configuration | Boot order, USB booting |
| Hardware Diagnostics | `lspci`, `lsusb`, `lsmod`, `ip a` |
| Offline Problem Solving | Driver install without internet access |
| Kernel Module Management | `modprobe`, module config editing |
| Driver Debugging | Realtek power management fix |
| Network Configuration | Manual interface activation, ConnMan |
| Package Management | `apt`, firmware installation |
| USB Drive Management | Rufus vs Windows Disk Management |

---

## 💡 Lessons Learned

- Minimal distros like antiX ship without proprietary firmware — by design
- `lsusb` and `lspci` serve different purposes: USB vs PCI bus detection
- A driver loading (`lsmod`) does not guarantee a network interface appears
- Power management bugs in Realtek drivers are common and well-documented
- Rufus handles USB formatting more reliably than Windows Disk Management
- ConnMan (antiX's network manager) needs a restart to detect new interfaces

---

## 🗺️ Next Steps

- [ ] SSH remote access setup
- [ ] Linux user & permission management practice
- [ ] Network monitoring with `netstat`, `tcpdump`, `nmap`
- [ ] Conky system monitor customization
- [ ] Bash scripting: automation tasks
- [ ] Firewall basics with `ufw` / `iptables`
