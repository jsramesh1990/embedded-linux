# volatile

---

#  Introduction

In C and C++, the **`volatile` keyword** is used to tell the compiler that a variable’s value can change unexpectedly at any time, without any action from the program.

In embedded systems, `volatile` is one of the most critical keywords because it is used for:

* hardware registers
* interrupts (ISR)
* real-time flags
* shared memory between main code and ISR
* communication peripherals

---

#  Definition

A `volatile` variable is a variable whose value **must always be read from memory**, because it can change externally (outside program flow).

---

# 🧾 Syntax

## Basic Syntax

```c id="v1"
volatile data_type variable_name;
```

## Example

```c id="v2"
volatile int flag;
```

---

#  Working Flow of volatile

```text id="flow1"
Hardware / ISR / External Event
             |
             v
     Memory variable updated
             |
             v
CPU must read from memory every time
             |
             v
No compiler optimization allowed
             |
             v
Accurate real-time value used
```

---

#  Why volatile is Needed

Without `volatile`, compiler may:

* cache variable in register
* skip repeated memory reads
* optimize away checks
* assume value does not change

This causes **wrong behavior in embedded systems**.

---

#  Embedded System Flow (volatile)

```text id="flow2"
Sensor / Interrupt / Peripheral
             |
             v
        Memory location updated
             |
             v
     volatile variable reflects change
             |
             v
     CPU reads fresh value every time
             |
             v
     Correct system response
```

---

#  Basic Example

```c id="ex1"
#include <stdio.h>

volatile int flag = 0;

int main() {

    while(flag == 0) {
        // waiting for hardware interrupt
    }

    printf("Flag set!\n");

    return 0;
}
```

---

# ⚠ Without volatile (Problem Case)

```c id="ex2"
int flag = 0;

while(flag == 0) {
    // compiler may optimize this loop forever
}
```

Issue:

* CPU may never reload `flag`
* loop becomes infinite even after update

---

#  Embedded System Examples

---

## 1. Interrupt Flag

```c id="emb1"
volatile int interruptFlag = 0;
```

Used when:

* ISR sets flag
* main loop checks flag

---

## 2. Hardware Register

```c id="emb2"
#define GPIO (*(volatile int*)0x40020000)

GPIO = 1;
```

Used for:

* GPIO control
* peripheral registers

---

## 3. Sensor Data Updated by ISR

```c id="emb3"
volatile int adcValue;
```

Used in:

* ADC conversion complete interrupt

---

## 4. UART Receive Flag

```c id="emb4"
volatile int uartRxFlag;
```

Used in:

* serial communication systems

---

#  Memory Access Flow

```text id="mem1"
Peripheral / ISR
       |
       v
Memory Register Updated
       |
       v
volatile variable ensures fresh read
       |
       v
CPU processes correct data
```

---

#  Real-Time Embedded Use Case

## Example: Button Press Interrupt

```c id="use1"
volatile int buttonPressed = 0;

void ISR_Button() {
    buttonPressed = 1;
}

int main() {
    while(1) {
        if(buttonPressed == 1) {
            // handle event
            buttonPressed = 0;
        }
    }
}
```

---

#  Working in RTOS / Embedded Loop

```text id="flow3"
Main Loop
   |
   v
Check volatile flag
   |
   v
Interrupt updates flag anytime
   |
   v
Main loop reacts immediately
```

---

#  volatile vs Normal Variable

| Feature               | Normal Variable        | volatile Variable  |
| --------------------- | ---------------------- | ------------------ |
| Compiler optimization | Allowed                | Not allowed        |
| Memory access         | May use cache/register | Always from memory |
| ISR usage             | Unsafe                 | Safe               |
| Hardware registers    | Not suitable           | Required           |

---

#  Advantages

| Advantage                   | Explanation               |
| --------------------------- | ------------------------- |
| Real-time correctness       | Always reads latest value |
| Hardware compatibility      | Required for registers    |
| ISR safety                  | Works with interrupts     |
| Prevent optimization issues | Ensures correct execution |

---

#  Disadvantages

| Issue            | Explanation                            |
| ---------------- | -------------------------------------- |
| Slower execution | No optimization allowed                |
| No atomicity     | Does not solve race conditions         |
| Misuse risk      | Overusing volatile reduces performance |

---

#  Common Mistakes

---

## 1. Missing volatile in ISR variable

```c id="err1"
int flag; //  wrong for ISR
```

---

## 2. Overusing volatile everywhere

```c id="err2"
volatile int x = 10; // unnecessary
```

---

## 3. Thinking volatile provides thread safety

Volatile does NOT:

* prevent race conditions
* ensure atomic operations

---


