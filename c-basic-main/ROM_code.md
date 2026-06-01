# ROM Code in Embedded Systems 

## Overview

ROM Code is the **first software executed** when an embedded processor powers on or resets.

It is permanently stored inside the SoC (System on Chip) and is responsible for:
- hardware initialization
- boot source selection
- loading the first-stage bootloader
- secure boot verification (optional)

ROM code is also called:

```text
Boot ROM
Bootloader ROM
Internal ROM
Mask ROM
```

---

# 1. Embedded Boot Flow Overview

```text
Power ON / Reset
        ↓
CPU starts execution
        ↓
ROM Code executes
        ↓
Load First Stage Bootloader
        ↓
Initialize DRAM
        ↓
Load Second Stage Bootloader
        ↓
Load Linux Kernel
        ↓
Mount RootFS
        ↓
Start Init System
``` id="boot1"

---

# 2. What is ROM?

ROM stands for:

```text
Read Only Memory
```

Characteristics:
- non-volatile
- permanently programmed
- survives power loss

---

# 3. Why ROM Code is Needed

When CPU powers on:
- RAM is empty
- storage is inaccessible initially
- no OS exists yet

ROM code provides:
- initial startup logic
- boot source detection
- minimal hardware setup

---

# 4. ROM Code Location

Stored inside the SoC silicon itself.

Examples:
- ARM SoCs
- STM32 MCUs
- NXP i.MX
- TI Sitara
- Raspberry Pi SoCs

---

# 5. ROM Code Responsibilities

| Responsibility | Description |
|----------------|-------------|
| CPU initialization | Basic processor setup |
| Clock setup | Enable clocks |
| Boot source detection | NAND/SD/eMMC/UART |
| Security checks | Secure boot |
| Bootloader loading | Load SPL/U-Boot |
| Error recovery | Fallback boot |

---

# 6. ROM Boot Sequence

```text
Reset Vector
      ↓
ROM Code Start
      ↓
Initialize CPU
      ↓
Detect Boot Source
      ↓
Load Bootloader
      ↓
Jump to Bootloader
``` id="flow1"

---

# 7. Reset Vector

After reset:
CPU starts executing from a fixed address.

Example:

```text
0x00000000
```

or

```text
0xFFFF0000
```

depending on architecture.

---

# 8. ARM Boot Example

Typical ARM boot:

```text
CPU Reset
    ↓
Boot ROM
    ↓
SPL (Secondary Program Loader)
    ↓
U-Boot
    ↓
Linux Kernel
``` id="arm1"

---

# 9. ROM Code and Boot Media

ROM code can boot from:

| Boot Source | Description |
|-------------|-------------|
| SD card | removable media |
| eMMC | embedded storage |
| NAND flash | raw flash |
| NOR flash | execute-in-place |
| SPI flash | serial flash |
| UART/USB | recovery boot |

---

# 10. Boot Source Selection

Usually controlled by:
- boot pins
- eFuse settings
- OTP memory
- configuration resistors

---

# 11. Boot Pins Example

```text
BOOT0 = 1 → UART boot
BOOT0 = 0 → Flash boot
``` id="pins1"

---

# 12. ROM Code and SPL

ROM code often loads:

```text
SPL (Secondary Program Loader)
```

because:
- ROM is small
- DRAM not initialized yet

---

# 13. Why SPL Exists

ROM code usually cannot:
- initialize full DDR
- load large binaries

So SPL:
- initializes DRAM
- loads full bootloader

---

# 14. SPL Boot Flow

```text
ROM Code
    ↓
SPL
    ↓
DDR Initialization
    ↓
Load U-Boot
``` id="spl1"

---

# 15. U-Boot Boot Flow

```text
ROM Code
    ↓
SPL
    ↓
U-Boot
    ↓
Kernel
``` id="ub1"

---

# 16. ROM Code and Secure Boot

ROM may verify:
- signatures
- hashes
- certificates

before booting.

---

# 17. Secure Boot Flow

```text
ROM Code
     ↓
Verify Bootloader Signature
     ↓
Load Trusted Bootloader
     ↓
Continue Boot
``` id="sec1"

---

# 18. Chain of Trust

```text
ROM Code (trusted)
      ↓
Verify SPL
      ↓
Verify U-Boot
      ↓
Verify Kernel
``` id="trust1"

---

# 19. ROM Code Size Constraints

