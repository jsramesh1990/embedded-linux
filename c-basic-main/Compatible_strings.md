# Compatible Strings in Device Tree

## Overview

Compatible strings are one of the MOST IMPORTANT concepts in:
- Device Tree (DTS/DTB)
- Linux device drivers
- Embedded Linux
- kernel hardware matching

They are used by the Linux kernel to:
```text
match hardware devices with appropriate drivers
```

Without compatible strings:
```text
kernel cannot know which driver should control which hardware
```

Compatible strings are central to:
- driver probing
- hardware abstraction
- platform portability

---

# 1. What is a Compatible String?

A compatible string is:

```text
A textual identifier describing a hardware device
```

used in:
```text
Device Tree nodes
```

---

# 2. Purpose of Compatible Strings

Compatible strings allow:
```text
automatic driver-device matching
```

between:
- Device Tree
- Linux kernel drivers

---

# 3. High-Level Driver Matching Flow

```text
Device Tree Node
       ↓
compatible Property
       ↓
Kernel Searches Drivers
       ↓
Matching Driver Found
       ↓
Driver Probed
``` id="flow1"

---

# 4. Basic Compatible String Example

```dts
compatible = "arm,pl011";
``` id="comp1"

---

# 5. Where Compatible Strings are Used

Used in:
- UART devices
- SPI controllers
- I2C devices
- GPIO controllers
- DMA controllers
- clocks
- interrupt controllers

---

# 6. Device Tree Example

```dts
uart0: serial@101f1000 {
    compatible = "arm,pl011";
    reg = <0x101f1000 0x1000>;
};
``` id="dts1"

---

# 7. Linux Driver Matching Architecture

```text
+-------------------------+
| Device Tree Node        |
| compatible = "vendor,x" |
+-------------------------+
             ↓
+-------------------------+
| Linux Driver            |
| of_match_table          |
+-------------------------+
             ↓
+-------------------------+
| Driver Loaded           |
+-------------------------+
``` id="arch1"

---

# 8. Driver Matching Mechanism

Kernel compares:
```text
Device Tree compatible string
```

with:
```text
driver supported IDs
```

---

# 9. Driver Match Table

Linux drivers use:
```c
of_device_id
```

---

# 10. Example Driver Match Table

```c
static const struct of_device_id my_ids[] = {
    { .compatible = "vendor,mydevice" },
    { }
};
``` id="drv1"

---

# 11. Driver Registration

```c
MODULE_DEVICE_TABLE(of, my_ids);
``` id="drv2"

---

# 12. Matching Process

```text
Kernel Reads DTB
       ↓
Finds compatible Property
       ↓
Searches Driver Match Tables
       ↓
Calls Matching Driver Probe Function
``` id="flow2"

---

# 13. Probe Function

When match found:
```text
driver probe() executes
```

---

# 14. Probe Flow

```text
Driver Matched
      ↓
probe() Called
      ↓
Registers Mapped
      ↓
Interrupts Configured
      ↓
Device Initialized
``` id="probe1"

---

# 15. Compatible String Format

Standard format:

```text
"vendor,device"
```

Example:
```text
"arm,pl011"
```

---

# 16. Why Vendor Prefix Needed

Prevents:
```text
name collisions
```

between manufacturers.

---

# 17. Vendor Prefix Examples

| Vendor | Prefix |
|--------|---------|
| ARM | arm |
| Samsung | samsung |
| NXP | fsl |
| TI | ti |
| Xilinx | xlnx |

---

# 18. Device Name Examples

| Device | Compatible String |
|--------|------------------|
| UART | "arm,pl011" |
| GPIO | "vendor,gpio" |
| SPI | "vendor,spi" |

---

# 19. Multiple Compatible Strings

A node may contain:
```text
multiple compatible strings
```

---

# 20. Multiple String Example

```dts
compatible = "vendor,new-uart",
             "vendor,old-uart";
``` id="multi1"

---

# 21. Why Multiple Compatible Strings Used

Allows:
- backward compatibility
- fallback driver support

---

# 22. Matching Priority

Kernel checks:
```text
compatible strings in order
```

Most specific:
```text
first
```

Generic fallback:
```text
later
```

---

# 23. Compatibility Inheritance

Example:

```dts
compatible = "ti,am3352-uart",
             "ns16550a";
``` id="inherit1"

Meaning:
- TI-specific UART
- compatible with generic 16550 UART

---

# 24. Generic Driver Support

Generic drivers can support:
```text
multiple hardware variants
```

using fallback compatible strings.

---

# 25. Real Embedded Example

