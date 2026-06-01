# SPL (Secondary Program Loader) in Embedded Linux

## Overview

SPL stands for:

```text
Secondary Program Loader
```

It is a **small first-stage bootloader** used in embedded systems before the main bootloader (usually U-Boot).

SPL exists because:
- ROM code is very limited
- DRAM is not initialized yet
- full bootloaders are too large to load directly

SPL performs:
- minimal hardware initialization
- DDR/DRAM setup
- loading the main bootloader

---

# 1. Embedded Linux Boot Flow

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

# 2. Why SPL is Needed

At reset:
- DRAM is unavailable
- only small internal SRAM exists
- ROM code has limited capabilities

Main bootloader (U-Boot):
- too large
- requires DRAM

So SPL acts as a tiny intermediate stage.

---

# 3. SPL Responsibilities

| Responsibility | Description |
|----------------|-------------|
| CPU setup | Basic initialization |
| Clock initialization | PLL/clock setup |
| DDR initialization | Enable DRAM |
| Storage access | Read flash/eMMC/SD |
| Load U-Boot | Main bootloader loading |
| Basic security | Optional verification |

---

# 4. SPL Boot Flow

```text
ROM Code
    ↓
Load SPL into SRAM
    ↓
Execute SPL
    ↓
Initialize DDR
    ↓
Load U-Boot into DDR
    ↓
Jump to U-Boot
``` id="flow1"

---

# 5. Memory Before SPL

Initially available:

```text
Internal SRAM only
```

Example:
```text
64 KB SRAM
```

Not available:
```text
External DDR RAM
```

---

# 6. Why DRAM Initialization Matters

Without DDR:
- cannot load large bootloader
- cannot load kernel
- memory extremely limited

SPL initializes:
- DDR controller
- memory timings
- PHY settings

---

# 7. SPL and ROM Code Relationship

ROM Code loads SPL because:
- SPL is small
- fits inside SRAM
- easy for ROM to load

---

# 8. Typical Bootloader Stages

```text
ROM Code
   ↓
SPL
   ↓
U-Boot
   ↓
Linux Kernel
``` id="stage1"

---

# 9. SPL vs U-Boot

| Feature | SPL | U-Boot |
|---------|-----|---------|
| Size | Small | Large |
| Runs in SRAM | Yes | No |
| DDR init | Yes | Already initialized |
| Features | Minimal | Full bootloader |
| Filesystem support | Limited | Extensive |

---

# 10. SPL Internal Architecture

```text
SPL
 ├── CPU Init
 ├── Clock Init
 ├── DDR Init
 ├── Storage Driver
 └── U-Boot Loader
``` id="arch1"

---

# 11. SPL Storage Sources

SPL may load U-Boot from:

| Storage | Example |
|---------|---------|
| SD card | Raspberry Pi |
| eMMC | Embedded boards |
| NAND flash | Industrial devices |
| SPI flash | NOR/SPI systems |

---

# 12. SPL and SRAM

SPL executes entirely from:

```text
Internal SRAM
```

due to unavailable DDR.

---

# 13. SPL in U-Boot

U-Boot contains SPL support.

---

## Build Example

```bash
make spl
``` id="build1"

---

# 14. SPL Build Artifacts

| File | Purpose |
|------|---------|
| spl/u-boot-spl.bin | SPL binary |
| u-boot.bin | Main U-Boot |
| MLO | TI SPL image |

---

# 15. TI Sitara SPL Example

Boot flow:

```text
ROM
 ↓
MLO (SPL)
 ↓
U-Boot
 ↓
Kernel
``` id="ti1"

---

# 16. SPL and DDR Initialization

Most important SPL task.

Includes:
- DDR controller config
- timing calibration
- PHY training

---

# 17. DDR Init Flow

```text
SPL Start
    ↓
Clock Setup
    ↓
DDR Controller Config
    ↓
Memory Training
    ↓
DDR Available
``` id="ddr1"

---

# 18. SPL and Clock Initialization

SPL configures:
- PLLs
- CPU clocks
- peripheral clocks

Necessary before DDR access.

---

# 19. SPL and Pin Multiplexing

SPL may configure:
- UART pins
- DDR pins
- storage interface pins

---

# 20. SPL and UART Debugging

SPL often outputs logs via UART.

Example:

```text
U-Boot SPL 2024.01
Trying to boot from MMC1
``` id="uart1"

---

# 21. SPL and Secure Boot

SPL may:
- verify signatures
- check hashes
- authenticate U-Boot

