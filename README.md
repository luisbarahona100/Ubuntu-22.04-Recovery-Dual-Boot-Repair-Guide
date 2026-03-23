# Ubuntu 22.04 Recovery & Dual Boot Repair Guide

## Overview

This document describes the full recovery process for a broken Ubuntu 22.04 installation in a dual boot system with Windows 11, including:

* Fixing "Oh no! Something has gone wrong" error
* Booting from USB on Lenovo systems
* Repairing Ubuntu from a Live USB
* Restoring GRUB and Windows Boot Manager

---

## Problem Description

The system failed to boot into Ubuntu with the error:

"Oh no! Something has gone wrong. A problem has occurred and the system can't recover."

This error also persisted in recovery mode.

---

## Root Causes (Possible)

* Corrupted GNOME environment
* Broken package dependencies
* GPU driver issues (especially NVIDIA)
* Full disk
* GRUB misconfiguration

---

## Step 1: Booting from USB (Lenovo Systems)

### Method 1: Boot Menu

* Restart system
* Press `F12` repeatedly
* Select USB device

### Method 2: BIOS Configuration

* Enter BIOS using `F1` or `F2`
* Navigate to Boot settings
* Remove USB from "Excluded from boot order"
* Move USB to top priority

### Method 3: Windows Advanced Startup

* Hold `Shift` and click Restart
* Go to:

  * Troubleshoot
  * Advanced Options
  * Use a Device
* Select USB

---

## Step 2: Boot into Live Ubuntu

* Select **"Try Ubuntu"** (do NOT install)

---

## Step 3: Identify Linux Partition

```bash
lsblk
```

Look for:

* `ext4` partition
* Largest Linux partition

---

## Step 4: Mount System

```bash
sudo mount /dev/sdXn /mnt
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

## Step 6: Enter System (chroot)

```bash
sudo chroot /mnt
```

---

## Step 7: Repair System

### Fix packages

```bash
apt update
dpkg --configure -a
apt install -f
```

### Reinstall desktop environment

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

### Clean system

```bash
apt autoremove
apt clean
```

---

## Step 8: Repair GRUB

```bash
grub-install /dev/sdX
update-grub
```

---

## Step 9: Exit and Reboot

```bash
exit
sudo reboot
```

---

## Step 10: Restore Windows in GRUB

If Windows does not appear:

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

---

## Verification

Check EFI entries:

```bash
ls /boot/efi/EFI
```

Expected:

* ubuntu
* Microsoft

---

## Final Result

System boots correctly with:

* Ubuntu 22.04
* Windows 11 available in GRUB

---

## Notes

* Always use UEFI mode with GPT
* Ensure USB is created correctly (Rufus recommended)
* Avoid full disk situations

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
