# How Linux Kernel Matches Device Tree Nodes to Drivers

## Overview

One of the MOST IMPORTANT concepts in:
- Embedded Linux
- Linux device drivers
- Device Tree (DTS/DTB)
- Platform drivers

is:

```text
How the Linux kernel matches
a Device Tree node to the correct driver
```

This mechanism enables:
- automatic hardware detection
- generic kernel support
- hardware abstraction
- reusable drivers

The matching process mainly uses:
```text
compatible strings
```

---

# 1. The Core Problem

Linux kernel must answer:

```text
Which driver should control which hardware device?
```

Example:
- UART hardware exists
- SPI controller exists
- GPIO controller exists

Kernel must load:
- UART driver for UART
- SPI driver for SPI
- GPIO driver for GPIO

---

# 2. Device Tree-Based Driver Matching

Linux solves this using:
```text
Device Tree compatible strings
```

---

# 3. High-Level Matching Flow

```text
DT Node
   ↓
compatible Property
   ↓
Kernel Searches Driver Tables
   ↓
Matching Driver Found
   ↓
probe() Function Called
``` id="flow1"

---

# 4. Device Tree Node Example

```dts
uart0: serial@101f1000 {
    compatible = "arm,pl011";
    reg = <0x101f1000 0x1000>;
    interrupts = <5>;
};
``` id="dts1"

---

# 5. Meaning of compatible

```dts
compatible = "arm,pl011";
```

means:
```text
This hardware is ARM PL011 UART
```

---

# 6. Driver Side Matching Table

Linux driver provides:
```c
of_match_table
```

---

# 7. Driver Match Table Example

```c
static const struct of_device_id pl011_ids[] = {
    { .compatible = "arm,pl011" },
    { }
};
``` id="drv1"

---

# 8. Kernel Matching Logic

Kernel compares:

```text
DT compatible string
```

with:

```text
Driver of_match_table entries
```

---

# 9. Full Architecture

```text
+----------------------------+
| Device Tree Node           |
| compatible="arm,pl011"     |
+----------------------------+
              ↓
+----------------------------+
| Kernel Device Model        |
+----------------------------+
              ↓
+----------------------------+
| Driver of_match_table      |
| "arm,pl011"                |
+----------------------------+
              ↓
+----------------------------+
| Driver probe()             |
+----------------------------+
``` id="arch1"

---

# 10. Boot-Time Sequence

```text
Bootloader Loads DTB
        ↓
Kernel Parses DTB
        ↓
Platform Devices Created
        ↓
Driver Matching Starts
        ↓
Matching Driver Probed
``` id="boot1"

---

# 11. Important Linux Components

| Component | Purpose |
|-----------|---------|
| DTB | Hardware description |
| Platform Device | Device instance |
| Platform Driver | Driver implementation |
| of_match_table | Matching table |

---

# 12. Step 1 — Kernel Parses DTB

During boot:
```text
kernel reads DTB
```

---

# 13. Step 2 — DT Nodes Discovered

Kernel discovers:
- UART nodes
- SPI nodes
- GPIO nodes

---

# 14. Step 3 — Platform Devices Created

Kernel converts DT nodes into:
```text
platform_device objects
```

---

# 15. Platform Device Creation Flow

```text
DT Node
   ↓
platform_device Created
``` id="pdev1"

---

# 16. Platform Device Example

Kernel internally creates:

```c
struct platform_device
```

for each DT node.

---

# 17. Step 4 — Driver Registration

Drivers register themselves using:
```c
platform_driver_register()
```

---

# 18. Platform Driver Example

```c
static struct platform_driver my_driver = {
    .probe = my_probe,
    .driver = {
        .name = "my_uart",
        .of_match_table = my_ids,
    },
};
``` id="plat1"

---

# 19. Step 5 — Matching Begins

Kernel compares:
```text
platform device ↔ platform driver
```

---

# 20. Matching Mechanism

Kernel checks:
```text
compatible property
```

against:
```text
of_match_table[]
```

---

# 21. Successful Match Example

---

## Device Tree

```dts
compatible = "vendor,myuart";
``` id="ok1"

---

## Driver

```c
{ .compatible = "vendor,myuart" }
``` id="ok2"

---

## Result

```text
MATCH SUCCESSFUL
```

---

# 22. Step 6 — probe() Called

If match succeeds:
```text
driver probe() executes
```

---

# 23. probe() Purpose

probe() initializes:
- MMIO registers
- interrupts
- clocks
- GPIOs
- DMA

---

# 24. probe() Flow

```text
Driver Matched
      ↓
probe() Called
      ↓
Resources Retrieved
      ↓
Hardware Initialized
``` id="probe1"

---

# 25. Resource Retrieval

Driver retrieves resources from DT.

---

# 26. Memory Resource Example

```dts
reg = <0x101f1000 0x1000>;
``` id="reg1"

---

# 27. Driver MMIO Mapping

