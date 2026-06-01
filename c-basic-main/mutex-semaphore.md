# mutex-semaphore

---

#  Introduction

In embedded systems and RTOS, **mutex** and **semaphore** are synchronization mechanisms used to control access to shared resources.

They prevent:

* race conditions
* data corruption
* inconsistent system states

But they are used in different ways and serve different purposes.

---

#  Definitions

## Mutex

A **mutex (mutual exclusion)** is a locking mechanism used to ensure that **only one task accesses a shared resource at a time**.

---

## Semaphore

A **semaphore** is a signaling mechanism used to **control access to one or more resources or synchronize tasks**.

---

#  Core Difference Idea

```text id="flow0"
Mutex → Ownership (1 owner only)
Semaphore → Counting / signaling mechanism
```

---

#  Working Flow

## Mutex Flow

```text id="flow1"
Task A locks mutex
        |
Task B tries to lock → WAIT
        |
Task A unlocks mutex
        |
Task B proceeds
```

---

## Semaphore Flow

```text id="flow2"
Semaphore count = 2

Task A → uses resource (count--)
Task B → uses resource (count--)
Task C → waits (count = 0)
```

---

#  Mutex vs Semaphore Diagram

```text id="diag1"
MUTEX:
Task A → [LOCK] → Resource → [UNLOCK]
Task B → WAITING

SEMAPHORE:
Task A → uses resource
Task B → uses resource
Task C → waits (no slots)
```

---

#  Mutex Example (RTOS)

```c id="ex1"
xSemaphoreHandle mutex;

void task1() {
    xSemaphoreTake(mutex, portMAX_DELAY);

    // critical section
    counter++;

    xSemaphoreGive(mutex);
}
```

---

#  Semaphore Example

```c id="ex2"
xSemaphoreHandle sem;

void taskA() {
    xSemaphoreTake(sem, portMAX_DELAY);
}

void taskB() {
    xSemaphoreGive(sem);
}
```

---

#  Types of Semaphores

---

## 1. Binary Semaphore

* value: 0 or 1
* used for signaling

```text id="sem1"
0 → locked
1 → available
```

---

## 2. Counting Semaphore

* value: 0 to N
* used for multiple resources

```text id="sem2"
Example: 3 printers → count = 3
```

---

#  Real Embedded Example

## UART Driver Protection (Mutex)

```c id="ex3"
void UART_Send(char *data) {
    xSemaphoreTake(uartMutex, portMAX_DELAY);

    // send data safely
    HAL_UART_Transmit(data);

    xSemaphoreGive(uartMutex);
}
```

---

## Task Synchronization (Semaphore)

```c id="ex4"
void ISR() {
    xSemaphoreGive(sem);
}

void Task() {
    xSemaphoreTake(sem, portMAX_DELAY);
    // process data
}
```

---

#  Working Flow in Embedded Systems

```text id="flow3"
ISR triggers event
        |
        v
Semaphore is given
        |
        v
Task wakes up
        |
        v
Task processes data
```

---

#  Key Differences Table

| Feature              | Mutex                    | Semaphore                    |
| -------------------- | ------------------------ | ---------------------------- |
| Purpose              | Mutual exclusion         | Signaling / resource control |
| Ownership            | Yes (task owns it)       | No ownership                 |
| Value                | Binary (locked/unlocked) | Counting                     |
| Use case             | Protect shared resource  | Task sync / resource pool    |
| Priority inheritance | Supported                | Not supported                |

---

#  Key Concepts

---

## 1. Mutex = Lock

* only one task allowed
* ensures exclusive access

---

## 2. Semaphore = Signal

* not owned by task
* used for event notification

---

#  Common Mistakes

---

## 1. Using semaphore instead of mutex

```text id="m1"
leads to race conditions in critical section
```

---

## 2. Forgetting to release mutex

```c id="m2"
deadlock risk
```

---

## 3. Wrong synchronization design

```text id="m3"
causes unpredictable system behavior
```

---

## 4. Using mutex in ISR

```text id="m4"
 not allowed in most RTOS
```

---

#  Real Embedded Impact

Without proper synchronization:

* UART data corruption
* sensor value mismatch
* communication failure
* system instability
* race conditions

---

#  Mutex vs Semaphore Flow

## Mutex Flow

```text id="flow4"
Task A locks → uses resource → unlocks
Task B waits → then proceeds
```

## Semaphore Flow

```text id="flow5"
ISR gives signal → Task wakes → processes data
```

---

#  Best Practices

---

## 1. Use mutex for shared resources

```text id="best1"
UART, SPI, shared variables
```

---

## 2. Use semaphore for signaling

```text id="best2"
ISR → Task communication
```

---

## 3. Keep critical section small

---

## 4. Avoid nested locks

---

# 🏁 Final Summary

* **Mutex** → protects shared resources (exclusive access)
* **Semaphore** → signals events or manages multiple resources

In embedded systems:

* mutex prevents data corruption
* semaphore enables task synchronization

Both are essential for safe RTOS design.


