# deadlock

---

#  Introduction

In embedded systems and RTOS-based applications, a **deadlock** is a situation where two or more tasks are permanently blocked because each is waiting for a resource held by the other.

When a deadlock occurs, the system stops making progress and can appear “frozen”.

---

#  Definition

A deadlock is a condition where a set of tasks are **blocked forever**, each waiting for a resource that another task in the same set holds.

---

#  Real-World Analogy

Two people trying to cross a narrow bridge from opposite sides:

* Person A waits for Person B to move
* Person B waits for Person A to move
* Neither moves → deadlock

---

#  Deadlock Basic Flow

```text id="flow1"
Task A locks Resource 1
        |
Task B locks Resource 2
        |
Task A waits for Resource 2
        |
Task B waits for Resource 1
        |
Both wait forever 
```

---

#  Deadlock Diagram

```text id="diag1"
Task A                 Task B
------                 ------
Lock R1               Lock R2
   |                     |
   v                     v
Wait R2               Wait R1
   |                     |
   +--------❌----------+
        DEADLOCK
```

---

# 📌 Conditions for Deadlock (Coffman Conditions)

Deadlock occurs only when ALL four conditions are met:

---

## 1. Mutual Exclusion

Only one task can use a resource at a time.

---

## 2. Hold and Wait

A task holds one resource while waiting for another.

---

## 3. No Preemption

Resources cannot be forcibly taken.

---

## 4. Circular Wait

A cycle of tasks waiting for each other.

```text id="cycle1"
T1 → T2 → T3 → T1
```

---

#  Embedded System Example

## Deadlock with Mutex

```c id="ex1"
xSemaphoreTake(mutexA);
xSemaphoreTake(mutexB);

// critical section

xSemaphoreGive(mutexB);
xSemaphoreGive(mutexA);
```

---

## Another Task (Reverse Order)

```c id="ex2"
xSemaphoreTake(mutexB);
xSemaphoreTake(mutexA);

// critical section

xSemaphoreGive(mutexA);
xSemaphoreGive(mutexB);
```

---

#  What Happens

```text id="flow2"
Task 1 locks A → waits for B
Task 2 locks B → waits for A
System stuck forever
```

---

#  RTOS Deadlock Flow

```text id="flow3"
Task A locks Mutex 1
Task B locks Mutex 2

Task A waits for Mutex 2
Task B waits for Mutex 1

→ System freeze
```

---

#  Real Embedded Impact

* system freeze (no task execution)
* watchdog reset trigger
* loss of communication (UART/SPI/CAN stuck)
* motor/control system halt
* safety-critical failure

---

#  Embedded Example: UART + Sensor

```c id="ex3"
Task UART:
    lock UART mutex
    wait sensor data

Task Sensor:
    lock sensor mutex
    wait UART resource
```

If both wait → deadlock.

---

#  Why Embedded Systems Are Vulnerable

* multiple RTOS tasks
* shared peripherals (UART, SPI, I2C)
* mutex-heavy synchronization
* interrupts interacting with tasks
* strict timing constraints

---

#  How to Prevent Deadlocks

---

## 1. Resource Ordering (BEST PRACTICE)

Always lock resources in same order.

```text id="sol1"
Always: A → B → C
Never:  B → A
```

---

## 2. Timeout on Locks

```c id="sol2"
if(xSemaphoreTake(mutex, 100)) {
    // proceed
} else {
    // handle failure
}
```

---

## 3. Avoid Nested Locks

```text id="sol3"
Do NOT lock multiple resources inside each other
```

---

## 4. Use Try-Lock Mechanism

---

## 5. Minimize Critical Sections

---

#  Deadlock Prevention Flow

```text id="flow4"
Request resource
      |
      v
Try lock with timeout
      |
      v
If success → continue
      |
      v
If fail → release and retry
```

---

# ⚠ Common Mistakes

---

## 1. Locking resources in different order

```text id="m1"
A locks B, C
B locks C, A →  deadlock risk
```

---

## 2. Forgetting to release mutex

```c id="m2"
xSemaphoreTake(mutex);
// missing give → deadlock
```

---

## 3. Long critical sections

```text id="m3"
holding lock too long blocks system
```

---

## 4. Circular dependency design

---

# 📊 Deadlock vs Race Condition

| Feature  | Deadlock               | Race Condition  |
| -------- | ---------------------- | --------------- |
| State    | System stuck           | Wrong output    |
| Progress | None                   | Continues       |
| Cause    | Resource waiting cycle | Timing issue    |
| Severity | System freeze          | Data corruption |

---


