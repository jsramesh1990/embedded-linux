# Linux Kernel Loading Process in Embedded Linux

## Overview

Kernel loading is the process where the bootloader:
- loads the Linux kernel into RAM
- loads the Device Tree Blob (DTB)
- optionally loads initramfs
- passes boot arguments
- transfers execution control to the kernel

This stage occurs after:
- ROM Code
- SPL
- U-Boot initialization

and before:
- Root Filesystem mounting
- Init system startup

---

# 1. Complete Embedded Linux Boot Flow

```text
Power ON
    ↓
ROM Code
    ↓
SPL
    ↓
U-Boot
    ↓
Kernel Loading
    ↓
Kernel Initialization
    ↓
RootFS Mount
    ↓
Init System
``` id="boot1"

---

# 2. What is Kernel Loading?

Kernel loading means:
- reading kernel image from storage/network
- placing it into RAM
- preparing boot parameters
- starting execution

---

# 3. Components Loaded During Boot

| Component | Purpose |
|-----------|---------|
| Linux Kernel | Core operating system |
| DTB | Hardware description |
| initramfs | Temporary root filesystem |
| Boot arguments | Kernel configuration |

---

# 4. Typical Kernel Images

| Image Type | Architecture |
|------------|--------------|
| zImage | ARM compressed |
| Image | ARM64 |
| uImage | U-Boot legacy |
| bzImage | x86 compressed |

---

# 5. Bootloader Role

U-Boot performs:
- filesystem access
- storage driver initialization
- kernel image loading
- DTB loading
- boot parameter passing

---

# 6. Kernel Loading Flow

```text
U-Boot
   ↓
Read Kernel Image
   ↓
Load DTB
   ↓
Load initramfs (optional)
   ↓
Set bootargs
   ↓
Jump to Kernel Entry Point
``` id="flow1"

---

# 7. Kernel Memory Layout

Typical memory arrangement:

```text
+----------------------+
| Kernel Image         |
+----------------------+
| Device Tree Blob     |
+----------------------+
| initramfs            |
+----------------------+
| U-Boot               |
+----------------------+
``` id="mem1"

---

# 8. Kernel Image Locations

Common storage locations:

| Storage | Example |
|---------|---------|
| SD card | Raspberry Pi |
| eMMC | Embedded boards |
| NAND flash | Industrial systems |
| SPI flash | Small systems |
| Network (TFTP) | Development |

---

# 9. Loading from SD Card

Example:

```bash
fatload mmc 0:1 0x80000000 zImage
``` id="sd1"

---

# 10. Loading from ext4 Partition

```bash
ext4load mmc 0:2 0x80000000 Image
``` id="ext1"

---

# 11. Network Kernel Loading

Kernel can be loaded using:
- TFTP
- DHCP
- NFS boot

---

## TFTP Example

```bash
tftpboot 0x80000000 zImage
``` id="tftp1"

---

# 12. Device Tree Blob (DTB)

DTB contains:
- CPU info
- memory layout
- peripheral configuration
- interrupt mapping

---

# 13. DTB Loading Example

```bash
fatload mmc 0:1 0x82000000 board.dtb
``` id="dtb1"

---

# 14. Why DTB is Needed

Kernel binary is generic.

DTB provides:
```text
board-specific hardware information
```

---

# 15. initramfs Loading

Optional RAM-based filesystem.

---

## Example

```bash
fatload mmc 0:1 0x83000000 initramfs.cpio.gz
``` id="ram1"

---

# 16. Boot Arguments (bootargs)

Kernel parameters passed by bootloader.

---

## Example

```bash
setenv bootargs console=ttyS0 root=/dev/mmcblk0p2 rw
``` id="args1"

---

# 17. Common bootargs Parameters

| Parameter | Purpose |
|-----------|---------|
| console= | UART console |
| root= | Root filesystem |
| rw | Read-write rootfs |
| init= | Initial process |
| mem= | Memory limit |

---

# 18. Boot Command Types

| Command | Purpose |
|---------|---------|
| bootz | Boot zImage |
| bootm | Boot uImage |
| booti | Boot ARM64 Image |

---

# 19. bootz Example

```bash
bootz 0x80000000 - 0x82000000
``` id="bootz1"

---

## Parameters

| Parameter | Meaning |
|-----------|---------|
| kernel addr | zImage location |
| ramdisk addr | initramfs |
| dtb addr | DTB location |

---

# 20. bootm Example

```bash
bootm 0x80000000
``` id="bootm1"

Used for:
```text
uImage
```

---

# 21. booti Example

```bash
booti 0x80000000 - 0x82000000
``` id="booti1"

Used for:
```text
ARM64 Image
```

---

# 22. Kernel Decompression

