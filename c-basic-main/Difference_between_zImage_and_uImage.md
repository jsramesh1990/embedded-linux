# Difference Between zImage and uImage in Embedded Linux 

## Overview

`zImage` and `uImage` are Linux kernel image formats used during the boot process in embedded Linux systems.

They are commonly used with:
- ARM systems
- U-Boot bootloader
- embedded Linux devices

Both contain compressed Linux kernels, but they differ in:
- image format
- metadata
- bootloader compatibility
- headers

---

# 1. High-Level Boot Flow

```text
Power ON
    ↓
ROM Code
    ↓
SPL
    ↓
U-Boot
    ↓
Load zImage/uImage
    ↓
Kernel Decompression
    ↓
Linux Boot
``` id="boot1"

---

# 2. What is zImage?

`zImage` is:
- a compressed Linux kernel image
- self-decompressing
- generated directly by Linux kernel build system

Commonly used on:
```text
ARM systems
```

---

# 3. What is uImage?

`uImage` is:
- a U-Boot wrapped kernel image
- contains a special U-Boot header
- created from zImage/Image

Used specifically with:
```text
U-Boot bootloader
```

---

# 4. Main Difference

```text
uImage = zImage + U-Boot Header
```

---

# 5. zImage Architecture

```text
+----------------------+
| Compressed Kernel    |
| Self-Decompressor    |
+----------------------+
``` id="zarch1"

---

# 6. uImage Architecture

```text
+----------------------+
| U-Boot Header        |
+----------------------+
| Compressed Kernel    |
| Self-Decompressor    |
+----------------------+
``` id="uarch1"

---

# 7. Why uImage Exists

U-Boot originally required:
- metadata
- load address
- checksum
- image type information

Thus:
```text
uImage format was created
```

---

# 8. U-Boot Header Contents

uImage header contains:

| Field | Purpose |
|------|---------|
| Magic number | Image validation |
| Load address | RAM location |
| Entry point | Start execution |
| Checksum | Integrity check |
| OS type | Linux/other |
| CPU architecture | ARM/MIPS/etc |

---

# 9. zImage Boot Flow

```text
U-Boot
   ↓
Load zImage
   ↓
bootz command
   ↓
Kernel self-decompresses
   ↓
Linux starts
``` id="zflow1"

---

# 10. uImage Boot Flow

```text
U-Boot
   ↓
Load uImage
   ↓
Verify Header
   ↓
bootm command
   ↓
Kernel decompresses
   ↓
Linux starts
``` id="uflow1"

---

# 11. File Locations

Kernel build outputs:

```text
arch/arm/boot/zImage
```

uImage generated separately.

---

# 12. zImage Generation

Generated during kernel build.

Example:

```bash
make zImage
``` id="zmake1"

---

# 13. uImage Generation

Generated using:

```text
mkimage
```

tool from U-Boot.

---

# 14. mkimage Example

```bash
mkimage -A arm -O linux -T kernel -C none \
-a 0x80008000 -e 0x80008000 \
-n "Linux Kernel" \
-d zImage uImage
``` id="mk1"

---

# 15. Important mkimage Parameters

| Option | Meaning |
|--------|---------|
| -A | Architecture |
| -O | Operating system |
| -T | Image type |
| -C | Compression |
| -a | Load address |
| -e | Entry point |

---

# 16. zImage Boot Command

```bash
bootz 0x80000000
``` id="bootz1"

---

# 17. uImage Boot Command

```bash
bootm 0x80000000
``` id="bootm1"

---

# 18. bootz vs bootm

| Command | Used For |
|---------|-----------|
| bootz | zImage |
| bootm | uImage |
| booti | ARM64 Image |

---

# 19. Kernel Compression

Both usually contain:
```text
compressed kernel
```

Compression methods:
- gzip
- xz
- lz4

---

# 20. Self-Decompression

zImage and uImage:
- contain decompressor code
- expand kernel into RAM during boot

---

# 21. Kernel Memory Layout

```text
+----------------------+
| Compressed Kernel    |
+----------------------+
| Decompressed Kernel  |
+----------------------+
| DTB                  |
+----------------------+
``` id="mem1"

---

# 22. zImage Advantages

| Advantage | Description |
|-----------|-------------|
| Simpler | No extra header |
| Smaller | Less overhead |
| Native Linux format | Direct kernel output |