```c
platform_get_resource()
devm_ioremap_resource()
``` id="mmio1"

---

# 28. Interrupt Resource Example

```dts
interrupts = <5>;
``` id="irq1"

---

# 29. Driver IRQ Retrieval

```c
platform_get_irq()
``` id="irq2"

---

# 30. Clock Resource Example

```dts
clocks = <&clkctrl 3>;
``` id="clk1"

---

# 31. GPIO Resource Example

```dts
reset-gpios = <&gpio0 5 GPIO_ACTIVE_LOW>;
``` id="gpio1"

---

# 32. Driver Retrieves GPIO

```c
gpiod_get()
``` id="gpio2"

---

# 33. Multiple Compatible Strings

DT may contain:
```dts
compatible = "vendor,new-uart",
             "ns16550a";
``` id="multi1"

---

# 34. Why Multiple Compatible Strings?

Provides:
- backward compatibility
- generic fallback support

---

# 35. Matching Priority

Kernel checks:
```text
compatible strings in order
```

Most specific first.

---

# 36. Match Table Terminator

```c
{ }
```

marks end of table.

---

# 37. MODULE_DEVICE_TABLE

```c
MODULE_DEVICE_TABLE(of, my_ids);
``` id="mod1"

Used for:
- module autoloading
- dependency management

---

# 38. Device Tree Match API

Kernel internally uses:
```c
of_match_device()
``` id="api1"

---

# 39. Internal Match Flow

```text
Read compatible
       ↓
Iterate of_match_table
       ↓
strcmp()
       ↓
Found Match
``` id="internal1"

---

# 40. Device Tree Node States

Node enabled using:

```dts
status = "okay";
``` id="status1"

---

# 41. Disabled Nodes

```dts
status = "disabled";
``` id="status2"

Kernel skips disabled devices.

---

# 42. Example Complete Matching

---

## DTS Node

```dts
uart0 {
    compatible = "vendor,myuart";
};
``` id="full1"

---

## Driver

```c
static const struct of_device_id my_ids[] = {
    { .compatible = "vendor,myuart" },
    { }
};
``` id="full2"

---

## Registration

```c
.driver = {
    .of_match_table = my_ids,
}
``` id="full3"

---

## Result

```text
probe() gets called
```

---

# 43. Linux Device Model Integration

Matching integrated with:
```text
Linux driver model
```

---

# 44. Platform Bus

Most DT devices appear on:
```text
platform bus
```

---

# 45. Platform Bus Architecture

```text
DT Node
    ↓
Platform Device
    ↓
Platform Driver
``` id="plat2"

---

# 46. Dynamic Module Loading

When DT node appears:
```text
kernel may auto-load matching module
```

---

# 47. Driver Probe Failure

Possible reasons:
- wrong compatible string
- missing driver
- missing resources

---

# 48. Debugging Driver Matching

---

## Check dmesg

```bash
dmesg
``` id="dbg1"

---

## View Device Tree

```bash
ls /proc/device-tree
``` id="dbg2"

---

## View Platform Devices

```bash
ls /sys/bus/platform/devices
``` id="dbg3"

---

## View Drivers

```bash
ls /sys/bus/platform/drivers
``` id="dbg4"

---

# 49. Common Matching Problems

| Problem | Description |
|---------|-------------|
| Wrong compatible string | No driver loaded |
| Missing of_match_table | DT matching fails |
| Node disabled | Device ignored |
| Missing MODULE_DEVICE_TABLE | Auto-loading fails |

---

# 50. Matching vs Initialization

Important:

Matching:
```text
find correct driver
```

Initialization:
```text
done inside probe()
```

---

# 51. Device Tree Driver Binding Flow

```text
DT Node Exists
      ↓
Kernel Creates platform_device
      ↓
Driver Registered
      ↓
Kernel Compares compatible Strings
      ↓
Match Found
      ↓
probe() Called
      ↓
Driver Initializes Hardware
``` id="final1"

---

# 52. Real Embedded Linux Example

```text
UART Hardware on SoC
        ↓
UART Node in DTB
        ↓
Kernel Parses compatible="arm,pl011"
        ↓
PL011 Driver Match Found
        ↓
probe() Called
        ↓
/dev/ttyAMA0 Created
``` id="real1"

---

# 53. Why This Mechanism is Powerful

Allows:
- generic kernels
- reusable drivers
- hardware abstraction
- portable BSPs

---

# 54. Key Kernel Structures

| Structure | Purpose |
|-----------|---------|
| platform_device | Device instance |
| platform_driver | Driver object |
| of_device_id | DT match table |

---

# 55. Final Core Concept

```text
Device Tree compatible strings
are the bridge between:
hardware and drivers
```

---

# 56. Summary

- Kernel parses DTB during boot
- DT nodes become platform devices
- Drivers provide of_match_table
- Kernel matches compatible strings
- Successful match triggers probe()
- probe() initializes hardware
- Core mechanism for Embedded Linux device initialization

---


````
