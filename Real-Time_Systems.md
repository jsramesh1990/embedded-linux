# Real-Time Systems (RTS)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Real-Time System?](#what-is-a-real-time-system)
3. [Why Do We Need Real-Time Systems?](#why-do-we-need-real-time-systems)
4. [Key Characteristics](#key-characteristics)
5. [How a Real-Time System Works](#how-a-real-time-system-works)
6. [Real-Time Constraints](#real-time-constraints)
7. [Types of Real-Time Systems](#types-of-real-time-systems)

   * [Hard Real-Time Systems](#hard-real-time-systems)
   * [Soft Real-Time Systems](#soft-real-time-systems)
   * [Firm Real-Time Systems](#firm-real-time-systems)
8. [Components of a Real-Time System](#components-of-a-real-time-system)
9. [Real-Time Operating System (RTOS)](#real-time-operating-system-rtos)
10. [Scheduling Algorithms](#scheduling-algorithms)
11. [Interrupt Handling](#interrupt-handling)
12. [Latency and Jitter](#latency-and-jitter)
13. [Real-Time Systems in Automotive](#real-time-systems-in-automotive)
14. [Real-Time Systems in Embedded Linux](#real-time-systems-in-embedded-linux)
15. [Advantages](#advantages)
16. [Disadvantages](#disadvantages)
17. [Comparison Table](#comparison-table)
18. [Interview Questions](#interview-questions)
19. [Conclusion](#conclusion)

---

# Introduction

In many embedded applications, producing the correct output is not enough.

The output must be produced within a specific time limit.

Example:

```text
Airbag System
```

If the airbag opens:

```text
After 5 ms  → Safe
After 5 sec → Useless
```

Therefore:

```text
Correct Output
      +
Correct Time
```

is the foundation of a Real-Time System.

---

# What is a Real-Time System?

### Definition

> A Real-Time System (RTS) is a system in which the correctness of an operation depends not only on the logical result but also on the time at which the result is produced.

---

## Simple Understanding

Normal System:

```text
Correct Result
```

Real-Time System:

```text
Correct Result
      +
Correct Timing
```

---

## Interview Answer

### What is a Real-Time System?

A real-time system is a system that must respond to events within a specified deadline. The correctness of the system depends on both the output and the response time.

---

# Why Do We Need Real-Time Systems?

Many applications are time-critical.

Examples:

| Application      | Deadline                     |
| ---------------- | ---------------------------- |
| Airbag ECU       | Few milliseconds             |
| ABS Braking      | Few milliseconds             |
| Pacemaker        | Microseconds to milliseconds |
| Aircraft Control | Strict deadline              |
| Industrial Robot | Real-time response           |

Missing deadlines may cause:

* Equipment damage
* Financial loss
* Safety hazards
* Loss of life

---

# Key Characteristics

## Deterministic Behavior

The response time should be predictable.

---

## Fast Response

Events must be handled quickly.

---

## Reliability

System should operate continuously.

---

## Low Latency

Minimal delay between event and response.

---

## Deadline-Oriented

Meeting deadlines is the primary goal.

---

# How a Real-Time System Works

Example:

Temperature Protection System

```text
Temperature Sensor
         ↓
Interrupt Generated
         ↓
CPU Receives Event
         ↓
Control Algorithm
         ↓
Cooling Fan ON
```

---

## Internal Flow

```text
Event Occurs
      ↓
Interrupt Triggered
      ↓
ISR Executes
      ↓
Task Activated
      ↓
Action Performed
```

---

# Real-Time Constraints

A real-time system must satisfy:

```text
Task Completion Time
          ≤
Required Deadline
```

---

## Example

Deadline:

```text
10 ms
```

Task Completion:

```text
7 ms
```

Result:

```text
SUCCESS
```

---

If:

```text
Task Completion = 15 ms
```

Result:

```text
DEADLINE MISSED
```

---

# Types of Real-Time Systems

```text
Real-Time Systems
│
├── Hard Real-Time
├── Firm Real-Time
└── Soft Real-Time
```

---

# Hard Real-Time Systems

## Definition

A hard real-time system must always meet its deadlines.

Missing a deadline is considered a system failure.

---

## Example

### Airbag ECU

```text
Crash Detected
       ↓
Deploy Airbag
       ↓
Within Milliseconds
```

---

### Pacemaker

```text
Heartbeat Event
        ↓
Electrical Pulse
        ↓
Immediate Response
```

---

## Applications

* Airbag Systems
* Aircraft Control
* Pacemakers
* Missile Guidance
* Nuclear Control Systems

---

## Interview Answer

Hard real-time systems cannot tolerate missed deadlines because failure may result in catastrophic consequences.

---

# Soft Real-Time Systems

## Definition

Occasional deadline misses are acceptable.

System quality may degrade but continues operating.

---

## Example

### Video Streaming

```text
Frame Delayed
      ↓
Video Continues
```

---

### Online Gaming

```text
Packet Delayed
      ↓
Minor Lag
```

---

## Applications

* Multimedia Systems
* Streaming Platforms
* Video Conferencing
* Gaming Systems

---

# Firm Real-Time Systems

## Definition

A result produced after the deadline has no value, but occasional misses do not cause complete failure.

---

## Example

### Radar Tracking

```text
Old Target Position
        ↓
No Longer Useful
```

---

## Applications

* Radar Systems
* Stock Trading
* Sensor Fusion Systems

---

# Components of a Real-Time System

```text
Sensors
   ↓
Interrupt Controller
   ↓
CPU
   ↓
RTOS
   ↓
Tasks
   ↓
Actuators
```

---

## Main Components

### Sensors

Generate events.

Examples:

* Temperature Sensor
* Pressure Sensor
* Wheel Speed Sensor

---

### Processor

Executes control algorithms.

Examples:

* ARM Cortex-M
* DSP
* RISC-V

---

### RTOS

Schedules tasks predictably.

---

### Actuators

Perform actions.

Examples:

* Motors
* Relays
* Valves

---

# Real-Time Operating System (RTOS)

## What is RTOS?

> An RTOS is an operating system designed to provide deterministic and predictable task scheduling.

---

## Examples

* FreeRTOS
* Zephyr
* VxWorks
* QNX
* RTEMS

---

## Why RTOS?

Provides:

* Task Scheduling
* Priority Management
* Timers
* Synchronization
* Interrupt Handling

---

# Scheduling Algorithms

Scheduling decides:

```text
Which Task Runs Next
```

---

## Priority-Based Scheduling

```text
High Priority Task
       ↓
Runs First
```

Common in RTOS.

---

## Round Robin

```text
Task1
Task2
Task3
```

Each gets CPU time.

---

## Earliest Deadline First (EDF)

```text
Closest Deadline
      ↓
Highest Priority
```

---

# Interrupt Handling

Real-time systems depend heavily on interrupts.

---

## Example

Button Press

```text
Button Pressed
       ↓
Interrupt Generated
       ↓
ISR Executed
       ↓
Task Activated
```

---

## ISR (Interrupt Service Routine)

```c
void ISR_Button(void)
{
    button_pressed = 1;
}
```

---

# Latency and Jitter

## Latency

### Definition

Time between:

```text
Event
   ↓
Response
```

---

### Example

```text
Interrupt Occurs
       ↓
CPU Responds After 5 μs
```

Latency:

```text
5 μs
```

---

## Jitter

### Definition

Variation in response time.

---

Example:

```text
Response 1 = 5 μs
Response 2 = 7 μs
Response 3 = 6 μs
```

Variation:

```text
Jitter
```

---

## Goal

```text
Low Latency
Low Jitter
```

---

# Real-Time Systems in Automotive

Modern vehicles contain many real-time systems.

---

## Examples

### Hard Real-Time

* Airbag ECU
* ABS ECU
* Electronic Stability Control

---

### Firm Real-Time

* Radar Tracking
* ADAS Processing

---

### Soft Real-Time

* Infotainment
* Navigation

---

## Automotive Workflow

```text
Wheel Speed Sensor
         ↓
ABS ECU
         ↓
Brake Control
         ↓
Vehicle Stability
```

---

# Real-Time Systems in Embedded Linux

Standard Linux:

```text
Not Fully Deterministic
```

---

For real-time applications:

### PREEMPT_RT

Provides:

* Lower Latency
* Better Scheduling
* Improved Determinism

---

## Applications

* Industrial Automation
* Robotics
* Automotive Gateways
* Medical Systems

---

# Advantages

## Deterministic Operation

Predictable behavior.

---

## Fast Response

Quick event handling.

---

## Improved Safety

Essential for critical applications.

---

## Reliability

Suitable for continuous operation.

---

## Efficient Resource Usage

Optimized scheduling.

---

# Disadvantages

## Increased Complexity

Requires careful design.

---

## Higher Development Cost

Verification and testing are extensive.

---

## Resource Constraints

Real-time guarantees may limit flexibility.

---

## Debugging Difficulty

Timing-related bugs can be difficult to reproduce.

---

# Comparison Table

| Feature               | Hard RT      | Firm RT        | Soft RT          |
| --------------------- | ------------ | -------------- | ---------------- |
| Deadline Miss Allowed | No           | Rarely         | Yes              |
| Failure Severity      | Catastrophic | Reduced Value  | Performance Loss |
| Example               | Airbag ECU   | Radar Tracking | Video Streaming  |
| Determinism           | Very High    | High           | Moderate         |

---

# Interview Questions

### What is a Real-Time System?

A system where correctness depends on both the result and the time at which the result is produced.

---

### What is a Deadline?

The maximum allowed time for a task to complete.

---

### Difference Between Hard and Soft Real-Time Systems?

| Hard RT              | Soft RT                 |
| -------------------- | ----------------------- |
| Deadline Must Be Met | Deadline Preferred      |
| Failure if Missed    | Performance Degradation |
| Airbag ECU           | Video Streaming         |

---

### What is RTOS?

A Real-Time Operating System designed to provide deterministic task scheduling and predictable response times.

---

### What is Latency?

The delay between an event occurring and the system responding.

---

### What is Jitter?

The variation in response time between similar events.

---

### Why Are Interrupts Important?

Interrupts allow the CPU to respond immediately to external events without continuously polling hardware.

---

### Why is Standard Linux Not Considered Hard Real-Time?

Because scheduling delays and kernel operations can introduce unpredictable latency.

---

# What Happens Internally?

Example:

```text
Crash Sensor Detects Impact
```

---

### Step 1

```text
Sensor Generates Interrupt
```

---

### Step 2

```text
CPU Suspends Current Task
```

---

### Step 3

```text
ISR Executes
```

---

### Step 4

```text
Airbag Task Activated
```

---

### Step 5

```text
Airbag Deployed
```

---

### Step 6

```text
Deadline Met
```

---

# Most Asked Interview Question

### Explain Real-Time Systems with Example.

A real-time system is a system that must produce correct results within a specified time limit. The correctness of the system depends on both the logical output and the timing of the response. Real-time systems are classified as Hard Real-Time, Firm Real-Time, and Soft Real-Time. Examples include airbag controllers, ABS braking systems, industrial robots, pacemakers, and multimedia applications. RTOS, interrupts, low latency, and deterministic scheduling are key concepts in real-time systems.

---

# Quick Revision (30-Second Interview Answer)

```text
Real-Time System
        =
Correct Output
        +
Correct Timing
```

### Types

```text
Hard RT  → Never Miss Deadline
Firm RT  → Late Result Has No Value
Soft RT  → Late Result Still Useful
```

### Examples

```text
Hard RT  → Airbag ECU
Firm RT  → Radar Tracking
Soft RT  → Video Streaming
```

---

# Conclusion

Real-time systems are the foundation of modern embedded applications where timing is critical. They ensure that tasks are completed within defined deadlines, making them essential in automotive, aerospace, industrial automation, medical devices, robotics, and telecommunications. Understanding deadlines, latency, jitter, RTOS scheduling, and interrupt handling is crucial for every Embedded Systems and Embedded Linux engineer.
