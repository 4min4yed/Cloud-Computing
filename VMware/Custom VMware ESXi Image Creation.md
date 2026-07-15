# Deploying VMware ESXi 8.0 3e on UNSUPPORTED HW: Realtek RTL8111/8168 Network Controller
## Manual creation of a custom ESXi iso image + deploying VMware ESXi 8.


---

# Abstract

This report documents the complete process of deploying VMware ESXi 8 on consumer-grade HP hardware which the **NIC**: `Realtek RTL8111/8168 PCIe Gigabit Ethernet Controller (PCI ID: 10ec:8168)` that is not included in VMware's Hardware Compatibility List (HCL) causing the installer to terminate with the error:

```
No Network Adapters Detected
```

Multiple installation methods were evaluated, including direct ISO flashing, manual USB creation, post-install driver installation, and image customization. The successful solution involved building a custom ESXi installation image using VMware Image Builder (PowerCLI) by integrating the VMware Certified Realtek `if-re` driver into the ESXi image profile before installation.

---

# Table of Contents

tbd

---

# 1. Introduction

VMware ESXi is a bare-metal Type-1 hypervisor designed primarily for enterprise servers listed in VMware's Hardware Compatibility List (HCL).

Unlike desktop operating systems, ESXi includes only drivers that VMware explicitly validates and certifies.

Consumer HP desktops commonly use Realtek Ethernet controllers which are intentionally excluded from modern ESXi installation images.

Consequently, the installer cannot initialize the network interface and refuses installation.

---

# 2. System Specifications

---

## Network Controller

```bash
lspci -nnk | grep -A4 -i ethernet
```

Output

```
02:00.0 Ethernet controller
Realtek RTL8111/8168/8211 PCI Express Gigabit Ethernet Controller
PCI ID: 10ec:8168
Kernel driver: r8169
```

This PCI ID identifies the onboard Ethernet controller.

---

# 3. Initial Installation Attempt

The ESXi installer ISO was written directly to USB using

```bash
sudo dd if=VMware-VMvisor.iso of=/dev/sdb bs=4M status=progress conv=fsync
sync
```

The HP firmware did not recognize the USB as a bootable UEFI device.

---

# 4. Root Cause Analysis

Although `dd` performs a sector-for-sector copy, not every UEFI firmware properly recognizes hybrid ISO images written directly to removable media.

Enterprise servers generally boot these media correctly.

Consumer HP firmware proved significantly stricter.

---

# 5. USB Boot Issues

```
USB absent from Boot Menu
```

 => The firmware simply rejected the USB boot structure.

---

# UEFI Boot

# 6 Why Manual USB Construction Worked

Instead of cloning the ISO with `dd`, the USB was rebuilt manually.

Partition table - GPT - Partition - EFI System Partition - Filesystem - FAT32

Commands

```bash
sudo parted /dev/sdb --script \
mklabel gpt \
mkpart ESP fat32 1MiB 100% \
set 1 esp on
```

Formatting

```bash
sudo mkfs.vfat -F32 /dev/sdb1
```

Mounting

```bash
sudo mount -o loop VMware.iso ~/mnt/esxi
sudo mount /dev/sdb1 ~/mnt/usb
```

Copying

```bash
sudo rsync -avh ~/mnt/esxi/ ~/mnt/usb/
```

This produced a USB layout accepted by the HP UEFI firmware.

---

# 7. BIOS Configuration

Required settings

```
UEFI Boot          Enabled
Legacy Boot        Disabled
USB Boot           Enabled
Fast Boot          Disabled
Secure Boot        Disabled
```

Fast Boot may skip USB initialization.

Legacy Boot bypasses the EFI loader.

Secure Boot may reject unsigned drivers.

---

# 8. Installation Failure

The installer now booted correctly.

Installation failed with

```
No Network Adapters Detected
```

---

# 9. Root Cause

ESXi loads only drivers present inside its Image Profile.

The installer contained Intel, Broadcom, Mellanox, QLogic and other enterprise NIC drivers, but It did **not** contain

```
if-re
```

Therefore

```
PCI Scan
      │
      ▼
RTL8111 detected
      │
      ▼
No matching driver
      │
      ▼
Network stack unavailable
      │
      ▼
Installer aborts
```

---

# 10. Why Installing the Driver Afterwards Cannot Work

An attempted approach was

```
Install ESXi
↓
Install VIB afterwards
```

This is impossible because the installer itself aborts before ESXi is installed.

So no VIB installation can occur.

---

# 11. Why Copying the Driver into the ISO Does Not Work

Another possible idea was

```
ISO
├── b.b00
├── k.b00
├── vmx.v00
└── if-re.vib
```

This is invalid because A VIB is **not** a boot module but `boot.cfg` references VMkernel modules.

It does not install VIBs.

Simply copying a VIB changes neither

