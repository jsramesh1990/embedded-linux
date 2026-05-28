# Synchronization in Embedded Linux

## Overview

Synchronization is the mechanism used to:
- coordinate multiple tasks
- protect shared resources
- prevent race conditions
- maintain data consistency

Synchronization is critical in:
- Linux kernel
- embedded systems
- multithreading
- multiprocessing
- interrupt handling

Without synchronization:
- memory corruption
- unpredictable behavior
- crashes
- deadlocks

can occur.

---

# 1. What is Synchronization?

Synchronization means:

```text
Controlling concurrent access to shared resources
```

Used when:
- multiple processes
- threads
- interrupts
- CPUs

access the same data.

---

# 2. Why Synchronization is Needed

Modern systems are:
- multitasking
- multicore
- interrupt-driven

Multiple execution paths may access:
- variables
- hardware
- buffers
- queues

simultaneously.

---

# 3. Shared Resource Problem

Example:

```text
CPU0 increments variable
CPU1 increments same variable
```

Without synchronization:
```text
data corruption possible
```

---

# 4. Race Condition

Occurs when:
```text
result depends on execution timing
```

---

# 5. Race Condition Example

```c
counter++;
```

Actually:

```text
Read counter
Add 1
Write counter
```

Two CPUs may overlap operations.

---

# 6. Race Condition Flow

```text
CPU0 Reads Counter = 5
CPU1 Reads Counter = 5
CPU0 Writes 6
CPU1 Writes 6
```

Expected:
```text
7
```

Actual:
```text
6
``` id="race1"

---

# 7. Synchronization Goals

Synchronization ensures:
- mutual exclusion
- ordering
- consistency
- safe concurrency

---

# 8. Critical Section

Code accessing shared resource:

```text
Critical Section
```

Must allow:
```text
only one execution path at a time
```

---

# 9. Synchronization Mechanisms

| Mechanism | Purpose |
|-----------|---------|
| Mutex | Mutual exclusion |
| Spinlock | Busy waiting lock |
| Semaphore | Resource counting |
| RW Lock | Multiple readers |
| Atomic Operations | Single instruction operations |
| Completion | Event synchronization |

---

# 10. Linux Synchronization Architecture

```text
+-------------------------+
| User Processes          |
+-------------------------+
| Linux Kernel            |
|-------------------------|
| Mutexes                 |
| Spinlocks               |
| Semaphores              |
| Atomic Operations       |
| Wait Queues             |
+-------------------------+
| CPU / Hardware          |
+-------------------------+
``` id="arch1"

---

# 11. Mutex

Mutex =
```text
Mutual Exclusion
```

Allows:
```text
only one thread/process
```

to enter critical section.

---

# 12. Mutex Flow

```text
Task Requests Mutex
       ↓
Mutex Free?
   ↓ Yes      ↓ No
Acquire       Sleep
Mutex         Wait
``` id="mutex1"

---

# 13. Mutex Example

```c
struct mutex my_mutex;

mutex_lock(&my_mutex);

/* critical section */

mutex_unlock(&my_mutex);
``` id="mutex2"

---

# 14. Mutex Characteristics

| Feature | Description |
|---------|-------------|
| Sleeping lock | Yes |
| Process context only | Yes |
| Interrupt safe | No |

---

# 15. Spinlock

Spinlock uses:
```text
busy waiting
```

instead of sleeping.

---

# 16. Spinlock Flow

```text
Acquire Lock
     ↓
Lock Busy?
 ↓ Yes       ↓ No
Spin         Enter Critical Section
``` id="spin1"

---

# 17. Spinlock Example

```c
spinlock_t lock;

spin_lock(&lock);

/* critical section */

spin_unlock(&lock);
``` id="spin2"

---

# 18. Spinlock Characteristics

| Feature | Description |
|---------|-------------|
| Busy wait | Yes |
| Sleep allowed | No |
| Interrupt safe | Yes |
| Fast | Yes |

---

# 19. When to Use Spinlocks

Used when:
- critical section very short
- interrupt context
- SMP systems

---

# 20. Mutex vs Spinlock

| Feature | Mutex | Spinlock |
|---------|-------|----------|
| Sleeping | Yes | No |
| Busy wait | No | Yes |
| Interrupt context | No | Yes |
| Long critical section | Good | Bad |

---

# 21. Semaphore

Semaphore controls:
```text
multiple resource access
```

---

# 22. Semaphore Types

| Type | Purpose |
|------|---------|
| Binary Semaphore | Similar to mutex |
| Counting Semaphore | Multiple resources |

---

# 23. Semaphore Example

```c
struct semaphore sem;

down(&sem);

/* critical section */

up(&sem);
``` id="sem1"

---

# 24. Counting Semaphore Example

Allows:
```text
N simultaneous users
```

---

# 25. Read-Write Lock (RW Lock)

Allows:
- multiple readers
- single writer

---

# 26. RW Lock Flow

```text
Readers Allowed Together
        ↓
Writer Requires Exclusive Access
``` id="rw1"

---