```dts
compatible = "xlnx,zynq-uart",
             "ns16550a";
``` id="real1"

---

# 26. Linux Driver Lookup Flow

```text
DT Compatible String
        ↓
of_match_table[]
        ↓
Matching Driver
        ↓
probe()
``` id="lookup1"

---

# 27. Device Tree Parsing

Kernel parses:
```text
DTB during boot
```

---

# 28. Kernel Internal APIs

Important APIs:
- of_match_device()
- of_find_node_by_name()
- of_property_read()

---

# 29. of_match_table

Located in:
```c
platform_driver
```

---

# 30. Platform Driver Example

```c
static struct platform_driver my_driver = {
    .driver = {
        .name = "mydriver",
        .of_match_table = my_ids,
    },
};
``` id="plat1"

---

# 31. Device Tree and Platform Bus

Many DT devices become:
```text
platform devices
```

inside Linux.

---

# 32. Device Creation Flow

```text
DT Node Parsed
      ↓
Platform Device Created
      ↓
Driver Match Performed
      ↓
probe() Called
``` id="pdev1"

---

# 33. Common Compatible String Examples

---

## UART

```dts
compatible = "arm,pl011";
``` id="uart1"

---

## GPIO

```dts
compatible = "vendor,gpio";
``` id="gpio1"

---

## SPI

```dts
compatible = "vendor,spi";
``` id="spi1"

---

## Ethernet

```dts
compatible = "vendor,ethernet";
``` id="eth1"

---

# 34. Device Tree Bindings

Bindings define:
```text
required compatible strings and properties
```

---

# 35. Device Tree Binding Documentation

Located in kernel source:

```text
Documentation/devicetree/bindings/
```

---

# 36. YAML Bindings

Modern kernels use:
```text
YAML binding files
```

---

# 37. Example YAML Binding

```yaml
compatible:
  enum:
    - vendor,mydevice
``` id="yaml1"

---

# 38. Invalid Compatible Strings

Incorrect strings cause:
- driver not loading
- device initialization failure

---

# 39. Debugging Compatible Strings

---

## Check Device Tree

```bash
cat /proc/device-tree/compatible
``` id="dbg1"

---

## Check Kernel Logs

```bash
dmesg
``` id="dbg2"

---

## View Loaded Devices

```bash
ls /sys/bus/platform/devices
``` id="dbg3"

---

# 40. Driver Probe Failure Example

```text
Compatible string mismatch
         ↓
No matching driver
         ↓
Device unavailable
``` id="fail1"

---

# 41. Device Tree Overlay Compatibility

Overlays also use:
```text
compatible strings
```

for matching target hardware.

---

# 42. Platform Compatibility

Compatible strings support:
- SoC portability
- board reuse
- kernel portability

---

# 43. Best Practices

- use vendor prefix
- use fallback strings
- follow kernel bindings
- maintain backward compatibility

---

# 44. Common Problems

| Problem | Description |
|---------|-------------|
| Wrong vendor prefix | Match failure |
| Typo in compatible | Driver not loaded |
| Missing fallback | Older drivers fail |

---

# 45. Device Tree Matching Example

---

## DTS Node

```dts
uart0 {
    compatible = "vendor,myuart";
};
``` id="dts2"

---

## Driver

```c
{ .compatible = "vendor,myuart" }
``` id="drv3"

---

## Result

```text
Driver Successfully Loaded
```

---

# 46. Compatible String Hierarchy

```text
Specific Device
      ↓
Generic Device
      ↓
Standard Device
``` id="hier1"

---

# 47. Embedded Linux Boot Flow

```text
Bootloader Loads DTB
       ↓
Kernel Parses compatible Strings
       ↓
Drivers Matched
       ↓
Hardware Initialized
``` id="boot1"

---

# 48. Complete Driver Matching Workflow

```text
DTS File
   ↓
compatible Property
   ↓
DTB Generated
   ↓
Kernel Parses DTB
   ↓
Searches Driver Match Tables
   ↓
Matching Driver Found
   ↓
probe() Called
``` id="final1"

---

# 49. Advantages of Compatible Strings

| Advantage | Description |
|-----------|-------------|
| Hardware abstraction | Yes |
| Driver portability | Yes |
| Board flexibility | Yes |
| Reusable kernels | Yes |

---

# 50. Summary

- Compatible strings identify hardware devices
- Linux uses them for driver matching
- Defined in Device Tree nodes
- Drivers provide matching tables
- Essential for Embedded Linux boot and hardware initialization
- Supports portability and hardware abstraction

---

````
