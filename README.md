# Ubuntu 22.04 Recovery & Dual Boot Repair Guide (Detailed)

## Overview

This document provides a **complete, real-world recovery workflow** for fixing a broken Ubuntu 22.04 installation in a dual boot setup with Windows 11. It includes low-level troubleshooting steps, BIOS issues, GRUB recovery, DNS/network fixes, and post-recovery validation.

---

## Problem Description

The system failed to boot into Ubuntu with the error:

"Oh no! Something has gone wrong. A problem has occurred and the system can't recover."

The same error also appeared in recovery mode.

![image(6).png](image(6).png)
---

## Root Causes (Observed & Possible)

* Corrupted GNOME / display manager (gdm3)
* Broken or partially installed packages
* GPU driver issues (especially NVIDIA)
* Full root partition (`/` at 100%)
* Wayland incompatibility
* GRUB misconfiguration after updates
* BIOS boot order / USB exclusion (Lenovo-specific)
* DNS/network issues preventing package repair

---

## Step 1: Booting from USB (Lenovo Systems)

### Issue Encountered

USB devices were **excluded from boot order** in BIOS, and keyboard navigation failed in specific BIOS sections.

### Solutions

#### Method 1: Boot Menu

* Restart system
* Press `F12` repeatedly
* Select USB device

#### Method 2: BIOS Configuration

* Enter BIOS using `F1` or `F2`
* Navigate to `Startup → Primary Boot Sequence`
* Remove USB from **"Excluded from boot order"**
* Move USB to top priority using `+` or function keys

#### Method 3: Windows Advanced Startup (used successfully)

* Hold `Shift` and click Restart
* Navigate:

  * Troubleshoot
  * Advanced Options
  * Use a Device
* Attempt to select USB (may fail if USB is not UEFI-compatible)

#### Real Outcome

A second properly created USB device was required to successfully boot into Live Ubuntu.

---

## Step 2: Boot into Live Ubuntu

* Select **"Try Ubuntu"** (NOT install)

---

## Step 3: Identify Linux Partition

```bash
lsblk
```

Look for:

* `ext4` filesystem
* Large partition (Ubuntu root)

Optional verification:

```bash
sudo blkid
```

---

## Step 4: Mount System

```bash
sudo mount /dev/sdXn /mnt
```

Mount EFI partition (critical for dual boot):

```bash
sudo mount /dev/sdX1 /mnt/boot/efi
```

---

## Step 5: Bind System Directories

```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
```

---

## Step 6: Fix DNS (CRITICAL STEP)

During chroot, internet may fail due to missing DNS resolution.

### Symptom

* `apt update` fails
* "Temporary failure resolving" errors

### Fix

```bash
sudo cp /etc/resolv.conf /mnt/etc/resolv.conf
```

This ensures proper DNS resolution inside chroot.

---

## Step 7: Enter System (chroot)

```bash
sudo chroot /mnt
```

---

## Step 8: Repair System

### Fix broken packages

```bash
apt update
dpkg --configure -a
apt install -f
```

### Reinstall desktop environment (main fix)

```bash
apt install --reinstall ubuntu-desktop gdm3 gnome-shell
```

### Fix NVIDIA drivers (if applicable)

```bash
apt purge nvidia*
ubuntu-drivers autoinstall
```

### Force Xorg (disable Wayland)

```bash
nano /etc/gdm3/custom.conf
```

Set:

```
WaylandEnable=false
```

### Check disk space

```bash
df -h
```

If full, clean:

```bash
rm -rf /var/log/*.gz
apt clean
```

### Final cleanup

```bash
apt autoremove
apt clean
```

---

## Step 9: Repair GRUB

```bash
grub-install /dev/sdX
update-grub
```

---

## Step 10: Exit and Reboot

```bash
exit
sudo reboot
```

---

## Step 11: Restore Windows in GRUB

### Issue

Windows Boot Manager missing after recovery.

### Fix

```bash
sudo apt install os-prober
```

Enable detection:

```bash
sudo nano /etc/default/grub
```

Set:

```
GRUB_DISABLE_OS_PROBER=false
```

Then update:

```bash
sudo update-grub
```

Expected output:

```
Found Windows Boot Manager
```

---

## Step 12: Verification

Check EFI entries:

```bash
ls /boot/efi/EFI
```

Expected:

* ubuntu
* Microsoft

---

## Step 13: Optional GRUB Manual Recovery (Advanced)

If system fails to boot, use GRUB console:

```bash
ls
ls (hd0,gptX)/
```

Boot manually:

```bash
set root=(hd0,gptX)
linux /boot/vmlinuz root=/dev/sdXn ro quiet splash
initrd /boot/initrd.img
boot
```

---

## Final Result

System successfully restored:

* Ubuntu 22.04 boots normally
* Windows 11 restored in GRUB
* GUI functional
* Network and DNS working

---

## Lessons Learned

* Always verify USB is UEFI-compatible
* BIOS may silently block USB boot
* DNS inside chroot is a common hidden failure
* GNOME corruption is a frequent root cause
* GRUB often loses Windows after repair

---

## Disclaimer

This repository is for educational purposes only. The recovery process described here was assisted by an Artificial Intelligence system.

---

## Author

Luis David Barahona Valdivieso
Electronic Engineering Student

---

## AI Assistant

ChatGPT (GPT-5.3 model)
