
# Header Guards

## Overview

Header guards prevent **multiple inclusion of the same header file** in a single compilation unit.

Without header guards, the compiler may see the same declarations multiple times, leading to:
- redefinition errors
- compilation failures
- linker conflicts

---

# 1. Why Header Guards are Needed

When multiple `.c` files include the same `.h` file:

```

file1.c → includes gpio.h
file2.c → includes gpio.h
main.c  → includes gpio.h

````

All get expanded into one translation unit → duplication problem.

---

# 2. Problem Without Header Guards

## Example

### gpio.h (BAD)
```c
int gpio_status;
void gpio_init();
``` id="bad1"

---

### file1.c
```c
#include "gpio.h"
````

### file2.c

```c
#include "gpio.h"
```

---

## Result (Compiler Error)

````
redefinition of 'gpio_status'
``` id="err1"

---

# 3. Solution: Header Guards

## Standard Pattern

```c
#ifndef GPIO_H
#define GPIO_H

// declarations
int gpio_status;
void gpio_init(void);

#endif
``` id="good1"

---

# 4. How Header Guards Work

## Step-by-step expansion

### First inclusion:
````

GPIO_H is not defined → include file
define GPIO_H

```

### Second inclusion:
```

GPIO_H already defined → skip file

```

---

# 5. Visual Flow

```

First include → allowed
Second include → ignored
Third include → ignored

````id="flow1"

---

# 6. Best Practice Naming

Use unique macro names:

| File | Guard |
|------|------|
| gpio.h | GPIO_H |
| uart.h | UART_H |
| motor.h | MOTOR_H |

---

## Better naming (recommended)

```c
#ifndef PROJECT_GPIO_H
#define PROJECT_GPIO_H
``` id="best1"

---

# 7. Modern Alternative: #pragma once

## Example

```c
#pragma once

void gpio_init(void);
``` id="pragma1"

---

## Advantages

- Simpler
- No macro naming issues
- Faster compilation (some compilers)

---

## Disadvantages

- Not part of strict C standard (but widely supported)

---

# 8. Header Guards vs #pragma once

| Feature | Header Guards | #pragma once |
|--------|--------------|--------------|
| Portability | 100% | High |
| Speed | Normal | Faster |
| Risk of duplication | None | Very low |
| Standard compliance | Yes | No (but widely accepted) |

---

# 9. Embedded Linux Usage

Header guards are used in:

- Kernel headers
- Device drivers
- HAL layers
- RTOS code
- Bootloader code (U-Boot)

---

## Example (Kernel-style header)

```c
#ifndef _LINUX_GPIO_H
#define _LINUX_GPIO_H

struct gpio_desc {
    int pin;
};

#endif
``` id="kernel1"

---

# 10. Common Mistakes

## 10.1 Missing guards

❌ Leads to:
````

multiple definition errors

````

---

## 10.2 Wrong macro name reuse

```c
#ifndef GPIO_H
#define GPIO_H
````

If reused in another file → conflict risk

---

## 10.3 Partial guards

❌ Wrong:

```c
#ifndef GPIO_H
// missing define
#endif
```

---

# 11. Compilation Flow Impact

````
Preprocessor:
- checks macro
- includes file only once
``` id="flow2"

---

# 12. Header Guards in Large Projects

In large embedded systems:

- thousands of headers
- strict naming conventions
- module-based guard prefixes

Example:
````

KERNEL_
U_BOOT_
PROJECT_

````id="scale1"

---

# 13. Real Embedded Example

## gpio.h

```c
#ifndef BSP_GPIO_H
#define BSP_GPIO_H

void gpio_init(void);
void gpio_set(int pin);

#endif
``` id="emb1"

---

## gpio.c

```c
#include "gpio.h"
````

---

## main.c

```c
#include "gpio.h"
```

---

# 14. Summary

* Header guards prevent multiple inclusion
* Use `#ifndef`, `#define`, `#endif`
* Prevent redefinition errors
* Essential for multi-file embedded projects
* `#pragma once` is modern alternative

---

