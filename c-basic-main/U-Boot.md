# U-Boot (Universal Bootloader) in Embedded Linux 

## Overview

U-Boot stands for:

```text
Universal Bootloader
```

It is the most widely used bootloader in embedded Linux systems.

U-Boot is responsible for:
- hardware initialization
- loading the Linux kernel
- loading Device Tree
- loading Root Filesystem
- passing boot arguments
- starting Linux

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
Linux Kernel
    ↓
Root Filesystem
    ↓
Init System
``` id="boot1"

---

# 2. What is a Bootloader?

A bootloader is software that:
- runs before the operating system
- initializes hardware
- loads the OS into memory
- transfers execution control

---

# 3. Why U-Boot is Needed

The Linux kernel cannot directly start after reset because:
- DRAM may not be initialized
- storage devices are inaccessible initially
- hardware setup is incomplete

U-Boot performs this initialization.

---

# 4. U-Boot Responsibilities

| Responsibility | Description |
|----------------|-------------|
| Hardware initialization | CPU, DRAM, clocks |
| Peripheral setup | UART, MMC, Ethernet |
| Load Linux kernel | From storage/network |
| Load DTB | Device Tree Blob |
| Boot arguments | Pass kernel parameters |
| Filesystem support | FAT/ext4/etc |
| Network boot | TFTP/NFS boot |
| Firmware updates | Flashing support |

---

# 5. U-Boot Internal Architecture

```text
U-Boot
 ├── CPU Init
 ├── DRAM Support
 ├── Device Drivers
 ├── Filesystem Support
 ├── Networking Stack
 ├── Shell Interface
 └── Boot Commands
``` id="arch1"

---

# 6. U-Boot Boot Flow

```text
ROM Code
    ↓
SPL
    ↓
U-Boot
    ↓
Load Kernel
    ↓
Load DTB
    ↓
Boot Linux
``` id="flow1"

---

# 7. U-Boot Stages

Modern U-Boot may contain:

| Stage | Purpose |
|-------|---------|
| SPL | DDR initialization |
| U-Boot proper | Full bootloader |

---

# 8. U-Boot Memory Usage

Runs from:
- DDR memory
- initialized by SPL

---

# 9. U-Boot Console

U-Boot provides an interactive shell via UART.

Example prompt:

```text
U-Boot>
```

---

# 10. Common U-Boot Commands

| Command | Purpose |
|---------|---------|
| help | List commands |
| printenv | Show environment |
| setenv | Set variables |
| saveenv | Save environment |
| bootm | Boot Linux image |
| bootz | Boot zImage |
| fatload | Load from FAT |
| ext4load | Load ext4 file |
| tftpboot | Network boot |

---

# 11. U-Boot Environment Variables

U-Boot stores configuration variables.

---

## Example

```bash
printenv
``` id="env1"

---

## Common Variables

| Variable | Purpose |
|----------|---------|
| bootcmd | Boot command |
| bootargs | Kernel parameters |
| ipaddr | Device IP |
| serverip | TFTP server IP |
| bootdelay | Boot timeout |

---

# 12. Example bootargs

```bash
setenv bootargs console=ttyS0 root=/dev/mmcblk0p2 rw
``` id="args1"

---

# 13. Boot Delay

U-Boot waits before autoboot:

```bash
setenv bootdelay 3
``` id="delay1"

---

# 14. Linux Kernel Loading

U-Boot loads:
- zImage
- Image
- uImage

into DDR memory.

---

# 15. Linux Boot Commands

---

## bootz

Boot compressed ARM zImage:

```bash
bootz 0x80000000 - 0x82000000
``` id="bootz1"

---

## bootm

Boot uImage:

```bash
bootm 0x80000000
``` id="bootm1"

---

# 16. Device Tree Blob (DTB)

DTB describes hardware to kernel.

U-Boot loads:
```text
.dtb file
```

before booting Linux.

---

# 17. Kernel Boot Flow with DTB

```text
U-Boot
   ↓
Load Kernel
   ↓
Load DTB
   ↓
Pass DTB address
   ↓
