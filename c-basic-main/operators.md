# Operators 

---

#  Introduction

In C and C++, operators are symbols used to perform operations on variables and values.

In embedded systems, operators are heavily used for:

* Bit-level hardware control
* Sensor data processing
* Decision making
* Register configuration
* Interrupt and flag handling

---

#  Definition

An operator is a symbol that tells the compiler to perform a specific operation on operands.

---

#  Syntax

```c
result = operand1 operator operand2;
```

---

#  Classification of Operators

---

# 1. Arithmetic Operators

## Definition

Used for mathematical operations.

## Operators

```c
+   -   *   /   %
```

## Example Code

```c id="ar1"
#include <stdio.h>

int main() {
    int a = 10, b = 5;

    int add = a + b;
    int sub = a - b;
    int mul = a * b;
    int div = a / b;
    int mod = a % b;

    printf("Add = %d\n", add);
    printf("Sub = %d\n", sub);
    printf("Mul = %d\n", mul);
    printf("Div = %d\n", div);
    printf("Mod = %d\n", mod);

    return 0;
}
```

## Embedded Use

* Sensor calculations
* Scaling ADC values

---

# 2. Relational Operators

## Definition

Used to compare values.

## Operators

```c
==  !=  >  <  >=  <=
```

## Example Code

```c id="re1"
#include <stdio.h>

int main() {
    int temp = 60;

    if (temp > 50) {
        printf("Overheat detected\n");
    } else {
        printf("Normal condition\n");
    }

    return 0;
}
```

---

# 3. Logical Operators

## Definition

Used to combine multiple conditions.

## Operators

```c
&&   ||   !
```

## Example Code

```c id="lo1"
#include <stdio.h>

int main() {
    int temp = 60;
    int pressure = 120;

    if (temp > 50 && pressure > 100) {
        printf("Critical system condition\n");
    }

    return 0;
}
```

---

# 4. Bitwise Operators (VERY IMPORTANT)

## Definition

Used for bit-level operations in hardware.

## Operators

```c
&   |   ^   ~   <<   >>
```

---

## Example Code

```c id="bi1"
#include <stdio.h>

int main() {
    unsigned char a = 0b00001111;
    unsigned char b = 0b00000101;

    printf("AND = %d\n", a & b);
    printf("OR  = %d\n", a | b);
    printf("XOR = %d\n", a ^ b);
    printf("NOT = %d\n", ~a);

    return 0;
}
```

---

## Embedded GPIO Example

```c id="bi2"
#define PORT (*(volatile unsigned char*)0x4000)

int main() {
    // Set bit 2 (LED ON)
    PORT |= (1 << 2);

    // Clear bit 2 (LED OFF)
    PORT &= ~(1 << 2);

    // Toggle bit 2
    PORT ^= (1 << 2);

    return 0;
}
```

---

# 5. Assignment Operators

## Operators

```c
=  +=  -=  *=  /=  %=
```

## Example Code

```c id="as1"
#include <stdio.h>

int main() {
    int a = 10;

    a += 5;
    printf("a = %d\n", a);

    a -= 2;
    printf("a = %d\n", a);

    a *= 2;
    printf("a = %d\n", a);

    return 0;
}
```

---

# 6. Increment / Decrement Operators

## Operators

```c
++   --
```

## Example Code

```c id="in1"
#include <stdio.h>

int main() {
    int i = 0;

    i++;
    printf("i = %d\n", i);

    ++i;
    printf("i = %d\n", i);

    i--;
    printf("i = %d\n", i);

    return 0;
}
```

---

# 7. Conditional (Ternary) Operator

## Syntax

```c
condition ? expression1 : expression2;
```

## Example Code

```c id="te1"
#include <stdio.h>

int main() {
    int temp = 45;

    int status = (temp > 50) ? 1 : 0;

    printf("Status = %d\n", status);

    return 0;
}
```

---

# 8. Special Operators

---

## sizeof Operator

```c id="sp1"
#include <stdio.h>

int main() {
    printf("Size of int = %lu\n", sizeof(int));
    return 0;
}
```

---

## Pointer Operators

```c id="sp2"
#include <stdio.h>

int main() {
    int a = 10;
    int *p = &a;

    printf("Value = %d\n", *p);
    return 0;
}
```

---

#  Working Flow of Operators in Embedded Systems

```text id="wf1"
Input Data (Sensor/Registers)
        |
        v
Variables Store Data
        |
        v
Operators Process Data
        |
        v
Decision Making (if/else)
        |
        v
Hardware Action (LED/Motor/UART)
```

---

#  Bitwise Operation Flow (Embedded Core Concept)

```text id="wf2"
Register Value
      |
      v
Bitwise Operator (& | ^ << >>)
      |
      v
Modified Register Value
      |
      v
Peripheral Control (GPIO / ADC / UART)
```

---

#  Real-Time Embedded Examples

---

## LED ON/OFF

```c id="ex1"
PORT |= (1 << 3);   // LED ON
PORT &= ~(1 << 3);  // LED OFF
```

---

## Sensor Threshold

```c id="ex2"
if (adcValue > 500) {
    alarm = 1;
}
```

---

## UART Communication

```c id="ex3"
if ((uartFlag == 1) && (error == 0)) {
    processData();
}
```

---

## Timer Increment

```c id="ex4"
systemTick++;
```

---

#  Advantages

| Advantage          | Explanation           |
| ------------------ | --------------------- |
| Fast execution     | Direct CPU operations |
| Hardware control   | Bit-level access      |
| Efficient code     | Less instructions     |
| Real-time response | Quick decisions       |
| Memory efficient   | Compact operations    |

---

#  Disadvantages

| Issue               | Explanation                 |
| ------------------- | --------------------------- |
| Debug complexity    | Bitwise logic hard to trace |
| Overflow risk       | Arithmetic errors           |
| Operator precedence | Confusion in expressions    |
| Hardware dependency | Register-specific logic     |

---

#  Operator Precedence

| Priority   | Operators  |   |   |
| ---------- | ---------- | - | - |
| High       | (), ++, -- |   |   |
| Medium     | *, /, %    |   |   |
| Low        | +, -       |   |   |
| Logical    | &&,        |   |   |
| Assignment | =, +=      |   |   |

---



