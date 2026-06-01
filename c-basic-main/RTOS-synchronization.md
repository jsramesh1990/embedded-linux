# RTOS-scheduling

---

#  Introduction

In embedded systems, an **RTOS (Real-Time Operating System)** scheduler decides **which task runs, when it runs, and for how long**.

Unlike general-purpose OS scheduling (like Linux), RTOS scheduling is designed for:

* deterministic timing
* predictable execution
* real-time constraints

It ensures critical tasks meet deadlines.

---

#  Definition

RTOS scheduling is the mechanism by which the RTOS **allocates CPU time to tasks based on priority, state, and scheduling policy**.

---

#  Core Idea

```text id="idea1"
CPU → Shared resource
RTOS Scheduler → Decision maker (who runs next)
```

---

#  RTOS Task States

```text id="flow1"
READY → eligible to run
RUNNING → currently executing
BLOCKED → waiting for event/resource
SUSPENDED → manually stopped
```

---

#  Task State Diagram

```text id="diag1"
        +---------+
        | READY   |
        +----+----+
             |
             v
        +---------+
        | RUNNING |
        +----+----+
             |
   +---------+----------+
   |                    |
   v                    v
BLOCKED             READY (preempted)
   |
   v
READY (event occurs)
```

---

#  Types of RTOS Scheduling

---

## 1. Preemptive Scheduling (Most Common)

* Higher priority task interrupts lower priority task
* Real-time responsive

```text id="flow2"
Low task running
High priority task arrives
→ Low task preempted immediately
```

---

## 2. Cooperative Scheduling

* Task must voluntarily yield CPU
* Simpler but less reliable

```text id="flow3"
Task runs until it calls yield()
```

---

## 3. Round Robin (Time Slicing)

* Equal priority tasks share CPU
* Each gets time quantum

---

# 📊 Preemptive Scheduling Diagram

```text id="diag2"
T1 (Low): ███████---██---
T2 (High): ----████████---
            ↑ preempts T1
```

---

#  Priority-Based Scheduling

Each task is assigned a priority:

```text id="prio1"
Higher number → higher priority (or vice versa depending RTOS)
```

Example:

* Task A (High priority): Sensor read
* Task B (Low priority): Logging

---

#  Embedded Example

```c id="ex1"
void SensorTask(void *p) {
    while(1) {
        read_sensor();
    }
}

void LogTask(void *p) {
    while(1) {
        write_log();
    }
}
```

If SensorTask has higher priority → LogTask gets less CPU time.

---

#  Scheduler Decision Flow

```text id="flow4"
1. Check READY tasks
2. Select highest priority
3. Run task
4. If higher priority arrives → preempt
5. Repeat
```

---

#  RTOS Scheduling Timeline

```text id="timeline1"
Time →
T1 (Low):  ███---██------
T2 (High): ----████████--
T3 (Mid):  --██----██----
```

---

#  Key Scheduling Concepts

---

## 1. Context Switching

Saving and restoring CPU state:

```text id="cs1"
Registers, PC, stack pointer saved/restored
```

---

## 2. Time Slice

Fixed CPU time given to tasks.

---

## 3. Preemption

Higher priority task interrupts current task.

---

## 4. Determinism

Execution timing is predictable.

---

#  RTOS Scheduling Policies

| Policy                        | Description                     |
| ----------------------------- | ------------------------------- |
| Fixed Priority                | Tasks assigned static priority  |
| Dynamic Priority              | Priority changes over time      |
| Round Robin                   | Equal sharing                   |
| EDF (Earliest Deadline First) | Task with nearest deadline runs |

---

#  Embedded RTOS Example (FreeRTOS Style)

```c id="ex2"
xTaskCreate(TaskA, "Sensor", 100, NULL, 2, NULL);
xTaskCreate(TaskB, "Logger", 100, NULL, 1, NULL);
```

* TaskA runs before TaskB

---

#  Scheduling Problems

---

## 1. Starvation

Low priority task never runs.

---

## 2. Priority Inversion

Low task blocks high task via mutex.

---

## 3. Deadlock

Tasks wait forever for resources.

---

## 4. Livelock

Tasks keep retrying without progress.

---

#  Priority Inversion Example

```text id="flow5"
High priority task waiting
Low priority holds mutex
Medium task keeps running
→ High task delayed
```

---

#  Scheduler Optimization Techniques

---

## 1. Priority Inheritance

* Low task temporarily inherits high priority

---

## 2. Priority Ceiling

* Mutex has maximum priority limit

---

## 3. Time slicing tuning

---

## 4. Reduce critical section time

---

#  Real Embedded Example

## Motor Control System

* High priority: control loop (1ms)
* Medium priority: sensor read
* Low priority: logging

Scheduler ensures control loop always runs on time.

---

#  Scheduling Flow (Real System)

```text id="flow6"
Interrupt → wakes high priority task
Scheduler → preempts current task
High priority executes immediately
```

---

#  Why RTOS Scheduling Matters

* ensures real-time deadlines
* prevents system failure
* controls CPU fairness
* manages multitasking efficiently

---

#  Common Mistakes

---

## 1. Giving all tasks high priority

* system becomes unstable

---

## 2. Infinite loops in high priority tasks

```c id="m1"
while(1); // blocks everything
```

---

## 3. Not using delays

---

## 4. Long critical sections

---

#  RTOS Scheduling vs OS Scheduling

| Feature  | RTOS                 | General OS  |
| -------- | -------------------- | ----------- |
| Goal     | Deterministic timing | Fair usage  |
| Latency  | Very low             | Variable    |
| Priority | Strict               | Flexible    |
| Use case | Embedded systems     | PCs/Servers |

---


