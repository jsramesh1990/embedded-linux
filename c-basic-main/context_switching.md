# Context Switching in Embedded Linux

## Overview

Context Switching is the process where the CPU:
- stops executing one task
- saves its execution state
- loads another task’s state
- resumes execution of another task

Context switching is fundamental for:
- multitasking
- process scheduling
- real-time systems
- Linux kernel operation

Without context switching:
```text
only one process could run at a time
```

---

# 1. What is Context Switching?

Context switching means:

```text
Switching CPU execution from one task to another
```

A task may be:
- process
- thread
- kernel thread

---

# 2. Why Context Switching is Needed

Modern systems execute:
- multiple applications
- kernel threads
- interrupts
- background services

CPU time must be shared among tasks.

---

# 3. High-Level Context Switch Flow

```text
Running Task A
       ↓
Scheduler Decision
       ↓
Save Task A Context
       ↓
Load Task B Context
       ↓
Resume Task B
``` id="flow1"

---

# 4. What is "Context"?

Context means:
```text
CPU execution state of a task
```

Includes:
- CPU registers
- program counter
- stack pointer
- memory mappings

---

# 5. Components of Context

| Component | Purpose |
|-----------|---------|
| CPU Registers | Current computations |
| Program Counter (PC) | Next instruction |
| Stack Pointer (SP) | Stack location |
| MMU State | Virtual memory mapping |
| CPU Flags | Status information |

---

# 6. Linux Task Structure

Linux stores process/thread info in:

```c
task_struct
```

Contains:
- scheduling info
- memory info
- CPU context

---

# 7. Context Switching Architecture

```text
+--------------------------+
| Linux Scheduler          |
+--------------------------+
| Run Queue                |
| Context Switcher         |
| task_struct              |
+--------------------------+
| CPU Hardware             |
+--------------------------+
``` id="arch1"

---

# 8. Scheduler and Context Switching

Scheduler decides:
```text
which task runs next
```

Context switch performs:
```text
actual task transition
```

---

# 9. When Context Switch Happens

| Event | Description |
|------|-------------|
| Time slice expiry | Scheduler preemption |
| Blocking system call | Task waits |
| Interrupt | Higher priority task |
| Sleep/wakeup | Task state change |

---

# 10. Time Slice Expiry

Scheduler interrupts running task:

```text
preemption
```

---

# 11. Blocking Operation Example

```c
read()
```

Task waits for:
- disk
- network
- UART

Scheduler switches to another task.

---

# 12. Interrupt-triggered Switch

```text
Interrupt occurs
      ↓
Higher Priority Task Ready
      ↓
Scheduler Switches Tasks
``` id="irq1"

---

# 13. Process States

| State | Meaning |
|------|---------|
| Running | Executing |
| Runnable | Ready for CPU |
| Sleeping | Waiting |
| Stopped | Suspended |

---

# 14. Context Switch Lifecycle

```text
Task Running
     ↓
Scheduler Invoked
     ↓
Save Current Context
     ↓
Select Next Task
     ↓
Load New Context
     ↓
Resume Execution
``` id="life1"

---

# 15. Saving Context

Kernel saves:
- general registers
- stack pointer
- program counter
- CPU flags

---

# 16. Loading Context

Kernel restores:
- registers
- stack
- execution point

for next task.

---

# 17. CPU Registers Saved

Typical registers:
- R0-R15 (ARM)
- RIP/RSP (x86)
- status registers

---

# 18. Stack Switching

Each process/thread has:
```text
its own stack
```

Context switch changes:
```text
stack pointer
```

---

# 19. MMU Context Switch

Different processes have:
```text
different virtual address spaces
```

Kernel updates:
```text
page tables
```

---

# 20. TLB Impact

MMU translation cache:
```text
TLB (Translation Lookaside Buffer)
```

may require flushing during switch.

---

# 21. Process vs Thread Switching

| Type | Memory Space |
|------|--------------|
| Process Switch | Different |
| Thread Switch | Usually shared |

---

# 22. Thread Context Switching

Thread switching cheaper because:
```text
same address space
```

---

# 23. Kernel Threads

Linux kernel also switches:
```text
kernel threads
```

Examples:
- kworker
- ksoftirqd

---

# 24. Context Switching Flow Diagram

```text
Task A Running
      ↓
Interrupt/Scheduler Tick
      ↓
Save Registers
      ↓
Update task_struct
      ↓
Select Task B
      ↓
Load Task B Registers
      ↓
Task B Running
``` id="ctx1"

---

# 25. Linux Scheduler Tick

Periodic timer interrupt:
```text
scheduler tick
```

