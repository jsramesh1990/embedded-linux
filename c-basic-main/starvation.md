# starvation

---

#  Introduction

In embedded systems and RTOS, **starvation** is a scheduling problem where a task is **continuously denied CPU time or resources**, even though the system is running normally.

The task is ready to run, but it never gets a chance because other higher-priority tasks keep dominating the CPU.

---

#  Definition

Starvation is a condition where a task remains in a **ready state indefinitely but is never scheduled for execution due to resource or CPU allocation bias**.

---

#  Why Starvation Happens

* strict priority-based scheduling (RTOS)
* continuous execution of high-priority tasks
* resource locking by other tasks
* poor scheduler design
* unfair resource allocation

---

#  Starvation Flow

```text id="flow1"
Low priority task ready
        |
CPU always given to high priority tasks
        |
Low task keeps waiting
        |
Never gets CPU time 
```

---

#  Starvation Diagram

```text id="diag1"
High Priority Task → RUNNING → RUNNING → RUNNING
Medium Priority    → RUNNING → RUNNING
Low Priority Task  → READY   → READY   → READY  (starved)
```

---

#  Embedded System Example

## RTOS Priority Issue

```c id="ex1"
void HighPriorityTask() {
    while(1) {
        // always running
    }
}

void LowPriorityTask() {
    while(1) {
        // never gets CPU time
    }
}
```

---

#  What Happens

* CPU is always occupied by high-priority task
* low-priority task never scheduled
* system appears partially frozen

---

#  Real Embedded Example

## Sensor vs Control Task

```c id="ex2"
HighPriorityTask:
    reads sensor continuously

LowPriorityTask:
    logs data to memory
```

### Result:

* logging task never runs
* system loses logs → data loss

---

#  Starvation in RTOS Scheduling

```text id="flow2"
Task A (High priority) → always ready
Task B (Low priority) → waiting forever
Task C (Medium) → occasional execution
```

---

#  Starvation Timeline

```text id="timeline1"
Time →
A: ████████████████████
B: ......... (never runs)
C: ████ ████ ████
```

---

#  Causes of Starvation

---

## 1. Priority-based scheduling

High priority tasks dominate CPU.

---

## 2. Infinite loops in high priority tasks

```c id="cause1"
while(1) { }
```

---

## 3. Long critical sections

* blocking lower tasks
* holding mutex too long

---

## 4. Resource contention

* shared locks never released quickly

---

#  Starvation vs Deadlock

| Feature   | Starvation                | Deadlock      |
| --------- | ------------------------- | ------------- |
| State     | Task waiting indefinitely | System stuck  |
| CPU usage | High (other tasks run)    | None          |
| Cause     | scheduling bias           | circular wait |
| Recovery  | possible                  | difficult     |

---

#  Starvation vs Priority Inversion

| Feature | Starvation        | Priority Inversion |
| ------- | ----------------- | ------------------ |
| Cause   | unfair scheduling | lock blocking      |
| Effect  | task never runs   | high task delayed  |
| Scope   | system-level      | resource-level     |

---

#  Embedded Example: UART Logging Starvation

```c id="ex3"
HighPriorityTask:
    handle interrupts

LowPriorityTask:
    write logs to flash
```

### Problem:

* flash logging never executes
* debugging becomes impossible

---

#  Impact in Embedded Systems

* missed logging data
* degraded system functionality
* incomplete sensor processing
* watchdog resets
* system imbalance

---

#  How to Prevent Starvation

---

## 1. Priority Aging

```text id="sol1"
Increase priority of waiting tasks over time
```

---

## 2. Time slicing (Round Robin)

```text id="sol2"
Each task gets CPU time slot
```

---

## 3. Avoid infinite loops in high priority tasks

---

## 4. Use proper task delays

```c id="sol3"
vTaskDelay(10);
```

---

## 5. Balanced priority design

---

#  Starvation Prevention Flow

```text id="flow3"
Task waits too long
        |
Priority increases (aging)
        |
Scheduler selects task
        |
Task executes
```

---

#  RTOS Solution Example

```c id="sol4"
void LowTask() {
    while(1) {
        process();
        vTaskDelay(10);
    }
}
```

---

#  Key Characteristics

* no system halt (unlike deadlock)
* task is alive but inactive
* caused by unfair scheduling
* common in priority-based RTOS

---

#  Common Mistakes

---

## 1. Giving all tasks high priority

```text id="m1"
no scheduling balance
```

---

## 2. Infinite loops in high-priority tasks

---

## 3. Not using delays/yield

---

## 4. Blocking system resources too long

---

#  Starvation Handling Strategies

| Strategy       | Effect             |
| -------------- | ------------------ |
| Priority Aging | Fair scheduling    |
| Round Robin    | Equal CPU share    |
| Time slicing   | Prevent domination |
| RTOS tuning    | Balanced execution |

---


