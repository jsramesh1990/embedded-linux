# U-Boot Boot Sequence in Embedded Linux (Complete Deep Guide)

## Overview

The U-Boot boot sequence is the complete process followed by the U-Boot bootloader to:
- initialize hardware
- load the Linux kernel
- load Device Tree
- load initramfs
- pass boot parameters
- start the Linux kernel

U-Boot is one of the most important components in embedded Linux systems.

---

# 1. Complete Embedded Linux Boot Chain

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
Root Filesystem Mount
    ↓
Init System
``` id="boot1"

---

# 2. Position of U-Boot in Boot Flow

U-Boot executes:
- after SPL
- before Linux kernel

It acts as:
```text
main bootloader
```

---

# 3. High-Level U-Boot Boot Sequence

```text
SPL loads U-Boot
        ↓
U-Boot starts
        ↓
Hardware Initialization
        ↓
Load Environment Variables
        ↓
Initialize Devices
        ↓
Autoboot Countdown
        ↓
Load Kernel + DTB
        ↓
Pass bootargs
        ↓
Boot Linux
``` id="flow1"

---

# 4. U-Boot Responsibilities

| Responsibility | Description |
|----------------|-------------|
| CPU initialization | Setup processor |
| DRAM usage | Use initialized DDR |
| Device initialization | MMC, USB, Ethernet |
| Filesystem support | FAT/ext4 |
| Kernel loading | zImage/uImage/Image |
| DTB loading | Hardware description |
| Networking | TFTP/DHCP |
| Recovery | Flash/update support |

---

# 5. U-Boot Startup Stages

Typical startup stages:

```text
Reset Vector
     ↓
Assembly Startup
     ↓
Board Init
     ↓
Relocation
     ↓
Driver Init
     ↓
Environment Load
     ↓
Command Execution
``` id="stage1"

---

# 6. CPU Reset State

After SPL jumps to U-Boot:
- CPU enters bootloader entry point
- stack initialized
- C runtime prepared

---

# 7. Assembly Startup Code

Initial low-level setup:
- stack pointer
- BSS clearing
- CPU mode setup

---

# 8. C Runtime Initialization

U-Boot prepares:
- global variables
- memory sections
- heap
- stack

before executing C code.

---

# 9. Board Initialization

Board-specific initialization:

```text
board_init()
```

Tasks:
- clocks
- UART
- GPIO
- timers

---

# 10. DRAM Initialization

Usually already initialized by SPL.

U-Boot:
- verifies DRAM
- configures memory usage

---

# 11. Console Initialization

UART initialized for:
```text
U-Boot console
```

Example output:

```text
U-Boot 2024.01
``` id="uart1"

---

# 12. Memory Relocation

U-Boot relocates itself:
- from temporary location
- to final DDR address

---

# 13. Relocation Flow

```text
Initial Load Address
        ↓
Copy U-Boot to DDR
        ↓
Update Pointers
        ↓
Continue Execution
``` id="rel1"

---

# 14. Driver Initialization

U-Boot initializes:
- MMC
- Ethernet
- USB
- SPI
- NAND

---

# 15. Environment Variable Loading

Environment variables loaded from:
- eMMC
- NAND
- SPI flash
- SD card

---

# 16. Environment Variables

Important variables:

| Variable | Purpose |
|----------|---------|
| bootcmd | Boot command |
| bootargs | Kernel arguments |
| bootdelay | Timeout |
| ipaddr | Device IP |
| serverip | TFTP server |

---

# 17. printenv Example

```bash
printenv
``` id="env1"

---

# 18. bootdelay Countdown

U-Boot waits before autoboot.

Example:

```text
Hit any key to stop autoboot: 3
``` id="delay1"

---

# 19. Interrupting Autoboot

User can:
- stop automatic boot
- enter U-Boot shell

---

# 20. U-Boot Shell

Interactive command interface:

```bash
U-Boot>
```

---

# 21. Common U-Boot Commands

| Command | Purpose |
|---------|---------|
| help | List commands |
| setenv | Set variables |
| saveenv | Save variables |
| fatload | Load file |
| ext4load | Load ext4 file |
| bootz | Boot zImage |
| bootm | Boot uImage |

---

# 22. Kernel Loading Stage

U-Boot loads:
- Linux kernel
- DTB
- initramfs

into RAM.

---

# 23. Kernel Loading Flow

```text
Read Storage Device
       ↓
Copy Kernel to RAM
       ↓
Load DTB
       ↓
