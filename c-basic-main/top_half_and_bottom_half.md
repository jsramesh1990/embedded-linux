# Top Half and Bottom Half in Embedded Linux

## Overview

Linux interrupt handling is divided into:
- Top Half
- Bottom Half

This design improves:
- interrupt latency
- system responsiveness
- multitasking efficiency

The idea is:
```text
Do urgent work immediately
Defer non-urgent work later
```

This mechanism is extremely important in:
- Linux kernel
- embedded Linux
- device drivers
- real-time systems

---

# 1. Why Top Half and Bottom Half Exist

Interrupts must be handled:
```text
VERY FAST
```

If interrupt handlers take too long:
- CPU responsiveness decreases
- other interrupts delayed
- real-time behavior suffers

Linux splits interrupt handling into:
- immediate critical work
- deferred non-critical work

---

# 2. Basic Concept

| Part | Purpose |
|------|---------|
| Top Half | Immediate interrupt response |
| Bottom Half | Deferred processing |

---

# 3. High-Level Interrupt Flow

```text
Hardware Interrupt
        ↓
Top Half Executes
        ↓
Schedules Bottom Half
        ↓
Returns Quickly
        ↓
Bottom Half Executes Later
``` id="flow1"

---

# 4. Top Half Definition

Top Half =
```text
Immediate interrupt handler (ISR)
```

Runs:
```text
immediately after interrupt occurs
```

---

# 5. Bottom Half Definition

Bottom Half =
```text
Deferred interrupt processing
```

Runs:
```text
later outside immediate IRQ context
```

---

# 6. Linux Interrupt Architecture

```text
+---------------------------+
| Hardware Device           |
+---------------------------+
            ↓
+---------------------------+
| Interrupt Controller      |
+---------------------------+
            ↓
+---------------------------+
| Top Half (ISR)            |
+---------------------------+
            ↓
+---------------------------+
| Bottom Half Mechanism     |
|---------------------------|
| SoftIRQ                   |
| Tasklet                   |
| Workqueue                 |
+---------------------------+
``` id="arch1"

---

# 7. Top Half Responsibilities

Top Half should:
- acknowledge interrupt
- read device status
- stop interrupt source
- collect critical data
- schedule deferred work

---

# 8. Bottom Half Responsibilities

Bottom Half handles:
- long processing
- packet handling
- buffer management
- filesystem work
- protocol processing

---

# 9. Top Half Characteristics

| Feature | Description |
|---------|-------------|
| Fast | Yes |
| Interrupt Context | Yes |
| Sleep Allowed | No |
| Blocking Allowed | No |

---

# 10. Bottom Half Characteristics

| Feature | Description |
|---------|-------------|
| Deferred execution | Yes |
| Longer processing | Yes |
| Sleep possible | Depends |
| Lower urgency | Yes |

---

# 11. Top Half Execution Flow

```text
Interrupt Occurs
      ↓
CPU Saves Context
      ↓
ISR Executes
      ↓
Critical Work Done
      ↓
Bottom Half Scheduled
      ↓
Return from Interrupt
``` id="top1"

---

# 12. Why Top Half Must Be Short

Long ISR execution causes:
- high interrupt latency
- delayed scheduling
- blocked interrupts

---

# 13. Top Half Example

UART receive interrupt:

```text
Read received byte quickly
Store in buffer
Schedule further processing
``` id="uart1"

---

# 14. Bottom Half Example

Networking:

```text
ISR receives packet
Bottom half processes TCP/IP stack
``` id="net1"

---

# 15. Linux Bottom Half Mechanisms

Linux provides:
- SoftIRQ
- Tasklets
- Workqueues

---

# 16. SoftIRQ

SoftIRQ =
```text
low-level deferred interrupt mechanism
```

Used for:
- networking
- block devices

---

# 17. SoftIRQ Characteristics

| Feature | Description |
|---------|-------------|
| High performance | Yes |
| Concurrent on SMP | Yes |
| Sleep Allowed | No |

---

# 18. SoftIRQ Flow

```text
Top Half Schedules SoftIRQ
         ↓
Kernel Executes SoftIRQ Later
``` id="soft1"

---

# 19. Tasklets

Tasklets are:
```text
simplified SoftIRQs
```

Used widely in drivers.

---

# 20. Tasklet Characteristics

| Feature | Description |
|---------|-------------|
| Easier than SoftIRQ | Yes |
| Serialized execution | Yes |
| Sleep Allowed | No |

---

# 21. Tasklet Flow

```text
ISR Schedules Tasklet
       ↓
Tasklet Executes Later
``` id="task1"

---

# 22. Workqueues

Workqueues execute deferred work using:
```text
kernel threads
```

---

# 23. Workqueue Characteristics

| Feature | Description |
|---------|-------------|
| Sleep Allowed | Yes |
| Process Context | Yes |
| Flexible | Yes |

---

# 24. Workqueue Flow

```text
ISR Schedules Work
       ↓
Kernel Thread Executes Work
``` id="work1"

---

# 25. Bottom Half Comparison

| Mechanism | Sleep Allowed | Context |
|-----------|---------------|---------|
| SoftIRQ | No | Interrupt |
| Tasklet | No | Interrupt |
| Workqueue | Yes | Process |

---

