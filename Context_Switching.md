# Context Switching in Operating Systems

## Table of Contents

1. Introduction
2. Definition
3. Why Context Switching Is Needed
4. What Is Context?
5. How Context Switching Works
6. Context Switching Process
7. Components Saved During Context Switching
8. Types of Context Switching
9. Process Context Switching
10. Thread Context Switching
11. CPU Scheduling and Context Switching
12. Context Switching Overhead
13. Advantages
14. Disadvantages
15. Real-Time Examples
16. Major Project Usage
17. Process vs Thread Context Switching
18. Performance Impact
19. Best Practices
20. Interview Questions
21. Conclusion

---

# 1. Introduction

Modern operating systems run multiple programs simultaneously.

Examples:

* Google Chrome
* VS Code
* Spotify
* Terminal
* Background Services

Since the CPU can execute only one instruction per core at a time, the operating system rapidly switches between tasks.

This switching is called **Context Switching**.

---

# 2. Definition

## Definition

**Context Switching** is the process of saving the current state (context) of a running process or thread and loading the state of another process or thread so that execution can continue from where it stopped.

---

## Simple Definition

```text
Save Current Task
        ↓
Load Next Task
        ↓
Continue Execution
```

---

## Example

Imagine:

```text
Process A Running
```

CPU interrupts it and runs:

```text
Process B Running
```

Later:

```text
Process A Continues
```

The CPU must remember where Process A stopped.

This memory of execution state is called the **Context**.

---

# 3. Why Context Switching Is Needed

Without context switching:

```text
Process A runs
Process B waits
Process C waits
```

CPU utilization becomes inefficient.

---

## Reasons

### Multitasking

Allows multiple applications to run.

---

### Resource Sharing

CPU shared among processes.

---

### Responsiveness

User applications remain interactive.

---

### Fair Scheduling

All processes get CPU time.

---

# 4. What Is Context?

The context is all information needed to resume execution later.

---

## Context Includes

### CPU Registers

```text
R1
R2
R3
...
```

---

### Program Counter (PC)

Stores next instruction address.

---

### Stack Pointer (SP)

Points to current stack location.

---

### Process State

```text
Running
Ready
Waiting
```

---

### Memory Information

Page tables and memory mappings.

---

### Open Resource Information

Files, sockets, handles.

---

# 5. How Context Switching Works

Basic flow:

```text
Running Process A
        ↓
Interrupt Occurs
        ↓
Save Context of A
        ↓
Scheduler Selects B
        ↓
Load Context of B
        ↓
Run Process B
```

---

# 6. Context Switching Process

## Step 1

Process A running.

```text
CPU → Process A
```

---

## Step 2

Interrupt occurs.

Examples:

* Timer interrupt
* I/O request
* Higher-priority process

---

## Step 3

Save Process A context.

Saved into PCB (Process Control Block).

---

## Step 4

Scheduler chooses next process.

```text
Ready Queue
```

---

## Step 5

Load Process B context.

---

## Step 6

CPU executes Process B.

---

## Diagram

```text
Process A
   |
Save Context
   |
Scheduler
   |
Load Context
   |
Process B
```

---

# 7. Components Saved During Context Switching

## CPU Registers

```text
AX
BX
CX
DX
```

---

## Program Counter

Stores next instruction.

---

## Stack Pointer

Stores current stack position.

---

## Status Register

Stores flags:

```text
Zero Flag
Carry Flag
Overflow Flag
```

---

## Memory Management Information

Page table references.

---

## Process State

```text
Ready
Blocked
Running
```

---

# 8. Types of Context Switching

## 1. Process Context Switching

Switching between processes.

Example:

```text
Chrome → VS Code
```

---

## 2. Thread Context Switching

Switching between threads.

Example:

```text
Download Thread
→
UI Thread
```

---

# 9. Process Context Switching

Process switching requires:

```text
Save Process A State
Load Process B State
```

Includes:

* Memory mappings
* Registers
* PCB data

---

## Cost

High overhead.

Reason:

Different memory spaces.

---

# 10. Thread Context Switching

Switching between threads of same process.

Example:

```text
Thread A
↓
Thread B
```

Shared:

* Memory
* Heap
* Files

---

## Cost

Lower than process switching.

Reason:

No memory-space change required.

---

# 11. CPU Scheduling and Context Switching

Scheduling determines when context switching happens.

---

## FCFS

```text
P1 → P2 → P3
```

Few switches.

---

## Round Robin

```text
P1 → P2 → P3 → P1
```

Many switches.