ROM is limited:
- fixed silicon area
- cannot be easily updated

Typical size:
```text
16 KB – 256 KB
```

---

# 20. ROM Code Limitations

ROM code usually:
- minimal filesystem support
- limited drivers
- limited protocols
- minimal memory support

---

# 21. ROM Recovery Modes

Many SoCs support recovery boot.

Examples:
- USB boot
- UART boot
- JTAG recovery

---

# 22. UART Boot Example

```text
PC Tool
   ↓
UART
   ↓
ROM Bootloader
   ↓
RAM Execution
``` id="uart1"

---

# 23. USB Recovery Boot

Used for:
- firmware flashing
- recovery
- development

---

# 24. ROM Code in Raspberry Pi

Raspberry Pi boot flow:

```text
Boot ROM
    ↓
bootcode.bin
    ↓
start.elf
    ↓
Kernel
``` id="pi1"

---

# 25. ROM Code in STM32

STM32 contains:
- internal boot ROM
- UART/USB flashing support

---

# 26. ROM Code in NXP i.MX

Boot flow:

```text
ROM
 ↓
SPL
 ↓
U-Boot
 ↓
Linux
``` id="nxp1"

---

# 27. ROM Code and DRAM

Initially:
- DRAM controller disabled
- SRAM used temporarily

ROM often uses:
```text
internal SRAM
```

before DDR init.

---

# 28. Internal SRAM Usage

Used for:
- stack
- SPL loading
- temporary buffers

---

# 29. ROM Code Error Handling

If boot fails:
- try alternate media
- enter recovery mode
- wait for UART/USB download

---

# 30. Multi-Boot Support

ROM may try:

```text
1. eMMC
2. SD card
3. USB
4. UART
``` id="multi1"

---

# 31. ROM Code Internal Flow

```text
CPU Reset
   ↓
Clock Setup
   ↓
Read Boot Configuration
   ↓
Initialize Storage Interface
   ↓
Load Boot Image
   ↓
Jump to Bootloader
``` id="flow2"

---

# 32. Bootloader Image Format

ROM may expect:
- raw binary
- signed image
- specific header format

---

# 33. ROM Code and Image Headers

Headers contain:
- image size
- load address
- checksum
- entry point

---

# 34. Entry Point

ROM jumps to:

```text
bootloader_entry()
```

after loading image.

---

# 35. ROM Code and CPU Modes

ROM may:
- configure MMU
- set privilege level
- initialize exception vectors

---

# 36. ROM Debugging

---

## UART Logs

Some SoCs print ROM messages:

```text
Booting from MMC...
``` id="dbg1"

---

## JTAG Debugging

Used for:
- low-level debugging
- stepping through boot code

---

# 37. Common ROM Boot Errors

---

## No Boot Device

```text
No bootable image found
```

---

## Invalid Header

```text
Bad image format
```

---

## Signature Failure

```text
Authentication failed
```

---

# 38. ROM vs Bootloader

| Component | Purpose |
|-----------|---------|
| ROM Code | Initial startup |
| SPL | DDR init |
| U-Boot | Full bootloader |
| Kernel | Operating system |

---

# 39. ROM Code Security

ROM is usually:
- immutable
- hardware protected
- trusted root

---

# 40. ROM Code and eFuse

eFuse stores:
- secure boot keys
- boot configuration
- device IDs

---

# 41. Embedded Linux Full Boot Chain

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
``` id="boot2"

---

# 42. Typical Embedded Linux Storage Layout

```text
Flash Memory
 ├── ROM Code (internal)
 ├── SPL
 ├── U-Boot
 ├── Kernel
 ├── DTB
 └── RootFS
``` id="layout1"

---

# 43. Advantages of ROM Code

- guaranteed startup
- secure root of trust
- hardware-independent boot
- recovery support

---

# 44. Disadvantages

- cannot easily update
- limited features
- fixed bugs difficult to patch

---

# 45. Real-World Examples

| Platform | ROM Boot |
|----------|-----------|
| STM32 | UART/USB boot |
| Raspberry Pi | SD card boot |
| TI Sitara | NAND/eMMC boot |
| NXP i.MX | SPL/U-Boot boot |

---

# 46. Summary

- ROM code is the first software executed after reset
- Stored permanently inside SoC
- Responsible for loading bootloader
- Critical part of embedded Linux boot process
- Forms the hardware root of trust

---

````