Compressed kernels:
- zImage
- bzImage

must decompress themselves after loading.

---

# 23. Decompression Flow

```text
Compressed Kernel
      ↓
Decompressor
      ↓
Uncompressed Kernel
      ↓
Kernel Start
``` id="decomp1"

---

# 24. Kernel Entry Point

Bootloader jumps to:
```text
kernel_entry()
```

after loading.

---

# 25. CPU State Requirements

Before jumping:
- MMU configured properly
- CPU mode set
- interrupts disabled
- memory initialized

---

# 26. ARM Kernel Boot Requirements

ARM boot protocol requires:
- register setup
- DTB pointer
- machine ID (older kernels)

---

# 27. ARM64 Boot Requirements

ARM64 uses:
- x0 = DTB address
- EL level setup

before entering kernel.

---

# 28. Kernel Initialization Starts

After jump:

```text
start_kernel()
```

executes.

---

# 29. Kernel Initialization Flow

```text
start_kernel()
      ↓
Memory Init
      ↓
Scheduler Init
      ↓
Driver Init
      ↓
Mount RootFS
      ↓
Start init
``` id="kern1"

---

# 30. initramfs Boot Flow

```text
Kernel
   ↓
Mount initramfs
   ↓
Run /init
   ↓
Switch to real rootfs
``` id="initram1"

---

# 31. RootFS Mounting

Kernel mounts:
```text
root=/dev/xxx
```

from bootargs.

---

# 32. Kernel Loading from NAND

Flow:

```text
U-Boot
   ↓
Read NAND blocks
   ↓
Load Kernel to DDR
   ↓
bootm/bootz
``` id="nand1"

---

# 33. FIT Image Loading

FIT image may contain:
- kernel
- DTB
- ramdisk
- signatures

in one package.

---

# 34. FIT Boot Flow

```text
Load FIT Image
      ↓
Verify Signature
      ↓
Extract Components
      ↓
Boot Kernel
``` id="fit1"

---

# 35. Secure Kernel Loading

Secure boot chain:

```text
ROM verifies SPL
SPL verifies U-Boot
U-Boot verifies Kernel
``` id="sec1"

---

# 36. Common Kernel Load Addresses

| Component | Typical Address |
|-----------|----------------|
| Kernel | 0x80000000 |
| DTB | 0x82000000 |
| initramfs | 0x83000000 |

---

# 37. Example Full Boot Sequence

```bash
fatload mmc 0:1 0x80000000 zImage
fatload mmc 0:1 0x82000000 board.dtb
setenv bootargs console=ttyS0 root=/dev/mmcblk0p2 rw
bootz 0x80000000 - 0x82000000
``` id="full1"

---

# 38. Kernel Panic During Loading

Common issues:

---

## Wrong DTB

Symptoms:
```text
Kernel hangs early
```

---

## Wrong root= parameter

```text
VFS: unable to mount root fs
```

---

## Invalid memory address

```text
Data abort
```

---

# 39. Debugging Kernel Loading

---

## U-Boot Console

```bash
printenv
``` id="dbg1"

---

## Memory Inspection

```bash
md 0x80000000
``` id="dbg2"

---

## Kernel Logs

```bash
dmesg
``` id="dbg3"

---

# 40. Kernel Image Verification

Use:
```bash
file zImage
``` id="verify1"

---

# 41. Kernel Loading in Embedded Linux

Typical embedded setup:

```text
U-Boot
   ↓
Load zImage from eMMC
   ↓
Load DTB
   ↓
Pass bootargs
   ↓
Boot Kernel
``` id="emb1"

---

# 42. x86 Kernel Loading

x86 commonly uses:
- GRUB bootloader
- bzImage kernel

---

# 43. ARM Embedded Boot Example

```text
ROM
 ↓
SPL
 ↓
U-Boot
 ↓
zImage + DTB
 ↓
Kernel
``` id="arm1"

---

# 44. Relationship Between Components

| Component | Purpose |
|----------|---------|
| SPL | DDR initialization |
| U-Boot | Kernel loader |
| Kernel | Operating system |
| DTB | Hardware description |
| RootFS | Userspace |

---

# 45. Kernel Loading Internal Flow

```text
Storage Device
      ↓
Bootloader Reads Image
      ↓
Image Copied to RAM
      ↓
CPU Jump to Kernel
      ↓
Kernel Initialization
``` id="flow2"

---

# 46. Advantages of Bootloader-Based Loading

- flexible boot configuration
- network boot support
- multiple kernels
- secure boot
- recovery support

---

# 47. Summary

- Kernel loading is performed by U-Boot
- Kernel image loaded into RAM
- DTB and bootargs passed to kernel
- Kernel decompresses and initializes
- Root filesystem mounted afterward

---

````
