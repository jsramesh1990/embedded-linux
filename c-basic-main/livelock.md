# livelock

---

#  Introduction

In embedded systems and RTOS, a **livelock** is a condition where tasks are **actively running and responding to each other**, but **no useful progress is made**.

Unlike a deadlock (where tasks are stuck), in a livelock:

* tasks are not blocked
* CPU is busy
* system is still “alive”
* but work never completes

---

#  Definition

A livelock is a state where two or more tasks **continuously change their state in response to each other without making actual progress**.

---

#  Simple Idea

```text id="idea1"
Deadlock → no movement 
Livelock → too much movement, no progress 
```

---

#  Livelock Flow

```text id="flow1"
Task A tries resource → fails → backs off
Task B tries resource → fails → backs off
Task A retries again → fails
Task B retries again → fails
        |
Loop continues forever 
```

---

#  Livelock Diagram

```text id="diag1"
Task A ↔ Task B
   ↑         ↓
retry     retry
   ↓         ↑
give way  give way
   ↺ endlessly ↻
(No progress made)
```

---

#  Embedded System Example

## Shared Resource Retry Problem

```c id="ex1"
int lockA = 0;
int lockB = 0;

void taskA() {
    while(1) {
        if(lockA == 0) {
            lockA = 1;
            if(lockB == 0) {
                lockB = 1;
                // work done
                lockB = 0;
            }
            lockA = 0;
        }
    }
}
```

---

## Task B

```c id="ex2"
void taskB() {
    while(1) {
        if(lockB == 0) {
            lockB = 1;
            if(lockA == 0) {
                lockA = 1;
                // work done
                lockA = 0;
            }
            lockB = 0;
        }
    }
}
```

---

#  What Happens

```text id="flow2"
Task A locks A
Task B locks B

Both detect conflict
Both release and retry
Both repeat endlessly 
```

---

#  Real Embedded Example

## Communication Bus Arbitration

* Task A and Task B both try SPI bus
* both back off when bus is busy
* both retry immediately
* bus is never used effectively

---

#  Livelock Timeline

```text id="timeline1"
Time →
A: try → backoff → try → backoff → try
B: try → backoff → try → backoff → try

Result: CPU busy, no progress
```

---

#  Causes of Livelock

---

## 1. Aggressive retry logic

* tasks keep retrying immediately

---

## 2. Poor backoff strategy

* no delay or random wait

---

## 3. Symmetric behavior

* both tasks behave identically

---

## 4. Shared resource contention

* both constantly yield to each other

---

#  Livelock vs Deadlock

| Feature      | Livelock | Deadlock |
| ------------ | -------- | -------- |
| CPU usage    | High     | None     |
| Task state   | Active   | Blocked  |
| Progress     | None     | None     |
| Recovery     | Hard     | Hard     |
| System state | Running  | Frozen   |

---

#  Key Insight

> In livelock, the system is **busy doing nothing useful**.

---

#  Embedded Example: Sensor Bus Conflict

```c id="ex3"
Task A: if bus busy → retry immediately
Task B: if bus busy → retry immediately
```

Result:

* continuous retry loop
* no actual data transfer

---

#  Livelock Flow in Embedded System

```text id="flow3"
Task A requests resource
Task B requests resource
Both detect conflict
Both yield to each other
Both retry immediately
Loop continues forever
```

---

#  Impact in Embedded Systems

* CPU wasted
* no real task completion
* increased power consumption
* missed deadlines
* system inefficiency

---

#  How to Prevent Livelock

---

## 1. Introduce random backoff

```c id="sol1"
delay(random_time);
```

---

## 2. Use priority rules

```text id="sol2"
Task with higher priority gets resource
```

---

## 3. Limit retries

```c id="sol3"
retry_count++;
if(retry_count > N) break;
```

---

## 4. Use mutex/semaphore properly

---

## 5. Avoid symmetric retry logic

---

#  Livelock Prevention Flow

```text id="flow4"
Resource busy detected
        |
Wait (backoff delay)
        |
Retry after delay
        |
Eventually one succeeds
```

---

#  Common Mistakes

---

## 1. Immediate retry without delay

```text id="m1"
causes infinite livelock loop
```

---

## 2. Identical task behavior

---

## 3. No retry limit

---

## 4. Poor synchronization design

---

#  Real-World Analogy

Two people trying to pass each other in a narrow corridor:

* both step left → blocked
* both step right → blocked
* both keep adjusting → never pass

---