Load initramfs
``` id="kern1"

---

# 24. Loading Kernel from SD Card

Example:

```bash
fatload mmc 0:1 0x80000000 zImage
``` id="sd1"

---

# 25. Loading Device Tree

Example:

```bash
fatload mmc 0:1 0x82000000 board.dtb
``` id="dtb1"

---

# 26. Loading initramfs

Example:

```bash
fatload mmc 0:1 0x83000000 initramfs.cpio.gz
``` id="ram1"

---

# 27. Setting bootargs

Example:

```bash
setenv bootargs console=ttyS0 root=/dev/mmcblk0p2 rw
``` id="args1"

---

# 28. Linux Boot Commands

| Command | Purpose |
|---------|---------|
| bootz | ARM zImage |
| bootm | uImage |
| booti | ARM64 Image |

---

# 29. bootz Example

```bash
bootz 0x80000000 0x83000000 0x82000000
``` id="bootz1"

---

# 30. bootm Example

```bash
bootm 0x80000000
``` id="bootm1"

---

# 31. booti Example

```bash
booti 0x80000000 - 0x82000000
``` id="booti1"

---

# 32. Device Tree Passing

U-Boot passes DTB address to kernel.

Kernel uses DTB for:
- hardware discovery
- driver initialization

---

# 33. initramfs Passing

Kernel receives:
```text
ramdisk address
```

from bootloader.

---

# 34. CPU State Before Kernel Jump

Before jumping:
- interrupts disabled
- MMU configured correctly
- registers prepared

---

# 35. Jumping to Kernel

U-Boot transfers control:

```text
boot_jump_linux()
```

---

# 36. Linux Kernel Entry

Kernel starts executing:

```text
start_kernel()
```

---

# 37. U-Boot Network Boot Sequence

```text
Initialize Ethernet
        ↓
DHCP Request
        ↓
TFTP Download
        ↓
Load Kernel
        ↓
Boot Linux
``` id="net1"

---

# 38. TFTP Boot Example

```bash
dhcp
tftpboot 0x80000000 zImage
bootz
``` id="tftp1"

---

# 39. NFS Root Boot

```text
Kernel from TFTP
RootFS from NFS
``` id="nfs1"

---

# 40. U-Boot Filesystem Support

Supported filesystems:

| Filesystem | Support |
|-----------|---------|
| FAT | Yes |
| ext2/ext4 | Yes |
| UBIFS | Yes |
| SquashFS | Limited |

---

# 41. U-Boot Storage Support

| Storage | Support |
|---------|---------|
| SD card | Yes |
| eMMC | Yes |
| NAND | Yes |
| SPI NOR | Yes |
| USB Storage | Yes |

---

# 42. U-Boot Memory Layout

```text
+----------------------+
| Kernel               |
+----------------------+
| DTB                  |
+----------------------+
| initramfs            |
+----------------------+
| U-Boot               |
+----------------------+
``` id="mem1"

---

# 43. U-Boot Environment Storage

Environment stored in:
- NAND
- SPI flash
- eMMC
- SD card

---

# 44. saveenv Example

```bash
saveenv
``` id="save1"

---

# 45. Secure Boot Sequence

```text
ROM verifies SPL
      ↓
SPL verifies U-Boot
      ↓
U-Boot verifies Kernel
``` id="sec1"

---

# 46. Common U-Boot Errors

---

## Bad CRC

```text
Bad CRC, using default environment
```

---

## MMC Failure

```text
MMC init failed
```

---

## Wrong Kernel Address

```text
Starting kernel...
```

(system hangs)

---

# 47. U-Boot Debugging

---

## UART Console

Primary debugging interface.

---

## Memory Dump

```bash
md 0x80000000
``` id="dbg1"

---

## Environment Debugging

```bash
printenv
``` id="dbg2"

---

# 48. Complete U-Boot Boot Example

```bash
fatload mmc 0:1 0x80000000 zImage
fatload mmc 0:1 0x82000000 board.dtb
fatload mmc 0:1 0x83000000 initramfs.cpio.gz

setenv bootargs console=ttyS0 root=/dev/mmcblk0p2 rw

bootz 0x80000000 0x83000000 0x82000000
``` id="full1"

---

# 49. Full Embedded Linux Boot Sequence

```text
Power ON
   ↓
ROM Code
   ↓
SPL
   ↓
U-Boot Initialization
   ↓
Load Kernel
   ↓
Load DTB
   ↓
Load initramfs
   ↓
Pass bootargs
   ↓
Jump to Linux Kernel
   ↓
Kernel Initialization
   ↓
Mount RootFS
   ↓
Start init
``` id="final1"

---

# 50. Advantages of U-Boot

- highly configurable
- supports many architectures
- networking support
- recovery features
- filesystem support
- scripting support

---

# 51. Summary

- U-Boot is the main embedded Linux bootloader
- Executes after SPL
- Initializes hardware and devices
- Loads kernel, DTB, and initramfs
- Passes boot parameters to Linux
- Transfers execution control to kernel

---

````
