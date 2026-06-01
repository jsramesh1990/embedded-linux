# inconsistent_system_states

---

#  Introduction

In embedded systems, an **inconsistent system state** occurs when different parts of the system hold **conflicting or outdated information at the same time**, leading to unpredictable behavior.

This usually happens in multitasking environments (RTOS), interrupt-driven systems, or shared-memory architectures.

---

#  Definition

An inconsistent system state is a condition where **shared data or system variables do not represent a single valid snapshot of the system at a given time**.

---

#  Why It Happens

* race conditions
* missing synchronization (mutex/semaphore)
* interrupt + task conflicts
* partial updates of shared data
* multi-step operations without protection
* DMA writing while CPU reads data

---

#  Inconsistent State Flow

```text id="flow1"
Task A updates part of data
        |
Interrupt occurs
        |
Task B reads partially updated data
        |
System sees mixed values
        |
 Invalid system state
```

---

#  Example of Inconsistent State

```text id="diag1"
Before:
[ SPEED = 100 ][ MODE = AUTO ][ TEMP = 30 ]

During update:
[ SPEED = 200 ][ MODE = ??? ][ TEMP = 30 ] 

After corruption:
System uses mixed configuration
```

---

#  Embedded System Example

## Shared Configuration Structure

```c id="ex1"
typedef struct {
    int speed;
    int mode;
    int temp;
} SystemConfig;

SystemConfig config;
```

---

## Problem Scenario

```c id="ex2"
void taskA() {
    config.speed = 100;
    config.mode = 1;
    config.temp = 30;
}

void ISR() {
    config.speed = 200; // interrupt during update
}
```

---

#  What Goes Wrong

```text id="flow2"
Task A partially updates structure
        |
ISR modifies one field
        |
Task B reads mixed values
        |
System behaves incorrectly
```

---

#  Real Embedded Example

## Motor Control System

```c id="ex3"
config.speed = 100;
config.direction = FORWARD;
config.enable = 1;
```

If interrupt occurs mid-update:

* motor may run with wrong speed or direction
* unsafe actuator behavior

---

#  Why Embedded Systems Are Vulnerable

* no atomic struct updates
* interrupts can preempt tasks anytime
* shared global variables
* no MMU protection
* real-time constraints prevent locking sometimes

---

#  Inconsistent State Lifecycle

```text id="flow3"
Valid state
      |
Partial update starts
      |
Interrupt/task switch
      |
Mixed data observed
      |
System enters invalid state
```

---

#  Multi-Step Update Problem

```text id="diag2"
Step 1: speed = 100
Step 2: mode  = AUTO   ← interrupt here
Step 3: temp  = 30

Reader sees:
speed = 100
mode  = OLD or NEW (uncertain)
temp  = 30
```

---

#  Impact in Embedded Systems

* unstable motor control
* wrong sensor calibration
* communication mismatch
* safety violations
* system crashes or watchdog reset

---

#  How to Prevent Inconsistent States

---

## 1. Use Critical Sections

```c id="sol1"
__disable_irq();

config.speed = 100;
config.mode = 1;
config.temp = 30;

__enable_irq();
```

---

## 2. Use Mutex in RTOS

```c id="sol2"
xSemaphoreTake(mutex, portMAX_DELAY);

update_config();

xSemaphoreGive(mutex);
```

---

## 3. Atomic Updates (if supported)

```c id="sol3"
__atomic_store_n(&config, newConfig, __ATOMIC_SEQ_CST);
```

---

## 4. Copy-Update Pattern (Safe Design)

```text id="sol4"
Update temp structure → then replace original pointer
```

---

## 5. Avoid Partial Writes

---

#  Safe Update Flow

```text id="flow4"
Create temp copy
        |
Update all fields safely
        |
Replace original atomically
        |
System reads consistent data
```

---

#  Common Mistakes

---

## 1. Updating shared struct without lock

```text id="m1"
leads to partial reads
```

---

## 2. Assuming single line update is safe

```c id="m2"
config.speed = 100; // not atomic in structure context
```

---

## 3. Ignoring interrupt timing

---

## 4. Not using synchronization in RTOS

---

#  Inconsistent State vs Data Corruption

| Feature    | Inconsistent State    | Data Corruption    |
| ---------- | --------------------- | ------------------ |
| Nature     | Logical mismatch      | Invalid data       |
| Cause      | timing/sync issue     | overwrite/hardware |
| Visibility | subtle                | often obvious      |
| Effect     | wrong system behavior | wrong values/crash |

---

#  Real-Time Embedded Impact

* unsafe actuator behavior
* incorrect control logic
* unstable system outputs
* safety-critical failures
* unpredictable firmware behavior

---

#  Key Idea

> Inconsistent state is not always “wrong data” — it is often **correct data at the wrong time window**.

---

