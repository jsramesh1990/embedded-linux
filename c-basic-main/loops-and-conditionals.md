# Loops and Conditionals 

---

#  Introduction

In C and C++ embedded systems, **loops and conditionals** are used to control program flow based on conditions and to repeat tasks.

They are essential for:

* reading sensors continuously
* controlling hardware (LED, motor, relay)
* real-time decision making
* handling communication protocols
* running infinite embedded firmware loops

---

#  Definition

## Conditional Statements

Conditional statements are used to execute code only when a condition is true or false.

## Loops

Loops are used to execute a block of code repeatedly until a condition is met.

---

#  Basic Syntax Overview

```c id="syn1"
if (condition) {
    // code
}

for (initialization; condition; update) {
    // code
}

while (condition) {
    // code
}

do {
    // code
} while (condition);
```

---

# 🧠 Conditional Statements

---

# 1. if Statement

## Definition

Executes code only if condition is true.

## Syntax

```c id="if1"
if (condition) {
    // statements
}
```

## Example

```c id="if2"
#include <stdio.h>

int main() {
    int temp = 60;

    if (temp > 50) {
        printf("Overheat detected\n");
    }

    return 0;
}
```

---

# 2. if-else Statement

## Definition

Executes one block if true, another if false.

## Syntax

```c id="if3"
if (condition) {
    // true block
} else {
    // false block
}
```

## Example

```c id="if4"
#include <stdio.h>

int main() {
    int sensor = 0;

    if (sensor == 1) {
        printf("ON\n");
    } else {
        printf("OFF\n");
    }

    return 0;
}
```

---

# 3. if-else if Ladder

## Definition

Used for multiple conditions.

## Syntax

```c id="if5"
if (cond1) {
}
else if (cond2) {
}
else {
}
```

## Example

```c id="if6"
int temp = 70;

if (temp > 80) {
    printf("Critical\n");
}
else if (temp > 60) {
    printf("Warning\n");
}
else {
    printf("Normal\n");
}
```

---

# 4. switch Statement

## Definition

Used for multiple fixed values.

## Syntax

```c id="sw1"
switch(expression) {
    case value1:
        break;
    case value2:
        break;
    default:
}
```

## Example

```c id="sw2"
#include <stdio.h>

int main() {
    int mode = 2;

    switch(mode) {
        case 1:
            printf("Manual Mode\n");
            break;

        case 2:
            printf("Auto Mode\n");
            break;

        default:
            printf("Invalid Mode\n");
    }

    return 0;
}
```

---

#  Embedded Use of Conditionals

* Sensor threshold checking
* Fault detection
* Mode selection (manual/auto)
* Interrupt flag handling
* Safety logic

---

#  Looping Statements

---

# 1. for Loop

## Definition

Used when number of iterations is known.

## Syntax

```c id="for1"
for (init; condition; update) {
    // code
}
```

## Example

```c id="for2"
#include <stdio.h>

int main() {
    for (int i = 0; i < 5; i++) {
        printf("Count = %d\n", i);
    }

    return 0;
}
```

---

## Embedded Use

* Buffer processing
* Array traversal
* PWM generation steps

---

# 2. while Loop

## Definition

Runs while condition is true.

## Syntax

```c id="wh1"
while (condition) {
    // code
}
```

## Example

```c id="wh2"
#include <stdio.h>

int main() {
    int i = 0;

    while (i < 5) {
        printf("i = %d\n", i);
        i++;
    }

    return 0;
}
```

---

## Embedded Use

* Sensor polling
* Waiting for flag
* Communication checks

---

# 3. do-while Loop

## Definition

Executes at least once.

## Syntax

```c id="dw1"
do {
    // code
} while (condition);
```

## Example

```c id="dw2"
#include <stdio.h>

int main() {
    int i = 0;

    do {
        printf("i = %d\n", i);
        i++;
    } while (i < 5);

    return 0;
}
```

---

## Embedded Use

* Initialization routines
* Retry mechanisms
* Hardware startup checks

---

#  Infinite Loop (MOST IMPORTANT IN EMBEDDED)

## Definition

Embedded systems run continuously.

## Syntax

```c id="inf1"
while(1) {
    // firmware loop
}
```

---

## Example (Embedded Firmware Loop)

```c id="inf2"
int main() {

    while(1) {

        readSensor();
        processData();
        controlMotor();
    }

    return 0;
}
```

---

# ⚙ Working Flow of Conditionals & Loops

---

# Conditional Flow

```text id="cf1"
Input Data
   |
   v
Condition Check (if/switch)
   |
   v
True / False Decision
   |
   v
Execute Block
```

---

# Loop Flow

```text id="lf1"
Start
  |
  v
Initialization
  |
  v
Condition Check
  |
  v
Execute Code
  |
  v
Update
  |
  v
Repeat or Exit
```

---

# Embedded System Full Execution Flow

```text id="ef1"
Power ON
   |
   v
Initialization Code
   |
   v
while(1) Loop
   |
   v
Sensor Read
   |
   v
Condition Check
   |
   v
Actuator Control
   |
   v
Repeat Forever
```

---

#  Real-Time Embedded Examples

---

## 1. LED Blinking

```c id="ex1"
while(1) {
    PORT ^= (1 << 2);  // toggle LED
}
```

---

## 2. Temperature Monitoring

```c id="ex2"
if (temp > 50) {
    alarm = 1;
} else {
    alarm = 0;
}
```

---

## 3. Sensor Polling

```c id="ex3"
while (1) {
    adcValue = readADC();
}
```

---

## 4. Mode Selection

```c id="ex4"
switch(mode) {
    case 0:
        idleMode();
        break;

    case 1:
        autoMode();
        break;
}
```

---

#  Advantages

| Feature       | Benefit                       |
| ------------- | ----------------------------- |
| Conditionals  | Smart decision making         |
| Loops         | Repetition automation         |
| Infinite loop | Continuous system operation   |
| switch case   | Faster multi-branch selection |
| while loop    | Real-time monitoring          |

---

#  Disadvantages

| Issue              | Explanation         |
| ------------------ | ------------------- |
| Infinite loop risk | System hang         |
| Complex nesting    | Hard debugging      |
| CPU load           | Busy polling        |
| Wrong condition    | Unexpected behavior |

---

