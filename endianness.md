# Endianness

## Overview

**Endianness** defines the order in which bytes are stored in memory for multi-byte data types (like 16-bit, 32-bit, 64-bit values).

It is critical in embedded systems because different CPUs store bytes differently, affecting:
- Memory interpretation
- Communication protocols
- File formats
- Cross-platform compatibility

---

# 1. What is Endianness?

A multi-byte value (example: `0x12345678`) is stored in memory as a sequence of bytes.

### Example value:
```

0x12345678

```

---

# 2. Big Endian vs Little Endian

## 2.1 Big Endian

### Definition
Most significant byte (MSB) stored at lowest memory address.

### Memory Layout

```

Address:   1000   1001   1002   1003
Value:      12     34     56     78

```

### Representation
```

0x12345678 → 12 34 56 78

```

### Used In
- Network protocols (TCP/IP)
- Some RISC processors
- Older architectures (PowerPC, SPARC)

---

## 2.2 Little Endian

### Definition
Least significant byte (LSB) stored at lowest memory address.

### Memory Layout

```

Address:   1000   1001   1002   1003
Value:      78     56     34     12

```

### Representation
```

0x12345678 → 78 56 34 12

```

### Used In
- x86 / x86_64 CPUs
- ARM (default mode in most systems)
- Embedded Linux systems

---

# 3. Visual Comparison

```

Big Endian:
MSB ------------------> LSB
12 34 56 78

Little Endian:
LSB ------------------> MSB
78 56 34 12

```id="endv1a"

---

# 4. Why Endianness Matters

## 4.1 Networking

Network byte order is always:

```

BIG ENDIAN

```

So conversion is needed:

```

htonl() → Host to Network Long
ntohl() → Network to Host Long

````id="net1b2"

---

## 4.2 Embedded Communication

Used in:
- UART
- SPI
- I2C
- CAN bus

If mismatched:
- Data corruption occurs

---

## 4.3 File Formats

Some formats enforce byte order:
- BMP images (little endian)
- PNG (big endian chunks)
- ELF files (architecture dependent)

---

# 5. CPU Endianness Types

## 5.1 Big Endian CPUs
- Store MSB first
- Easier human readability

## 5.2 Little Endian CPUs
- Store LSB first
- Efficient arithmetic operations

## 5.3 Bi-Endian CPUs
- Can switch modes
- Example: ARM architecture

---

# 6. Example in C

## 6.1 Detect Endianness

```c
#include <stdio.h>

int main() {
    unsigned int x = 0x1;
    char *c = (char*)&x;

    if (*c == 1)
        printf("Little Endian\n");
    else
        printf("Big Endian\n");

    return 0;
}
``` id="cend1a"

---

## 6.2 Byte Access Example

```c
#include <stdio.h>

int main() {
    unsigned int x = 0x12345678;
    unsigned char *p = (unsigned char*)&x;

    printf("%x %x %x %x\n", p[0], p[1], p[2], p[3]);

    return 0;
}
``` id="cend2b"

---

# 7. Byte Swapping

Used to convert between endianness.

## 7.1 Manual Swap

```c
unsigned int swap32(unsigned int val) {
    return ((val >> 24) & 0xff) |
           ((val << 8) & 0xff0000) |
           ((val >> 8) & 0xff00) |
           ((val << 24) & 0xff000000);
}
``` id="swap1c"

---

## 7.2 GCC Built-ins

```c
__builtin_bswap32(x);
__builtin_bswap64(x);
````

---

# 8. Endianness in Embedded Linux

## Common Reality

Most embedded Linux systems use:

```
Little Endian (ARM, x86)
```

## Exception Cases

* Network communication → Big Endian
* Bootloaders may switch modes
* Cross-platform firmware

---

# 9. Endianness Conversion Functions

## POSIX Standard

````c
htonl()   // host to network (32-bit)
htons()   // host to network (16-bit)
ntohl()   // network to host (32-bit)
ntohs()   // network to host (16-bit)
``` id="net2d3"

---

# 10. Memory Visualization Example

## Value: 0xA1B2C3D4

### Big Endian
````

Address → Value
1000 → A1
1001 → B2
1002 → C3
1003 → D4

```

### Little Endian
```

Address → Value
1000 → D4
1001 → C3
1002 → B2
1003 → A1

```

---

# 11. Common Bugs Due to Endianness

- Wrong sensor data interpretation
- Corrupted network packets
- Image format errors
- Binary file misreads
- Communication mismatch between MCU and PC

---

# 12. Debug Techniques

## Check memory via debugger
```

x/4xb address

```

## Print raw bytes in C
```

printf("%02x", byte);

```

## Use endian conversion macros
- Always use `htons`, `ntohl`, etc.

---

# 13. Summary

- Endianness defines byte order in memory
- Big endian: MSB first
- Little endian: LSB first
- Embedded Linux mostly uses little endian
- Networking always uses big endian
- Mismatch causes serious bugs

---


