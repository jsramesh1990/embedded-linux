# critical-section

---

#  Introduction

In embedded systems and RTOS-based applications, a **critical section** is a part of code where shared resources (variables, memory, peripherals) are accessed.

If multiple tasks or interrupts access this section simultaneously, it can lead to:

* race conditions
* data corruption
* unpredictable behavior

So, critical sections must be protected to ensure **mutual exclusion**.

---

#  Definition

A critical section is a **code segment that accesses shared resources and must not be executed by more than one task or interrupt at a time**.

---

#  Why Critical Section is Needed

* shared variables in multitasking systems
* interrupt + task shared data access
* hardware register access
* communication buffers (UART/SPI/CAN)
* real-time control systems

---

#  Critical Section Flow

```text id="flow1"
Task enters critical section
        |
Check shared resource
        |
Modify data
        |
Exit critical section
        |
Other tasks allowed access
```

---

#  Problem Without Protection

```text id="flow2"
Task A reads counter = 5
Task B reads counter = 5
Task A updates → 6
Task B updates → 6  (lost update)
```

---

#  Critical Section Diagram

```text id="diag1"
Without protection:

Task A ----\
             > Shared Resource → CORRUPTION
Task B ----/
```

---

#  Embedded Example (Problematic)

```c id="ex1"
int counter = 0;

void task1() {
    counter++;
}

void task2() {
    counter++;
}
```

---

#  Problem

`counter++` is NOT atomic:

```text id="flow3"
Read → Modify → Write
```

---

#  How to Protect Critical Section

---

## 1. Disable Interrupts (Bare Metal)

```c id="sol1"
__disable_irq();

counter++;

__enable_irq();
```

---

## 2. Mutex (RTOS)

```c id="sol2"
xSemaphoreTake(mutex, portMAX_DELAY);

counter++;

xSemaphoreGive(mutex);
```

---

## 3. Binary Semaphore

Used for task synchronization.

---

## 4. Atomic Operations

```c id="sol3"
__atomic_add_fetch(&counter, 1, __ATOMIC_SEQ_CST);
```

---

#  Critical Section Flow (Protected)

```text id="flow4"
Task enters
      |
Acquire lock / disable interrupt
      |
Access shared resource
      |
Release lock / enable interrupt
      |
Exit safely
```

---

#  Embedded Example (UART Buffer)

```c id="ex2"
char buffer[100];

void ISR_UART() {
    buffer[0] = 'A';
}

void task() {
    buffer[0] = 'B';
}
```

### Problem:

* ISR and task modify same memory

---

## Fix using critical section

```c id="fix1"
__disable_irq();
buffer[0] = 'B';
__enable_irq();
```

---

#  RTOS Critical Section Flow

```text id="flow5"
Task A → Lock Mutex → Access Resource → Unlock
Task B → Wait → Lock Mutex → Access Resource → Unlock
```

---

#  Real Embedded Impact if Not Used

* corrupted sensor values
* wrong actuator control
* UART data corruption
* system instability
* race conditions

---

#  Characteristics of Critical Section

* accesses shared resource
* must be protected
* short execution time preferred
* affects system performance
* used in multitasking environments

---

#  Common Mistakes

---

## 1. Long critical sections

```text id="m1"
Holding lock too long → performance issues
```

---

## 2. Forgetting unlock

```c id="m2"
mutex taken but not released → deadlock
```

---

## 3. Nested critical sections

```text id="m3"
can lead to deadlock
```

---

## 4. Using volatile instead of synchronization

```text id="m4"
volatile does NOT protect critical section
```

---

#  Critical Section vs Race Condition

| Feature | Critical Section  | Race Condition      |
| ------- | ----------------- | ------------------- |
| Purpose | Protection        | Problem             |
| State   | Controlled access | Uncontrolled access |
| Outcome | Safe execution    | Data corruption     |

---