# 26. Linux IRQ Handler Example

```c
irqreturn_t my_irq(int irq, void *dev)
{
    /* top half work */

    return IRQ_HANDLED;
}
``` id="irq1"

---

# 27. Scheduling Workqueue Example

```c
schedule_work(&my_work);
``` id="work2"

---

# 28. Tasklet Example

```c
tasklet_schedule(&my_tasklet);
``` id="task2"

---

# 29. Top Half Restrictions

Inside top half:
- no sleeping
- no mutex
- no blocking I/O

---

# 30. Why Sleeping Not Allowed

ISR runs in:
```text
interrupt context
```

No process exists to sleep/wakeup.

---

# 31. Allowed Synchronization in Top Half

Allowed:
- spinlocks
- atomic operations

Not allowed:
- mutex
- semaphore sleep

---

# 32. IRQ-safe Spinlock Example

```c
spin_lock_irqsave(&lock, flags);

/* critical section */

spin_unlock_irqrestore(&lock, flags);
``` id="spin1"

---

# 33. Deferred Work Benefits

Bottom halves:
- reduce ISR latency
- improve responsiveness
- allow complex processing

---

# 34. Networking Example

```text
NIC Interrupt
      ↓
ISR Reads Packet Descriptor
      ↓
SoftIRQ Processes Packet
``` id="net2"

---

# 35. Storage Driver Example

```text
Disk IRQ
    ↓
ISR Acknowledges Completion
    ↓
Workqueue Handles Filesystem Work
``` id="disk1"

---

# 36. Embedded Linux Usage

Common uses:
- UART drivers
- Ethernet drivers
- SPI/I2C drivers
- audio drivers

---

# 37. DMA and Bottom Halves

DMA completion IRQ:
```text
Top half acknowledges DMA
Bottom half processes data
``` id="dma1"

---

# 38. Real-Time Linux

RT systems require:
- minimal ISR execution
- deterministic latency

Bottom halves very important.

---

# 39. PREEMPT_RT Changes

PREEMPT_RT:
- threads interrupt handlers
- reduces hard IRQ work
- improves determinism

---

# 40. Threaded IRQs

Modern RT Linux often uses:
```text
threaded interrupts
```

---

# 41. Threaded IRQ Flow

```text
Hardware Interrupt
       ↓
Minimal Hard IRQ
       ↓
Wake IRQ Thread
       ↓
Thread Handles Work
``` id="thread1"

---

# 42. Top Half vs Bottom Half Timing

```text
Interrupt
   ↓
Top Half → microseconds
   ↓
Bottom Half → deferred processing
``` id="time1"

---

# 43. CPU Usage Optimization

Deferring work:
- keeps CPU responsive
- reduces interrupt blocking

---

# 44. Common Problems

| Problem | Description |
|---------|-------------|
| Long ISR | High latency |
| Too much deferred work | CPU overload |
| Improper locking | Deadlocks |

---

# 45. Interrupt Storm Problem

Too many interrupts may:
- overload top halves
- starve bottom halves

---

# 46. Debugging Interrupts

---

## View IRQ Stats

```bash
cat /proc/interrupts
``` id="dbg1"

---

## Trace Interrupts

```bash
ftrace
``` id="dbg2"

---

## Performance Monitoring

```bash
perf
``` id="dbg3"

---

# 47. Kernel Internal Files

Important directories:

```text
kernel/irq/
kernel/softirq.c
```

---

# 48. SoftIRQ Types

Common SoftIRQs:
- NET_RX_SOFTIRQ
- NET_TX_SOFTIRQ
- BLOCK_SOFTIRQ

---

# 49. ksoftirqd Thread

Linux kernel thread:
```text
ksoftirqd
```

handles deferred softirq processing.

---

# 50. Complete Top Half and Bottom Half Workflow

```text
Hardware Event
      ↓
Interrupt Generated
      ↓
Top Half Executes Quickly
      ↓
Schedules Deferred Work
      ↓
Interrupt Returns
      ↓
Bottom Half Executes Later
``` id="final1"

---

# 51. Advantages

| Advantage | Description |
|-----------|-------------|
| Lower interrupt latency | Faster response |
| Better multitasking | CPU available |
| Improved responsiveness | Yes |

---

# 52. Disadvantages

| Disadvantage | Description |
|-------------|-------------|
| More complexity | Harder debugging |
| Synchronization needed | Yes |

---

# 53. Top Half vs Bottom Half Summary Table

| Feature | Top Half | Bottom Half |
|---------|----------|-------------|
| Execution Time | Immediate | Deferred |
| Speed | Very fast | Longer |
| Sleep Allowed | No | Depends |
| Context | Interrupt | Deferred/process |
| Main Goal | Acknowledge IRQ | Heavy processing |

---

# 54. Real Embedded Linux Example

```text
UART Receives Data
       ↓
Top Half Reads UART Register
       ↓
Tasklet Processes Buffer
       ↓
Application Reads Data
``` id="uart2"

---

# 55. Summary

- Linux divides interrupt handling into top half and bottom half
- Top half handles urgent work immediately
- Bottom half performs deferred processing
- Improves responsiveness and interrupt latency
- Bottom halves include SoftIRQ, Tasklets, and Workqueues
- Critical in embedded Linux driver development

---


````