---

# 22. Secure Boot Flow

```text
ROM verifies SPL
       ↓
SPL verifies U-Boot
       ↓
U-Boot verifies Kernel
``` id="sec1"

---

# 23. SPL and Boot Media Drivers

Minimal drivers:
- MMC/SD
- NAND
- SPI flash
- UART

Only essential functionality included.

---

# 24. SPL Filesystem Support

Usually limited.

Examples:
- FAT
- raw sectors

because SPL size must remain small.

---

# 25. SPL Image Loading

SPL loads:
```text
u-boot.img
```

into DDR memory.

---

# 26. SPL Memory Layout

```text
+----------------------+
| Internal SRAM        |
|----------------------|
| SPL Code             |
| SPL Stack            |
| Temporary Buffers    |
+----------------------+

+----------------------+
| External DDR         |
|----------------------|
| U-Boot               |
| Kernel               |
+----------------------+
``` id="mem1"

---

# 27. SPL Code Size Constraints

Typical SPL size:
```text
20 KB – 200 KB
```

Depends on:
- SRAM size
- SoC limitations

---

# 28. SPL in NAND Boot

Flow:

```text
ROM
 ↓
Read SPL from NAND
 ↓
SPL initializes DDR
 ↓
Read U-Boot from NAND
``` id="nand1"

---

# 29. SPL in SD Card Boot

Flow:

```text
ROM
 ↓
Read SPL sectors
 ↓
Run SPL
 ↓
Load U-Boot from FAT partition
``` id="sd1"

---

# 30. SPL and Device Tree

Some SPL versions support:
```text
Device Tree (DTB)
```

for hardware configuration.

---

# 31. SPL Source Tree in U-Boot

```text
u-boot/
 ├── spl/
 ├── arch/
 ├── drivers/
 ├── common/
 └── board/
```

---

# 32. SPL Build Configuration

Configured via:

```bash
make menuconfig
``` id="cfg1"

---

# 33. Important SPL Configurations

Examples:

```text
CONFIG_SPL
CONFIG_SPL_MMC
CONFIG_SPL_SERIAL
CONFIG_SPL_LIBCOMMON_SUPPORT
```

---

# 34. SPL and Linker Scripts

SPL uses linker scripts to:
- place code in SRAM
- control memory layout

---

# 35. SPL Bootloader Handoff

After loading U-Boot:

```c
jump_to_image_no_args();
``` id="jump1"

---

# 36. SPL and Stack Initialization

SPL must initialize:
- stack pointer
- temporary heap
- BSS section

before C code executes.

---

# 37. SPL C Runtime Initialization

Typical startup:

```text
Reset Vector
     ↓
Assembly Startup
     ↓
Stack Init
     ↓
BSS Clear
     ↓
main()
``` id="crt1"

---

# 38. SPL Error Handling

Possible failures:
- DDR init fail
- boot media missing
- corrupted U-Boot

---

# 39. SPL Recovery Methods

Fallback methods:
- UART boot
- USB recovery
- alternate media

---

# 40. SPL Debugging Techniques

---

## UART Logs

```text
DDR init done
Loading U-Boot...
``` id="dbg1"

---

## JTAG Debugging

Used for:
- early boot debugging
- memory inspection

---

# 41. SPL vs Full Bootloader

| Feature | SPL | Full U-Boot |
|---------|-----|-------------|
| Network support | Minimal | Full |
| Shell | No | Yes |
| Filesystem support | Basic | Extensive |
| Size | Tiny | Large |

---

# 42. SPL and Embedded Linux

SPL is critical in:
- ARM systems
- SoCs with DDR
- NAND/eMMC boot systems

---

# 43. Real-World SPL Examples

| Platform | SPL Usage |
|----------|-----------|
| BeagleBone | MLO |
| NXP i.MX | u-boot-spl.bin |
| Rockchip | miniloader/SPL |
| STM32MP1 | TF-A + SPL |

---

# 44. Complete Embedded Boot Chain

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
Init
``` id="boot2"

---

# 45. Advantages of SPL

- enables DDR initialization
- keeps ROM simple
- modular boot process
- supports large bootloaders

---

# 46. Disadvantages

- additional boot stage
- more complexity
- longer boot time

---

# 47. Summary

- SPL is a tiny first-stage bootloader
- Loaded by ROM code
- Runs from SRAM
- Initializes DDR memory
- Loads the full bootloader
- Essential in modern embedded Linux systems

---

````
