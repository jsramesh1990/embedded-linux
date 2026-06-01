# Data Types

---

#  Introduction

In C and C++ embedded systems, data types define the type of data a variable can store, the memory it occupies, and how the processor interprets it.

In embedded systems, data types are critical because:

* RAM and Flash are limited
* Hardware registers require exact size mapping
* Performance and timing must be predictable
* Memory corruption must be avoided

---

#  Definition

A data type is a classification that tells the compiler:

* What kind of value a variable stores
* How much memory to allocate
* What operations can be performed on that variable

---

#  Syntax of Data Types

## General Syntax

```c
data_type variable_name;
```

## With Initialization

```c
data_type variable_name = value;
```

## Example

```c
int a;
float b = 10.5;
char c = 'A';
```

---

#  Classification of Data Types

---

# 1. Fundamental Data Types

## Definition

Fundamental data types are built-in basic types used to store simple values.

## Syntax

```c
int a;
char b;
float c;
double d;
```

## Table

| Type   | Size      | Description    |
| ------ | --------- | -------------- |
| char   | 1 byte    | Character      |
| int    | 2/4 bytes | Integer        |
| float  | 4 bytes   | Decimal value  |
| double | 8 bytes   | High precision |
| void   | 0         | No value       |

---

## Example

```c
int speed = 100;
float temperature = 36.5;
char grade = 'A';
```

---

# 2. Derived Data Types

## Definition

Derived data types are created from fundamental types.

## Syntax

### Array

```c
data_type array_name[size];
```

### Pointer

```c
data_type *pointer_name;
```

## Example

```c
int arr[5];
int *ptr;
```

---

# 3. User Defined Data Types

## Definition

User-defined data types allow grouping multiple values into one unit.

## Structure Syntax

```c
struct structure_name {
    data_type member1;
    data_type member2;
};
```

## Example

```c
struct Sensor {
    int temperature;
    int pressure;
};
```

---

## Enum Syntax

```c
enum enum_name { VALUE1, VALUE2, VALUE3 };
```

## Example

```c
enum State { OFF, ON, IDLE };
```

---

# 4. Fixed Width Data Types (Embedded Standard)

## Definition

Fixed-width types provide guaranteed memory size across platforms.

## Syntax

```c
#include <stdint.h>

uint8_t a;
uint16_t b;
uint32_t c;
```

## Table

| Type     | Size    | Range           |
| -------- | ------- | --------------- |
| uint8_t  | 1 byte  | 0–255           |
| int8_t   | 1 byte  | -128 to 127     |
| uint16_t | 2 bytes | 0–65535         |
| int16_t  | 2 bytes | -32768 to 32767 |
| uint32_t | 4 bytes | 0–4.29B         |

---

#  Real-Time Embedded Examples

---

## 1. ADC Sensor Value

```c
uint16_t adcValue;
```

Used for:

* temperature sensors
* pressure sensors
* analog input reading

---

## 2. Motor Control

```c
uint8_t motorSpeed;
```

Used for:

* PWM control
* robotics
* automotive systems

---

## 3. UART Buffer

```c
uint8_t uartBuffer[128];
```

Used for:

* serial communication
* data transmission
* debugging logs

---

## 4. System Timer

```c
uint32_t systemTick;
```

Used for:

* RTOS scheduling
* delay functions
* time tracking

---

#  Working Flow of Data Types in Embedded System

```text
Source Code
    |
    v
Variable Declaration (Data Type Selection)
    |
    v
Compiler allocates memory (RAM/Flash mapping)
    |
    v
CPU reads/writes variable
    |
    v
Peripheral interaction (GPIO / ADC / UART / SPI)
    |
    v
Runtime execution in embedded system
```

---

#  Memory Allocation Flow

```text
Data Type Selected
        |
        v
Compiler decides size (1B / 2B / 4B / 8B)
        |
        v
Memory region assigned:
    - Stack (local variables)
    - Data/BSS (global/static)
    - Flash (const data)
        |
        v
CPU accesses memory location
```

---

#  Memory Layout Diagram

```text
+----------------------+
| Flash (Program Code) |
+----------------------+
| .data (Init Globals) |
+----------------------+
| .bss (Uninit Vars)   |
+----------------------+
| Heap (Dynamic)       |
+----------------------+
| Stack (Local Vars)   |
+----------------------+
```

---

#  Signed vs Unsigned

## Signed

Stores both positive and negative values.

```c
int8_t temp = -20;
```

## Unsigned

Stores only positive values.

```c
uint8_t led = 1;
```

---

#  Float vs Integer in Embedded

| Feature  | Integer | Float       |
| -------- | ------- | ----------- |
| Speed    | Fast    | Slow        |
| Memory   | Low     | High        |
| Accuracy | Exact   | Approximate |

Embedded systems prefer integer with scaling:

```c
uint16_t temp = 253; // 25.3°C
```

---

#  Structure Example

```c
struct Sensor {
    uint16_t temperature;
    uint16_t pressure;
    uint8_t status;
};
```

---

#  Const Example

```c
const int maxSpeed = 120;
```

Used for:

* read-only values
* flash memory optimization

---

#  Volatile Example

```c
volatile uint8_t flag;
```

Used for:

* interrupts
* hardware registers
* real-time flags

---

#  Advantages of Proper Data Types

| Advantage            | Explanation              |
| -------------------- | ------------------------ |
| Memory efficient     | Optimized RAM usage      |
| Faster execution     | CPU optimized operations |
| Hardware compatible  | Register mapping         |
| Predictable behavior | Fixed size types         |
| Safer code           | Avoid overflow issues    |

---

#  Disadvantages of Improper Use

| Issue                | Result               |
| -------------------- | -------------------- |
| Wrong type selection | Data corruption      |
| Oversized variables  | Memory waste         |
| Float usage          | CPU overhead         |
| Global misuse        | Debugging difficulty |

---



---
