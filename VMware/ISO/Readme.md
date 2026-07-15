### In this folder:
- The **Original Broadcom ISO image**, but it is missing the Realtek Driver for my NIC needed to deploy ESXi on my incompatible hardware. And we cannot simply add a VIB file into it:  `VMware-VMvisor-Installer-8.0U3e-246...`
- The Depot which contains all the necessary VIBs (drivers, software, vendor apps... but not realtek driver also): `VMware-ESXi-8.0U3f-24784735-depot.zip` \[the image profile dictates which VIBs to use in the image]
- The Realtek NIC driver: `VMware-Re-Driver_1.101.01-5vmw.800.1.0.20613240.zip`

##### All we need to do now is to inject the driver in the depot and build the image; using the vSphere image builder.

# Step 1 — Load ESXi depot

```powershell
Add-EsxSoftwareDepot `
/home/abok/Downloads/VMware-ESXi-8.0U3f-24784735-depot.zip
```

Now Image Builder knows:

- ESXi base packages

---

# Step 2 — Load Realtek depot

```powershell
Add-EsxSoftwareDepot `
/home/abok/Downloads/VMware-Re-Driver_1.101.01-5vmw.800.1.0.20613240.zip
```

Now Image Builder knows:

- ESXi packages
- Realtek `if-re` package

---

# Step 3 — Clone the VMware image profile

Original:

```text
ESXi-8.0U3f-24784735-standard
```

Clone:

```text
ESXi-8.0U3f-24784735-Realtek
```

## Why clone?

Because VMware profiles are immutable.

You don't modify:

```text
VMware standard profile
```

You create:

```text
your custom profile
```

---

# Step 4 — Inject driver

Image Builder operation:

```text
Standard Profile

        +

     if-re VIB

        ↓

Custom Profile
```

The resulting profile contains:

```text
ESXi-8.0U3f-24784735-Realtek

├── VMkernel
├── Storage drivers
├── Enterprise NIC drivers
├── if-re
└── Tools
```

---

# Step 5 — Export ISO

Image Builder generates:

```text
ESXi-8.0U3f-24784735-Realtek.iso
```

This is the new installer.

It now contains:

```text
BOOTX64.EFI

boot.cfg

VMkernel

+

if-re driver
```
