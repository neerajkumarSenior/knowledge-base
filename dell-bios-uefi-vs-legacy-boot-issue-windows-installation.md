# Dell BIOS UEFI vs Legacy Boot Issue (Windows Installation)

> A troubleshooting guide for future reference when Windows Installation USB boots in Legacy mode but not in UEFI mode.

---

# Problem

## Scenario

Dell Laptop/PC

Current Situation:

- Legacy Mode → Windows Installation USB boots successfully.
- UEFI Mode → Windows Installation USB does NOT boot.
- Instead of opening Windows Setup, the system directly boots into the existing Operating System.

Also received:

> Windows cannot be installed on this disk.
> The selected disk is of the GPT partition style.

---

# Expected Behaviour

When selecting **UEFI Boot** from the F12 Boot Menu, the Windows Installation screen should open.

Instead:

- Existing Windows starts automatically.
- USB installer is ignored.

---

# Investigation

The following settings were already verified:

- BIOS opened using `F2`
- Boot Mode changed to **UEFI**
- Secure Boot Enabled
- Secure Boot Disabled (also tested)
- F12 Boot Menu tested
- Legacy Boot tested
- USB detected in Legacy mode

Result:

Only Legacy Mode boots from USB.

---

# Root Cause

This happens because the installation USB was created in **Legacy (MBR)** mode.

When BIOS is switched to **UEFI**, Dell firmware only boots media containing a valid **UEFI bootloader**.

Since the USB was created as Legacy:

- BIOS ignores the USB
- Finds Windows Boot Manager on SSD
- Boots directly into the installed Windows

So the issue is **NOT BIOS**.

The issue is the **USB Boot Format**.

---

# BIOS Boot Compatibility

| BIOS Mode | USB Format |
|-----------|------------|
| Legacy | MBR |
| UEFI | GPT |

Correct combinations:

✅ Legacy BIOS + MBR USB

✅ UEFI BIOS + GPT USB

Wrong combinations:

❌ Legacy BIOS + GPT USB

❌ UEFI BIOS + MBR USB

---

# How to Verify

Open F12 Boot Menu.

If you see only:

```

Windows Boot Manager

```

and no

```

UEFI: USB Device

```

then the USB is not UEFI bootable.

---

# Solution 1 (Recommended)

Recreate the installation USB.

Using Rufus:

Partition Scheme:

```

GPT

```

Target System:

```

UEFI (non-CSM)

```

File System:

```

NTFS (or FAT32 if supported)

```

After recreating:

1. Insert USB
2. Press F12
3. Select

```

UEFI: <USB Name>

```

Windows Setup should start normally.

---

# Solution 2 (If You Want Legacy Installation)

Keep BIOS in Legacy mode.

When Windows shows:

> Windows cannot be installed because the disk is GPT.

Convert the disk to MBR.

⚠️ WARNING

This removes **ALL partitions and ALL data**.

Command Prompt:

```

Shift + F10

```

Commands:

```text
diskpart
list disk
select disk 0
clean
convert mbr
exit
```

Refresh the partition list.

Select:

```
Drive 0 Unallocated Space
```

Click **Next**.

Windows installation begins.

---

# Why Partition 4 Cannot Be Converted Alone

Many users think:

> Convert only Partition 4 to MBR.

This is impossible.

Reason:

Partition Style belongs to the **entire disk**, not an individual partition.

Disk can only be:

- GPT
- OR
- MBR

Not both.

---

# If Data Must Be Saved

Do NOT use:

```text
clean
```

Instead:

- Recreate USB as GPT
- Boot in UEFI
- Install Windows normally

This keeps the disk in GPT format.

---

# Important Commands

Open Command Prompt:

```text
Shift + F10
```

DiskPart:

```text
diskpart
```

Show disks:

```text
list disk
```

Select disk:

```text
select disk 0
```

Erase disk:

```text
clean
```

Convert to MBR:

```text
convert mbr
```

Convert to GPT:

```text
convert gpt
```

Exit:

```text
exit
```

---

# Common Mistakes

❌ Creating USB in MBR and trying to boot in UEFI.

❌ Enabling UEFI without recreating the USB.

❌ Thinking Secure Boot is always the problem.

❌ Trying to convert only one partition to MBR.

❌ Running `clean` without backing up data.

---

# Lessons Learned

- BIOS Mode and USB Format must always match.
- UEFI requires a GPT/UEFI bootable USB.
- Legacy requires an MBR bootable USB.
- GPT/MBR is a disk property, not a partition property.
- If installation USB is ignored in UEFI but works in Legacy, the USB was most likely created incorrectly.

---

# Final Conclusion

Problem:

```
USB boots only in Legacy.
UEFI boots existing Windows.
```

Root Cause:

```
USB was created in Legacy (MBR) format.
```

Recommended Fix:

```
Recreate the USB using GPT + UEFI in Rufus.
```

Alternative Fix:

```
Stay in Legacy mode
↓

Delete entire disk

↓

clean

↓

convert mbr

↓

Install Windows
```

---

## Reference Flow

```
Legacy BIOS
        │
        ▼
MBR USB
        │
        ▼
Windows Setup

----------------------------------

UEFI BIOS
        │
        ▼
GPT USB
        │
        ▼
Windows Setup

----------------------------------

UEFI BIOS
        │
        ▼
MBR USB
        │
        ▼
Ignored
        │
        ▼
Windows Boot Manager
        │
        ▼
Existing Windows Starts
```

---
Author: Neeraj Kumar

Purpose:
Future troubleshooting reference for Dell UEFI/Legacy Windows installation issues.