* Image Profile
* metadata
* package database
* dependency graph

The installer would ignore the file completely.

---

# 12. Correct Solution

The correct solution is Image Builder.

Input

```
ESXi Base Depot
+
Realtek Offline Depot
```

↓

```
Clone Image Profile
```

↓

```
Inject if-re
```

↓

```
Export New ISO
```

---

# 13. Building the Image

Input files

```
VMware-ESXi-8.0U3f-24784735-depot.zip

VMware-Re-Driver_1.101.01-5vmw.800.1.0.20613240.zip
```

PowerShell

```powershell
Add-EsxSoftwareDepot
```

loaded both depots.

The Image Profile was cloned.

```
ESXi-8.0U3f-24784735-standard
```

↓

```
ESXi-8.0U3f-24784735-Realtek
```

The driver

```
if-re
```

was inserted.

Verification

```powershell
Get-EsxSoftwarePackage -Name if-re
```

Result

```
VMwareCertified
```

Image export

```powershell
Export-EsxImageProfile -ExportToIso
```

Generated

```
ESXi-8.0U3f-24784735-Realtek.iso
```

---

# 14. Rebuilding the USB

The custom ISO replaced the original ISO, but the USB creation procedure remained identical.

GPT

↓

FAT32

↓

Copy files

↓

Boot.

---

# 15. Final Installation

Boot sequence

```
HP Firmware
↓

BOOTX64.EFI

↓

boot.cfg

↓

VMkernel

↓

Image Profile

↓

if-re Driver

↓

RTL8111 Initialized

↓

Installer Continues
```

The previous error disappeared and installation completed successfully.

---

# 16. Why the Driver Worked

Hardware `PCI ID 10ec:8168`

Driver `if-re`supports the RTL8111/8168 family.

During PCI enumeration

```
PCI Device
↓
Vendor ID = 10ec
↓
Device ID = 8168
↓
Driver Match
↓
Attach Driver
↓
Create vmnic0
```

The installer now detected

```
vmnic0
```

allowing installation to proceed.

---

# 17. Lessons Learned

* Consumer hardware is frequently unsupported by default ESXi images.
* The installer requires the NIC driver before installation begins.
* Offline depots are not equivalent to ISO images.
* VIBs cannot simply be copied into an ISO.
* Image Profiles define the complete driver inventory of the installer.
* PowerCLI Image Builder is the supported mechanism for driver integration.
* Consumer UEFI firmware may reject hybrid ISO media written with `dd`.
* Creating a FAT32 EFI System Partition manually provides greater compatibility with HP firmware.

---

# 18. Architecture Summary

```
Ubuntu
│
├── VMware ESXi Depot
│
├── Realtek Offline Depot
│
└── PowerCLI Image Builder
          │
          ▼
 Clone Image Profile
          │
          ▼
 Inject if-re Driver
          │
          ▼
 Export ISO
          │
          ▼
 GPT/FAT32 USB
          │
          ▼
 HP UEFI Firmware
          │
          ▼
 BOOTX64.EFI
          │
          ▼
 VMkernel
          │
          ▼
 if-re Driver
          │
          ▼
 RTL8111 Initialized
          │
          ▼
 ESXi Installation Successful
```

---

# Appendix A — Commands Used

## Hardware Identification

```bash
lspci -nnk | grep -A4 -i ethernet
```

---

## USB Preparation

```bash
sudo wipefs -a /dev/sdb

sudo parted /dev/sdb --script \
mklabel gpt \
mkpart ESP fat32 1MiB 100% \
set 1 esp on

sudo mkfs.vfat -F32 /dev/sdb1
```

---

## Mounting

```bash
sudo mount -o loop ESXi.iso ~/mnt/esxi
sudo mount /dev/sdb1 ~/mnt/usb
```

---

## Copy

```bash
sudo rsync -avh ~/mnt/esxi/ ~/mnt/usb/
```

---

## PowerCLI

```powershell
Add-EsxSoftwareDepot

Get-EsxImageProfile

New-EsxImageProfile

Add-EsxSoftwarePackage

Export-EsxImageProfile
```

---

# Conclusion

The successful solution required two independent modifications to the deployment process:

1. Constructing a UEFI-compatible USB using a GPT partition table, a FAT32 EFI System Partition, and manually copying the ISO contents to satisfy the HP firmware's boot requirements.
2. Building a custom ESXi installation image by integrating the VMware Certified `if-re` driver into the ESXi Image Profile using VMware Image Builder. This ensured that the VMkernel loaded the appropriate network driver during the installer initialization phase, allowing the RTL8111/8168 controller to be detected as `vmnic0`.

After both issues were resolved, the ESXi installer completed successfully and the unsupported HP desktop became a functional VMware ESXi host despite not being listed in VMware's Hardware Compatibility List.