---

## Priority Scheduling

Higher priority may cause immediate switch.

---

# 12. Context Switching Overhead

## Definition

Time spent saving and restoring context.

---

## Important

During switching:

```text
CPU Does No Useful Work
```

This is overhead.

---

## Example

```text
Execute Task = 10ms
Switch Cost = 1ms
```

Efficiency:

```text
10 / 11 = 90.9%
```

---

# 13. Advantages

## Multitasking

Run many applications.

---

## Better Responsiveness

Applications remain interactive.

---

## Efficient CPU Sharing

Multiple tasks share CPU.

---

## Fair Scheduling

All processes receive CPU time.

---

## Parallel User Experience

Users perceive simultaneous execution.

---

# 14. Disadvantages

## Performance Overhead

Saving/loading context consumes time.

---

## Increased CPU Usage

Frequent switching wastes CPU cycles.

---

## Cache Pollution

CPU cache may be invalidated.

---

## Reduced Throughput

Too many switches reduce efficiency.

---

## Complexity

OS scheduler becomes more complex.

---

# 15. Real-Time Examples

## Example 1: Web Browser

Chrome:

```text
Tab 1
Tab 2
Download
Rendering
```

CPU switches among them continuously.

---

## Example 2: Video Streaming

YouTube:

```text
Buffering
Audio
Video
UI
```

Threads switch rapidly.

---

## Example 3: Gaming

```text
Graphics
Audio
Physics
Networking
```

Many thread switches occur every second.

---

## Example 4: Mobile Apps

Android:

```text
WhatsApp
Instagram
Chrome
Background Services
```

Frequent process switching.

---

## Example 5: Database Server

```text
Query Thread
Logging Thread
Cache Thread
Backup Thread
```

Thousands of context switches per second.

---

# 16. Major Project Usage

## Banking System

Context switching occurs between:

```text
Transaction Service
Fraud Detection
Notification Service
```

---

## E-Commerce Platform

```text
Search Service
Payment Service
Inventory Service
```

---

## Cloud Computing

```text
Virtual Machines
Containers
Monitoring Services
```

Frequent switching.

---

## Social Media Platform

```text
Feed Processing
Messaging
Media Upload
Notifications
```

---

## Web Servers

Thousands of requests handled through thread switching.

---

# 17. Process vs Thread Context Switching

| Feature          | Process Switch | Thread Switch |
| ---------------- | -------------- | ------------- |
| Speed            | Slower         | Faster        |
| Memory Change    | Yes            | Usually No    |
| Overhead         | High           | Low           |
| Cache Impact     | High           | Lower         |
| Resource Sharing | Separate       | Shared        |

---

# 18. Performance Impact

## Too Many Context Switches Cause

### CPU Overhead

More time switching than executing.

---

### Cache Misses

CPU cache becomes ineffective.

---

### Reduced Throughput

Applications slow down.

---

## Monitoring Tools

Linux:

```bash
vmstat
top
htop
pidstat
```

---

Windows:

```text
Task Manager
Performance Monitor
```

---

# 19. Best Practices

## Minimize Unnecessary Threads

Avoid creating excessive threads.

---

## Use Thread Pools

Reuse existing threads.

---

## Reduce Blocking Operations

Use asynchronous programming.

---

## Optimize Scheduling

Choose suitable scheduling policies.

---

## Monitor Context Switch Rate

Track performance metrics regularly.

---

# 20. Interview Questions

### Q1. What Is Context Switching?

Saving one process/thread state and loading another.

---

### Q2. Why Is Context Switching Needed?

To support multitasking.

---

### Q3. What Is Saved During Context Switching?

* Registers
* Program Counter
* Stack Pointer
* Process State

---

### Q4. Why Is Thread Switching Faster?

Threads share memory.

---

### Q5. What Is Context Switching Overhead?

CPU time spent switching instead of executing tasks.

---

### Q6. Which Is Costlier?

Process context switching.

---

### Q7. Can Excessive Context Switching Hurt Performance?

Yes, significantly.

---

# 21. Conclusion

Context Switching is a core operating system mechanism that enables multitasking by allowing the CPU to alternate between processes and threads. During a context switch, the operating system saves the current execution state and restores another one, making it appear that many tasks run simultaneously.

It is essential in:

* Operating Systems
* Web Servers
* Cloud Platforms
* Databases
* Mobile Applications
* Gaming Engines
* Distributed Systems

Understanding context switching is crucial for OS concepts, performance tuning, multithreading, concurrency, and system design interviews.