---

# 23. uImage Advantages

| Advantage | Description |
|-----------|-------------|
| Metadata support | Header information |
| Integrity checks | CRC validation |
| Better U-Boot integration | Traditional support |

---

# 24. zImage Disadvantages

| Issue | Description |
|------|-------------|
| No metadata | Limited boot info |
| Older U-Boot incompatibility | Some boards require uImage |

---

# 25. uImage Disadvantages

| Issue | Description |
|------|-------------|
| Larger size | Header overhead |
| Additional creation step | mkimage needed |

---

# 26. Modern U-Boot Usage

Modern U-Boot supports:
```text
zImage directly
```

thus uImage less common now.

---

# 27. ARM64 Difference

ARM64 usually uses:

```text
Image
```

instead of zImage/uImage.

---

# 28. ARM64 Boot Command

```bash
booti
``` id="booti1"

---

# 29. zImage Example Flow

```bash
fatload mmc 0:1 0x80000000 zImage
bootz 0x80000000
``` id="zex1"

---

# 30. uImage Example Flow

```bash
fatload mmc 0:1 0x80000000 uImage
bootm 0x80000000
``` id="uex1"

---

# 31. DTB with zImage

```bash
bootz 0x80000000 - 0x82000000
``` id="zdt1"

---

# 32. DTB with uImage

```bash
bootm 0x80000000 - 0x82000000
``` id="udt1"

---

# 33. FIT Images (Modern Alternative)

Modern systems often use:

```text
FIT Image
```

which combines:
- kernel
- DTB
- initramfs
- signatures

---

# 34. FIT Image Structure

```text
FIT Image
 ├── Kernel
 ├── DTB
 ├── initramfs
 └── Signatures
``` id="fit1"

---

# 35. zImage Internal Structure

```text
zImage
 ├── Boot Header
 ├── Decompressor
 └── Compressed Kernel
``` id="zstruct1"

---

# 36. uImage Internal Structure

```text
uImage
 ├── U-Boot Header
 ├── Boot Header
 ├── Decompressor
 └── Compressed Kernel
``` id="ustruct1"

---

# 37. Checking Image Type

---

## file Command

```bash
file zImage
``` id="file1"

---

## dumpimage Command

```bash
dumpimage -l uImage
``` id="dump1"

---

# 38. Common Errors

---

## Wrong boot command

```text
Wrong Image Format
```

Occurs if:
- using bootm for zImage
- using bootz for uImage

---

## Bad CRC

```text
Bad Data CRC
```

uImage header corruption.

---

# 39. U-Boot Header Verification

U-Boot checks:
- magic number
- CRC
- architecture
- image type

before booting uImage.

---

# 40. Embedded Linux Typical Usage

Older systems:
```text
uImage
```

Modern systems:
```text
zImage or FIT
```

---

# 41. Complete Boot Sequence Using zImage

```text
ROM Code
   ↓
SPL
   ↓
U-Boot
   ↓
Load zImage
   ↓
bootz
   ↓
Kernel Decompression
   ↓
Linux Boot
``` id="zfull1"

---

# 42. Complete Boot Sequence Using uImage

```text
ROM Code
   ↓
SPL
   ↓
U-Boot
   ↓
Load uImage
   ↓
Header Verification
   ↓
bootm
   ↓
Kernel Decompression
   ↓
Linux Boot
``` id="ufull1"

---

# 43. Comparison Table

| Feature | zImage | uImage |
|---------|---------|---------|
| Compression | Yes | Yes |
| U-Boot Header | No | Yes |
| Created By | Kernel build | mkimage |
| Boot Command | bootz | bootm |
| Metadata | Minimal | Extensive |
| CRC Validation | No | Yes |
| Modern Usage | Common | Less common |

---

# 44. Kernel Build Artifacts

| File | Purpose |
|------|---------|
| zImage | ARM compressed kernel |
| uImage | U-Boot wrapped image |
| Image | ARM64 raw kernel |
| bzImage | x86 compressed kernel |

---

# 45. Summary

- zImage is a compressed Linux kernel image
- uImage is zImage with U-Boot header
- bootz boots zImage
- bootm boots uImage
- Modern systems often use zImage or FIT images
- uImage mainly used for older U-Boot compatibility

---

````
