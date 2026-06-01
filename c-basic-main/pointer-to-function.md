# pointer-to-function

---

#  Introduction

In C and C++, a **function pointer** is a pointer that stores the address of a function.

In embedded systems, function pointers are heavily used in:

* interrupt service routines (ISR dispatching)
* driver abstraction layers
* callback mechanisms
* state machines
* RTOS task scheduling
* modular firmware design

---

#  Definition

A function pointer is a pointer that points to a function instead of a variable.

---

#  Syntax

## Function Pointer Declaration

```c id="f1"
return_type (*pointer_name)(parameter_types);
```

---

## Example

```c id="f2"
int (*fp)(int, int);
```

---

#  Basic Working Flow

```text id="flow1"
Function defined in memory
        |
        v
Function pointer stores its address
        |
        v
Pointer is called like a function
        |
        v
Actual function executes
```

---

#  Simple Example

```c id="ex1"
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int main() {
    int (*fp)(int, int) = add;

    printf("%d\n", fp(5, 3));

    return 0;
}
```

---

#  Embedded System Use Case

---

# 1. Callback Function Example

```c id="ex2"
void sensorHandler() {
    printf("Sensor triggered\n");
}

void registerCallback(void (*cb)()) {
    cb();
}

int main() {
    registerCallback(sensorHandler);
}
```

---

#  Flow

```text id="flow2"
Event occurs
      |
      v
Function pointer called
      |
      v
Callback function executes
```

---

# 2. Driver Abstraction Example

```c id="ex3"
int uart_send(char data) {
    return data;
}

int spi_send(char data) {
    return data;
}

int (*send_func)(char);

int main() {
    send_func = uart_send;
    send_func('A');

    send_func = spi_send;
    send_func('B');
}
```

---

#  Flow

```text id="flow3"
Function pointer changes dynamically
        |
        v
Same interface → different hardware behavior
```

---

# 3. ISR Table (Very Important in Embedded)

```c id="ex4"
void ISR_Timer() {
    // timer interrupt
}

void ISR_UART() {
    // uart interrupt
}

void (*ISR_VectorTable[2])();

int main() {
    ISR_VectorTable[0] = ISR_Timer;
    ISR_VectorTable[1] = ISR_UART;

    ISR_VectorTable[0](); // call timer ISR
}
```

---

#  Interrupt Vector Flow

```text id="flow4"
Interrupt occurs
      |
      v
Vector table lookup
      |
      v
Function pointer retrieved
      |
      v
ISR executed
```

---

#  Why Function Pointers are Used

* runtime decision making
* hardware abstraction
* memory efficiency
* modular firmware design
* callback-based architecture

---

#  Embedded Real-Time Example

## Motor Control Strategy Switching

```c id="ex5"
void mode1() {
    printf("Low speed mode\n");
}

void mode2() {
    printf("High speed mode\n");
}

void (*motorMode)();

int main() {
    motorMode = mode1;
    motorMode();

    motorMode = mode2;
    motorMode();
}
```

---

#  Working Flow

```text id="flow5"
System state changes
        |
        v
Function pointer updated
        |
        v
Different behavior executed
```

---

#  Memory Representation

```text id="mem1"
Function Code:
+--------------------+
| add()              |
| sensorHandler()    |
| uart_send()        |
+--------------------+

Function Pointer:
fp → address of function
```

---

#  Common Mistakes

---

## 1. Wrong declaration

```c id="m1"
int *fp(int, int); //  function returning pointer
```

---

## 2. Not matching signature

```c id="m2"
int (*fp)(int) = add; //  mismatch
```

---

## 3. Null function pointer call

```c id="m3"
fp(); //  if fp is NULL → crash
```

---

#  Safe Usage

```c id="safe1"
if (fp != NULL) {
    fp();
}
```

---

#  Advantages

| Advantage        | Explanation                     |
| ---------------- | ------------------------------- |
| Flexibility      | Runtime function selection      |
| Modularity       | Clean architecture              |
| Reusability      | Same interface, different logic |
| Embedded drivers | Hardware abstraction            |
| Efficient design | Less conditional logic          |

---

#  Disadvantages

| Issue             | Explanation             |
| ----------------- | ----------------------- |
| Complex debugging | Hard to trace calls     |
| NULL risk         | Crash if not checked    |
| Readability       | Confusing for beginners |
| Safety concerns   | Pointer misuse          |

---

