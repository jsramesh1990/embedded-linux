# constant

---

#  Introduction

In C and C++, the **`const` keyword** is used to define variables whose values cannot be modified after initialization.

In embedded systems, `const` is widely used for:

* protecting critical configuration data
* placing data in ROM/Flash memory
* improving code safety
* preventing accidental modification
* optimizing memory usage

---

#  Definition

A `const` variable is a **read-only variable** whose value is fixed after initialization and cannot be changed during program execution.

---

# 🧾 Syntax

## Basic Syntax

```c id="c1"
const data_type variable_name = value;
```

## Example

```c id="c2"
const int maxSpeed = 120;
```

---

#  Working Flow of const

```text id="flow1"
Program Start
     |
     v
const variable initialized
     |
     v
Stored in memory (often Flash/ROM)
     |
     v
Program can ONLY read value
     |
     v
Any write attempt → compiler error
```

---

#  Memory Behavior in Embedded Systems

```text id="mem1"
Flash / ROM
   |
   v
const variables stored here (usually)
   |
   v
Read-only access enforced by compiler
```

---

#  Types of const Usage

---

# 1. const Variable

## Definition

Simple read-only variable.

## Example

```c id="ex1"
const int baudRate = 9600;
```

---

# 2. const Pointer to Data

## Definition

Pointer can change, but data cannot be modified.

## Syntax

```c id="ex2"
const int *ptr;
```

## Example

```c id="ex3"
int a = 10;
const int *ptr = &a;

*ptr = 20;  //  Not allowed
ptr = &b;   // Allowed
```

---

# 3. const Pointer

## Definition

Pointer cannot change, but data can change.

## Syntax

```c id="ex4"
int *const ptr;
```

## Example

```c id="ex5"
int a = 10;
int *const ptr = &a;

*ptr = 20;  // Allowed
ptr = &b;   //  Not allowed
```

---

# 4. const Pointer to const Data

## Definition

Neither pointer nor data can be modified.

## Syntax

```c id="ex6"
const int *const ptr;
```

## Example

```c id="ex7"
int a = 10;
const int *const ptr = &a;
```

---

#  Embedded System Examples

---

## 1. Configuration Constant

```c id="emb1"
const int systemClock = 16000000;
```

Used for:

* timing calculations
* clock configuration

---

## 2. Fixed Buffer Size

```c id="emb2"
const int bufferSize = 128;
char buffer[bufferSize];
```

Used for:

* UART buffers
* SPI buffers

---

## 3. Hardware Limits

```c id="emb3"
const int maxTemperature = 100;
```

Used for:

* safety thresholds
* system protection

---

## 4. Lookup Tables (VERY IMPORTANT)

```c id="emb4"
const int sineTable[5] = {0, 1, 0, -1, 0};
```

Used for:

* DSP applications
* motor control
* signal processing

---

#  Working Flow in Embedded System

```text id="flow2"
const variable defined
        |
        v
Stored in Flash / ROM
        |
        v
CPU reads value during execution
        |
        v
Value used in calculations
        |
        v
No modification allowed
```

---

#  const in Memory Diagram

```text id="diag1"
+----------------------+
| Flash (Program)      |
|----------------------|
| const variables      |
| lookup tables        |
+----------------------+
| RAM                  |
| runtime variables    |
+----------------------+
```

---

#  Advantages

| Advantage     | Explanation                      |
| ------------- | -------------------------------- |
| Memory safety | Prevents accidental modification |
| ROM usage     | Saves RAM space                  |
| Code clarity  | Clearly indicates fixed values   |
| Optimization  | Compiler optimization possible   |
| Reliability   | Reduces bugs in firmware         |

---

#  Disadvantages

| Issue                   | Explanation                      |
| ----------------------- | -------------------------------- |
| No modification         | Value cannot change at runtime   |
| Initialization required | Must assign value at declaration |
| Pointer complexity      | const with pointers is confusing |

---

#  Real-Time Embedded Use Cases

---

## 1. System Clock Definition

```c id="use1"
const int F_CPU = 16000000;
```

---

## 2. Baud Rate Configuration

```c id="use2"
const int baudRate = 115200;
```

---

## 3. Safety Threshold

```c id="use3"
const int maxPressure = 200;
```

---

## 4. Calibration Data

```c id="use4"
const int sensorOffset = 5;
```

---

# ⚠ Common Mistakes

---

## 1. Trying to Modify const

```c id="err1"
const int a = 10;
a = 20; //  Error
```

---

## 2. Not using const for fixed values

```c id="err2"
int bufferSize = 128; // should be const
```

---

## 3. Incorrect pointer usage

```c id="err3"
const int *ptr;
*ptr = 10; //  Not allowed
```

---