Triggers:
- accounting
- preemption checks

---

# 26. Software Context Switch

Performed by:
```text
kernel scheduler code
```

---

# 27. Hardware Assistance

CPU provides:
- interrupt support
- privileged mode
- register handling

---

# 28. ARM Context Switching

ARM saves:
- R0-R15
- CPSR
- stack pointer

---

# 29. x86 Context Switching

x86 saves:
- RIP
- RSP
- general-purpose registers

---

# 30. SMP Context Switching

Multi-core systems:
- each CPU has scheduler
- per-CPU run queues

---

# 31. SMP Flow

```text
CPU0 → Task A
CPU1 → Task B
CPU2 → Task C
``` id="smp1"

---

# 32. Voluntary Context Switch

Task willingly gives up CPU:
```text
sleep()
wait()
```

---

# 33. Involuntary Context Switch

Scheduler forcibly preempts task:
```text
time slice expired
```

---

# 34. Measuring Context Switches

```bash
vmstat
``` id="vm1"

---

# 35. View Process Switches

```bash
pidstat -w
``` id="pid1"

---

# 36. Checking Scheduler Activity

```bash
top
``` id="top1"

---

# 37. Linux Context Switch Code

Important kernel files:

```text
kernel/sched/
```

Architecture-specific code:

```text
arch/arm/kernel/
```

---

# 38. switch_to() Macro

Linux uses:

```c
switch_to()
```

for low-level switching.

---

# 39. switch_to Flow

```text
Save Previous Task
       ↓
Load Next Task
       ↓
Resume Execution
``` id="switch1"

---

# 40. Context Switch Overhead

Switching costs:
- CPU cycles
- cache misses
- TLB flushes

---

# 41. Why Excessive Switching is Bad

Too many switches cause:
- reduced performance
- cache invalidation
- scheduling overhead

---

# 42. Real-Time Context Switching

RT systems require:
- predictable switching latency
- fast response

---

# 43. PREEMPT_RT

PREEMPT_RT patch:
- reduces latency
- improves determinism
- increases preemptibility

---

# 44. Interrupt Context vs Process Context

| Feature | Interrupt Context | Process Context |
|---------|------------------|----------------|
| Sleep allowed | No | Yes |
| Has process state | No | Yes |
| Scheduler managed | Limited | Yes |

---

# 45. Context Switching and IPC

IPC often triggers:
- wakeups
- blocking
- scheduler switches

---

# 46. Embedded Linux Context Switching

Embedded systems require:
- low latency
- efficient scheduling
- deterministic behavior

---

# 47. Context Switching Example

```text
Task A reads UART
      ↓
Blocks Waiting for Data
      ↓
Scheduler Switches to Task B
      ↓
UART Interrupt Arrives
      ↓
Task A Wakes Up
``` id="uart1"

---

# 48. Context Switching and Power Management

Idle tasks may:
```text
put CPU into low-power state
```

---

# 49. Context Switch Timing

Switch time depends on:
- CPU architecture
- cache state
- MMU/TLB behavior

Typical:
```text
microseconds
```

---

# 50. Context Switch Debugging

---

## View Scheduler Stats

```bash
cat /proc/schedstat
``` id="dbg1"

---

## Trace Scheduler

```bash
ftrace
``` id="dbg2"

---

## Performance Analysis

```bash
perf sched
``` id="dbg3"

---

# 51. Common Context Switching Problems

| Problem | Description |
|---------|-------------|
| Excessive switching | Performance loss |
| Priority inversion | High-priority delays |
| Cache thrashing | Reduced efficiency |
| Latency spikes | RT failure |

---

# 52. Context Switch Optimization

- reduce lock contention
- reduce blocking
- use CPU affinity
- minimize thread count

---

# 53. CPU Affinity Example

```bash
taskset -c 0 ./app
``` id="aff1"

---

# 54. Complete Context Switch Workflow

```text
Task Running
     ↓
Interrupt / Scheduler Event
     ↓
Save CPU Context
     ↓
Update task_struct
     ↓
Select Next Runnable Task
     ↓
Load New Task Context
     ↓
Resume Execution
``` id="final1"

---

# 55. Advantages of Context Switching

- multitasking support
- CPU sharing
- responsive systems
- concurrent execution

---

# 56. Disadvantages

- overhead
- cache invalidation
- TLB flushing
- scheduler complexity

---

# 57. Summary

- Context switching allows multitasking
- Linux scheduler selects runnable tasks
- Kernel saves/restores CPU state
- Essential for process/thread scheduling
- Important for embedded and real-time Linux systems
- Excessive switching reduces performance

---
````