# 27. RW Lock Example

```c
rwlock_t lock;

read_lock(&lock);
/* read */
read_unlock(&lock);

write_lock(&lock);
/* write */
write_unlock(&lock);
``` id="rw2"

---

# 28. Atomic Operations

Operations executed:
```text
without interruption
```

---

# 29. Atomic Variable Example

```c
atomic_t counter;

atomic_inc(&counter);
``` id="atomic1"

---

# 30. Atomic Operations Advantages

- very fast
- lock-free
- interrupt-safe

---

# 31. Common Atomic Operations

| Operation | Function |
|-----------|----------|
| Increment | atomic_inc |
| Decrement | atomic_dec |
| Read | atomic_read |
| Set | atomic_set |

---

# 32. Wait Queues

Used when process waits for:
- event
- resource
- hardware completion

---

# 33. Wait Queue Flow

```text
Process Sleeps
      ↓
Event Occurs
      ↓
Wakeup Signal
      ↓
Process Resumes
``` id="wait1"

---

# 34. Wait Queue Example

```c
wait_event(queue, condition);
``` id="wait2"

---

# 35. Completions

Used for:
```text
one-time event synchronization
```

---

# 36. Completion Example

```c
DECLARE_COMPLETION(done);

wait_for_completion(&done);

complete(&done);
``` id="comp1"

---

# 37. Interrupt Synchronization

Interrupt handlers may access:
- shared buffers
- device structures

Need synchronization with:
- processes
- kernel threads

---

# 38. IRQ-safe Spinlock

Example:

```c
spin_lock_irqsave(&lock, flags);

/* critical section */

spin_unlock_irqrestore(&lock, flags);
``` id="irq1"

---

# 39. Why IRQ-safe Locks?

Prevent:
```text
interrupt preemption while lock held
```

---

# 40. Deadlock

Occurs when:
```text
two tasks wait forever for each other
```

---

# 41. Deadlock Example

```text
Task A holds Lock1 waits Lock2
Task B holds Lock2 waits Lock1
``` id="dead1"

---

# 42. Deadlock Prevention

- fixed lock ordering
- avoid nested locks
- use timeout locks

---

# 43. Starvation

Occurs when:
```text
task never gets resource
```

---

# 44. Priority Inversion

Low-priority task blocks high-priority task.

Solution:
```text
priority inheritance
```

---

# 45. SMP Synchronization

Multi-core systems require:
- cache coherence
- memory barriers
- inter-CPU synchronization

---

# 46. Memory Barriers

Prevent CPU/compiler reordering.

Example:

```c
mb();
``` id="mb1"

---

# 47. Synchronization in User Space

User-space synchronization:
- pthread mutex
- semaphores
- futexes

---

# 48. pthread Mutex Example

```c
pthread_mutex_lock(&lock);

/* critical section */

pthread_mutex_unlock(&lock);
``` id="pthread1"

---

# 49. Futex

Fast Userspace Mutex.

Linux optimization:
```text
userspace lock + kernel fallback
```

---

# 50. Synchronization in Device Drivers

Drivers synchronize:
- interrupts
- DMA
- hardware access
- buffers

---

# 51. Embedded Linux Synchronization Flow

```text
Multiple Tasks/Interrupts
          ↓
Shared Resource Access
          ↓
Synchronization Primitive
          ↓
Protected Critical Section
``` id="emb1"

---

# 52. Real-Time Synchronization

Real-time systems require:
- deterministic locking
- low latency
- priority inheritance

---

# 53. PREEMPT_RT Changes

PREEMPT_RT:
- converts spinlocks to mutexes
- improves determinism
- reduces latency

---

# 54. Common Synchronization Problems

| Problem | Description |
|---------|-------------|
| Deadlock | Infinite waiting |
| Race condition | Data corruption |
| Starvation | No resource access |
| Priority inversion | RT blocking |

---

# 55. Debugging Synchronization

---

## Kernel Lock Debugging

```text
CONFIG_LOCKDEP
```

---

## View Locks

```bash
cat /proc/locks
``` id="dbg1"

---

## Trace Scheduling

```bash
ftrace
``` id="dbg2"

---

# 56. Synchronization Best Practices

- keep critical sections short
- avoid sleeping in spinlocks
- avoid nested locking
- use atomic ops when possible

---

# 57. Complete Synchronization Workflow

```text
Multiple Execution Paths
          ↓
Access Shared Resource
          ↓
Acquire Synchronization Primitive
          ↓
Execute Critical Section
          ↓
Release Lock
``` id="final1"

---

# 58. Advantages of Synchronization

- data consistency
- safe multitasking
- race prevention
- stable kernel behavior

---

# 59. Disadvantages

- overhead
- complexity
- deadlock risk
- performance bottlenecks

---

# 60. Summary

- Synchronization protects shared resources
- Linux provides mutexes, spinlocks, semaphores, atomics
- Essential for multicore and interrupt-driven systems
- Critical in kernel and embedded Linux development
- Proper synchronization prevents race conditions and corruption

---


````
