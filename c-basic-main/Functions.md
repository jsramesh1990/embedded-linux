# Functions 

---

#  Introduction

In C and C++, **functions** are blocks of reusable code that perform a specific task.

In embedded systems, functions are heavily used for:

* modular firmware design
* hardware abstraction (GPIO, ADC, UART)
* driver development
* reducing code complexity
* real-time task separation
* improving readability and maintainability

---

#  Definition

A function is a self-contained block of code that:

* performs a specific task
* can take inputs (parameters)
* can return output (return value)
* can be called multiple times

---

#  Function Syntax

## Basic Syntax

```c id="fn1"
return_type function_name(parameters) {
    // function body
    return value;
}
```

---

## Example

```c id="fn2"
int add(int a, int b) {
    return a + b;
}
```

---

#  Types of Functions

---

# 1. Library Functions

## Definition

Predefined functions provided by C standard library.

## Example

```c id="lib1"
#include <stdio.h>

printf("Hello");
scanf("%d", &a);
```

---

# 2. User Defined Functions

## Definition

Functions created by the programmer.

## Example

```c id="ud1"
int multiply(int a, int b) {
    return a * b;
}
```

---

#  Embedded System Function Types

---

# 1. No Argument, No Return

## Syntax

```c id="e1"
void functionName(void);
```

## Example

```c id="e2"
#include <stdio.h>

void ledON(void) {
    printf("LED ON\n");
}

int main() {
    ledON();
    return 0;
}
```

---

# 2. Argument, No Return

## Example

```c id="e3"
void display(int value) {
    printf("Value = %d\n", value);
}
```

---

# 3. No Argument, Return Value

## Example

```c id="e4"
int getStatus(void) {
    return 1;
}
```

---

# 4. Argument, Return Value (MOST USED)

## Example

```c id="e5"
int add(int a, int b) {
    return a + b;
}
```

---

# ⚙ Function Call Flow

```text id="flow1"
Main Function
     |
     v
Function Call
     |
     v
Execution Stack Created
     |
     v
Function Executes
     |
     v
Return Value
     |
     v
Back to Main
```

---

#  Embedded Function Execution Flow

```text id="flow2"
while(1)
   |
   v
Sensor Read Function
   |
   v
Processing Function
   |
   v
Control Function (GPIO/Motor)
   |
   v
Repeat
```

---

# 🧠 Function Components

---

## 1. Function Declaration (Prototype)

```c id="f1"
int add(int, int);
```

---

## 2. Function Definition

```c id="f2"
int add(int a, int b) {
    return a + b;
}
```

---

## 3. Function Call

```c id="f3"
int result = add(10, 20);
```

---

# 🔌 Real-Time Embedded Examples

---

# 1. GPIO Control Function

```c id="ex1"
void ledON(void) {
    PORT |= (1 << 2);
}

void ledOFF(void) {
    PORT &= ~(1 << 2);
}
```

---

# 2. Sensor Reading Function

```c id="ex2"
uint16_t readADC(void) {
    return ADC_VALUE;
}
```

---

# 3. Motor Control Function

```c id="ex3"
void setSpeed(uint8_t speed) {
    PWM_REGISTER = speed;
}
```

---

# 4. UART Communication Function

```c id="ex4"
void sendUART(char data) {
    UART_DR = data;
}
```

---

#  Embedded System Main Loop Example

```c id="main1"
int main() {

    initSystem();

    while(1) {
        uint16_t adc = readADC();
        processData(adc);
        controlMotor(adc);
    }

    return 0;
}
```

---

#  Function Stack Diagram

```text id="stack1"
|----------------------|
| return address       |
| local variables      |
| parameters           |
|----------------------|
| Function Stack Frame |
|----------------------|
```

---

# 🧠 Inline Functions (C++ / Embedded Optimization)

## Definition

Used to reduce function call overhead.

## Syntax

```c id="inl1"
inline int square(int x) {
    return x * x;
}
```

---

# ⚡ Recursive Function (Caution in Embedded)

## Example

```c id="rec1"
int factorial(int n) {
    if (n == 0)
        return 1;
    return n * factorial(n - 1);
}
```

---

## Embedded Note:

Avoid recursion in embedded systems due to:

* stack overflow risk
* memory constraints

---

#  Advantages of Functions

| Advantage            | Explanation                    |
| -------------------- | ------------------------------ |
| Code reuse           | Same logic used multiple times |
| Modularity           | Easy debugging                 |
| Maintainability      | Easier updates                 |
| Hardware abstraction | Driver separation              |
| Readability          | Clean structure                |

---

#  Disadvantages

| Issue         | Explanation                     |
| ------------- | ------------------------------- |
| Stack usage   | Memory overhead                 |
| Call overhead | Slight performance delay        |
| Complexity    | Too many functions confuse flow |
| Debugging     | Stack tracing required          |

---