Kernel Boot
``` id="dtb1"

---

# 18. Filesystem Support

U-Boot supports:

| Filesystem | Support |
|-----------|---------|
| FAT | Yes |
| ext2/ext4 | Yes |
| UBIFS | Yes |
| squashfs | Limited |

---

# 19. Storage Device Support

| Device | Support |
|--------|---------|
| SD card | Yes |
| eMMC | Yes |
| NAND | Yes |
| SPI NOR | Yes |
| USB storage | Yes |

---

# 20. fatload Example

```bash
fatload mmc 0:1 0x80000000 zImage
``` id="fat1"

---

# 21. ext4load Example

```bash
ext4load mmc 0:2 0x80000000 Image
``` id="ext1"

---

# 22. U-Boot Networking

Supports:
- TFTP
- DHCP
- NFS boot
- ping

---

# 23. TFTP Boot Example

```bash
tftpboot 0x80000000 zImage
``` id="tftp1"

---

# 24. NFS Root Boot

```text
Kernel from TFTP
RootFS from NFS
``` id="nfs1"

---

# 25. Ethernet Initialization

U-Boot initializes:
- MAC controller
- PHY
- network stack

---

# 26. U-Boot Shell Example

```bash
U-Boot> help
U-Boot> mmc list
U-Boot> printenv
``` id="shell1"

---

# 27. U-Boot Image Formats

| Format | Purpose |
|--------|---------|
| uImage | Legacy image |
| zImage | Compressed ARM kernel |
| Image | ARM64 kernel |
| FIT image | Modern image format |

---

# 28. FIT Images

FIT = Flattened Image Tree.

Supports:
- kernel
- DTB
- ramdisk
- signatures

in one image.

---

# 29. U-Boot and Secure Boot

Supports:
- signed images
- RSA verification
- secure boot chain

---

# 30. Secure Boot Flow

```text
ROM verifies SPL
      ↓
SPL verifies U-Boot
      ↓
U-Boot verifies Kernel
``` id="sec1"

---

# 31. U-Boot and Flashing

Used for:
- firmware updates
- flashing images
- recovery

---

## Example NAND Write

```bash
nand write 0x80000000 0x0 0x400000
``` id="nand1"

---

# 32. U-Boot Environment Storage

Environment stored in:
- NAND flash
- SPI flash
- eMMC
- SD card

---

# 33. saveenv Command

```bash
saveenv
``` id="save1"

Stores environment permanently.

---

# 34. U-Boot Build System

Built using:
```bash
make
```

---

## Example

```bash
make am335x_evm_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf-
``` id="build1"

---

# 35. U-Boot Source Tree

```text
u-boot/
 ├── arch/
 ├── board/
 ├── cmd/
 ├── common/
 ├── drivers/
 ├── fs/
 ├── net/
 └── include/
```

---

# 36. U-Boot Configuration

Uses:
```text
Kconfig
menuconfig
```

similar to Linux kernel.

---

# 37. Important U-Boot Configurations

Examples:

```text
CONFIG_MMC
CONFIG_CMD_NET
CONFIG_CMD_EXT4
CONFIG_BOOTDELAY
```

---

# 38. U-Boot Startup Sequence

```text
Reset
  ↓
CPU Init
  ↓
Board Init
  ↓
DRAM Init
  ↓
Peripheral Init
  ↓
Environment Load
  ↓
Autoboot
``` id="start1"

---

# 39. Autoboot Process

```text
bootdelay timeout
       ↓
bootcmd executes
``` id="auto1"

---

# 40. Example bootcmd

```bash
setenv bootcmd 'fatload mmc 0:1 0x80000000 zImage; bootz'
``` id="cmd1"

---

# 41. U-Boot and Device Drivers

Contains lightweight drivers:
- MMC
- USB
- Ethernet
- SPI
- NAND

---

# 42. U-Boot and Memory Map

Typical memory usage:

```text
+----------------------+
| Kernel               |
+----------------------+
| DTB                  |
+----------------------+
| Initramfs            |
+----------------------+
| U-Boot               |
+----------------------+
``` id="mem1"

---

# 43. U-Boot and Initramfs

Can load:
```text
initramfs.cpio.gz
```

along with kernel.

---

# 44. Debugging U-Boot

---

## UART Console

Most common debugging method.

---

## Print Memory

```bash
md 0x80000000
``` id="dbg1"

---

## Print Environment

```bash
printenv
``` id="dbg2"

---

# 45. Common U-Boot Problems

---

## Bad CRC

```text
Bad CRC, using default environment
```

---

## Wrong Kernel Address

```text
Starting kernel ...
```

(system hangs)

---

## Filesystem Read Error

```text
MMC read failed
```

---

# 46. U-Boot vs SPL

| Feature | SPL | U-Boot |
|---------|-----|---------|
| Size | Small | Large |
| Runs in SRAM | Yes | No |
| Shell | No | Yes |
| Networking | Minimal | Full |
| Filesystems | Limited | Advanced |

---

# 47. Embedded Linux Complete Boot Chain

```text
Power ON
   ↓
ROM Code
   ↓
SPL
   ↓
U-Boot
   ↓
Kernel
   ↓
RootFS
   ↓
Init System
``` id="boot2"

---

# 48. Advantages of U-Boot

- flexible
- highly configurable
- supports many architectures
- network boot support
- filesystem support
- recovery tools

---

# 49. Supported Architectures

| Architecture | Supported |
|--------------|-----------|
| ARM | Yes |
| ARM64 | Yes |
| RISC-V | Yes |
| PowerPC | Yes |
| MIPS | Yes |
| x86 | Yes |

---

# 50. Summary

- U-Boot is the most popular embedded Linux bootloader
- Loads and starts Linux kernel
- Provides hardware initialization
- Supports filesystems, networking, flashing
- Essential component of embedded Linux boot flow

---

````
