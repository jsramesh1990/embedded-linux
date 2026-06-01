# atomic-operations

---

#  Introduction

In embedded systems, **atomic operations** are operations that execute **completely in a single step without interruption**.

This means:

* no task switch in the middle
* no interrupt interference
* no partial updates

They are critical for preventing:

* race conditions
* inconsistent system states
* data corruption

---

#  Definition

An atomic operation is a **non-interruptible operation that completes entirely or not at all, ensuring data integrity in concurrent environments**.

---

#  Why Atomic Operations Are Needed

* shared variables in RTOS
* ISR and task interaction
* multi-core access (in advanced MCUs)
* counters, flags, and status registers
* communication buffers

---

#  Non-Atomic vs Atomic Flow

---

##  Non-Atomic Operation

```text id="flow1"
Read value
    |
Add 1
    |
Interrupt occurs 
    |
Another task modifies value
    |
Write back old value → data loss
```

---

##  Atomic Operation

```text id="flow2"
Single uninterruptible step
    |
Value updated safely
    |
No interference allowed
```

---

#  Example: Counter Problem

##  Non-atomic version

```c id="ex1"
int counter = 0;

void task() {
    counter++;
}
```

### Internally:

```text id="internal1"
READ counter
ADD 1
WRITE counter
```

---

## Problem:

If interrupt happens between steps → lost update.

---

#  Atomic Version (Using GCC Built-in)

```c id="ex2"
int counter = 0;

void task() {
    __atomic_add_fetch(&counter, 1, __ATOMIC_SEQ_CST);
}
```

---

#  Atomic vs Non-Atomic Diagram

```text id="diag1"
NON-ATOMIC:
Task A: Read 5
ISR:     modifies value
Task A: Write 6 ❌

ATOMIC:
Task A: [READ+MODIFY+WRITE] in one step
ISR: blocked until complete
```

---

#  Embedded System Example

## UART Flag Handling

```c id="ex3"
volatile int flag = 0;
```

### Without atomic:

```text id="flow3"
Task reads flag
ISR modifies flag
Task writes stale value
```

---

### With atomic:

```c id="ex4"
__atomic_store_n(&flag, 1, __ATOMIC_SEQ_CST);
```

---

#  Types of Atomic Operations

---

## 1. Atomic Read

```c id="t1"
value = __atomic_load_n(&var, __ATOMIC_SEQ_CST);
```

---

## 2. Atomic Write

```c id="t2"
__atomic_store_n(&var, 10, __ATOMIC_SEQ_CST);
```

---

## 3. Atomic Add/Subtract

```c id="t3"
__atomic_add_fetch(&counter, 1, __ATOMIC_SEQ_CST);
```

---

## 4. Compare and Swap (CAS)

```c id="t4"
__atomic_compare_exchange_n(...)
```

Used in lock-free systems.

---

#  Why Atomic Operations Matter in Embedded

* prevent race conditions
* ensure data consistency
* avoid disabling interrupts frequently
* improve system performance
* safer ISR-task communication

---

#  Atomic Operation Flow

```text id="flow4"
Task requests update
        |
CPU executes single atomic instruction
        |
No interrupt allowed in between
        |
Memory updated safely
```

---

#  Real Embedded Example

## Sensor Counter Update

```c id="ex5"
volatile int sensorCount = 0;

void ISR() {
    __atomic_add_fetch(&sensorCount, 1, __ATOMIC_SEQ_CST);
}
```

---

#  Limitations of Atomic Operations

* not all operations are atomic
* complex structures cannot be atomic directly
* may require CPU support
* slower than simple operations
* limited to specific instructions

---

#  Atomic vs Mutex

| Feature    | Atomic          | Mutex              |
| ---------- | --------------- | ------------------ |
| Speed      | Faster          | Slower             |
| Scope      | Single variable | Multiple variables |
| Blocking   | No              | Yes                |
| Complexity | Low             | High               |
| Use case   | counters, flags | critical sections  |

---

#  When to Use Atomic Operations

* increment/decrement counters
* status flags
* ISR-safe variables
* lock-free queues (advanced)
* simple shared variables

---

#  Common Mistakes

---

## 1. Thinking volatile is atomic

```c id="m1"
volatile int x; //  NOT atomic
```

---

## 2. Using non-atomic increment

```c id="m2"
counter++; //  unsafe in concurrency
```

---

## 3. Using atomic for large structures

```text id="m3"
not supported efficiently
```

---

#  Atomic vs Critical Section

| Feature           | Atomic           | Critical Section |
| ----------------- | ---------------- | ---------------- |
| Scope             | Single operation | Multiple lines   |
| Performance       | High             | Medium           |
| Interrupt control | No need          | Required         |
| Use case          | counters         | shared resources |

---


