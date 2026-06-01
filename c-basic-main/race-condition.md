# race-condition

---

#  Introduction

In embedded systems and concurrent programming, a **race condition** occurs when two or more tasks/threads/interrupts access and modify shared data at the same time, and the final result depends on the timing of execution.

This leads to **unpredictable and incorrect behavior**.

---

#  Definition

A race condition is a situation where the system’s output depends on the **non-deterministic timing** of multiple executions accessing shared resources.

---

#  Why It Happens

* shared global variables
* interrupts modifying data
* multitasking (RTOS)
* no synchronization (mutex/lock missing)
* concurrent read/write access

---

#  Basic Race Condition Flow

```text id="flow1"
Task A reads variable
        |
Task B interrupts and modifies it
        |
Task A writes old value back
        |
Result → corrupted/incorrect data
```

---

#  Simple Example (Embedded C)

```c id="ex1"
int counter = 0;

void task1() {
    counter = counter + 1;
}

void task2() {
    counter = counter + 1;
}
```

---

#  Problem Explanation

The operation `counter = counter + 1` is NOT atomic.

It actually does:

```text id="flow2"
Read counter
Add 1
Write back
```

If both tasks execute at the same time → update is lost.

---

#  Race Condition Diagram

```text id="diag1"
Initial counter = 5

Task1 reads 5
Task2 reads 5

Task1 writes 6
Task2 writes 6    (lost update)

Final result = 6 instead of 7
```

---

#  Embedded System Example (Interrupt Issue)

```c id="ex2"
volatile int flag = 0;

void ISR() {
    flag = 1;
}

int main() {
    while(flag == 0) {
        // wait
    }
}
```

---

#  Race Condition Risk

If compiler optimizes incorrectly or caching happens:

* main loop may not see ISR update immediately
* incorrect system behavior

---

#  RTOS Race Condition Example

```c id="ex3"
int shared_data = 0;

void taskA() {
    shared_data++;
}

void taskB() {
    shared_data++;
}
```

Without mutex → race condition occurs.

---

#  RTOS Race Flow

```text id="flow3"
Task A → read shared_data
Task B → read shared_data
Task A → write updated value
Task B → overwrite value 
```

---

#  Real Embedded Impact

* incorrect sensor readings
* corrupted communication data
* missed interrupts
* system instability
* unpredictable firmware behavior

---

#  Example: UART Buffer Race

```c id="ex4"
char buffer[100];

void ISR_UART() {
    buffer[0] = 'A';
}

void process() {
    buffer[0] = 'B';
}
```

Result depends on timing → race condition.

---

#  Why Embedded Systems Are More Vulnerable

* no OS protection (bare metal)
* interrupts can occur anytime
* shared memory access is common
* real-time constraints reduce safety checks

---

#  How to Prevent Race Conditions

---

## 1. Disable interrupts (critical section)

```c id="sol1"
__disable_irq();
counter++;
__enable_irq();
```

---

## 2. Use mutex (RTOS)

```c id="sol2"
xSemaphoreTake(mutex);
counter++;
xSemaphoreGive(mutex);
```

---

## 3. Use atomic operations

```c id="sol3"
__atomic_add_fetch(&counter, 1, __ATOMIC_SEQ_CST);
```

---

## 4. Avoid shared variables

---

## 5. Use volatile correctly (not a fix alone)

```c id="sol4"
volatile int flag;
```

---

#  Race Condition Prevention Flow

```text id="flow4"
Shared resource accessed
        |
        v
Lock / disable interrupt
        |
        v
Critical section execution
        |
        v
Unlock / enable interrupt
        |
        v
Safe execution
```

---

# ⚠ Common Mistakes

---

## 1. Assuming volatile fixes race condition

```text id="m1"
 volatile does NOT provide synchronization
```

---

## 2. Ignoring atomicity

```text id="m2"
counter++ is NOT atomic
```

---

## 3. No critical section protection

```text id="m3"
shared resource accessed freely
```

---

#  Race Condition vs Deadlock

| Feature      | Race Condition     | Deadlock          |
| ------------ | ------------------ | ----------------- |
| Problem type | Timing issue       | Resource blocking |
| Result       | Wrong output       | System freeze     |
| Cause        | No synchronization | Circular wait     |
| Severity     | Data corruption    | System halt       |

---


