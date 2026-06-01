# `enum` in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is enum?](#what-is-enum)
3. [Why do we use enum?](#why-do-we-use-enum)
4. [How enum works](#how-enum-works)
5. [Syntax](#syntax)
6. [Default values in enum](#default-values-in-enum)
7. [Examples](#examples)
8. [enum vs macros (#define)](#enum-vs-macros-define)
9. [enum with typedef](#enum-with-typedef)
10. [Where enum is used in Embedded Systems](#where-enum-is-used-in-embedded-systems)
11. [Advantages and Disadvantages](#advantages-and-disadvantages)
12. [Interview Questions](#interview-questions)
13. [Conclusion](#conclusion)

---

# Introduction

In embedded systems, we often deal with **states, modes, and error codes**.

Instead of using random numbers like `0, 1, 2`, we use **enum** to make code readable and meaningful.

---

# What is enum?

### Definition

> `enum` (enumeration) is a user-defined data type in C used to assign names to a set of integer constants.

---

## Simple Meaning

Instead of writing numbers, we use **meaningful names**.

---

## Example

```c id="e1"
enum Status { OFF, ON };
```

Here:

```text id="e2"
OFF = 0
ON  = 1
```

---

# Why do we use enum?

Without enum:

```c id="e3"
int state = 1;
```

Hard to understand meaning.

---

With enum:

```c id="e4"
enum State { IDLE, RUNNING, STOPPED };

enum State s = RUNNING;
```

Much clearer.

---

## Benefits

✔ Improves readability
✔ Reduces magic numbers
✔ Easier debugging
✔ Better code maintainability
✔ Used in firmware state machines

---

# How enum works

`enum` assigns **integer values automatically** starting from 0.

---

## Example

```c id="e5"
enum Color { RED, GREEN, BLUE };
```

Values:

```text id="e6"
RED   = 0
GREEN = 1
BLUE  = 2
```

---

# Syntax

```c id="e7"
enum enum_name {
    constant1,
    constant2,
    constant3
};
```

---

# Default values in enum

You can also assign custom values.

---

## Example

```c id="e8"
enum ErrorCode {
    OK = 0,
    ERROR = -1,
    TIMEOUT = 100
};
```

---

## Output mapping

```text id="e9"
OK      = 0
ERROR   = -1
TIMEOUT = 100
```

---

# Examples

## 1. Simple State Machine

```c id="e10"
enum State {
    IDLE,
    START,
    RUN,
    STOP
};

enum State current = IDLE;
```

---

## 2. Embedded System Example

```c id="e11"
enum MotorState {
    MOTOR_OFF,
    MOTOR_ON,
    MOTOR_FAULT
};
```

---

# enum vs macros (#define)

| Feature     | enum          | #define          |
| ----------- | ------------- | ---------------- |
| Type Safety | Yes           | No               |
| Debugging   | Easy          | Hard             |
| Memory      | Integer group | Text replacement |
| Readability | High          | Medium           |

---

## Example

### enum (better)

```c id="e12"
enum Mode { AUTO, MANUAL };
```

### macro (not preferred)

```c id="e13"
#define AUTO 0
#define MANUAL 1
```

---

# enum with typedef

Very common in embedded systems.

---

## Example

```c id="e14"
typedef enum {
    LOW,
    MEDIUM,
    HIGH
} Priority;
```

Now:

```c id="e15"
Priority p = HIGH;
```

---

# Where enum is used in Embedded Systems

## 1. State Machines

```c id="e16"
enum State { INIT, RUN, ERROR };
```

Used in:

* ECU control
* Motor control
* Communication systems

---

## 2. Error Handling

```c id="e17"
enum ErrorCode { OK, FAIL, TIMEOUT };
```

---

## 3. Device Modes

```c id="e18"
enum Mode { SLEEP, ACTIVE, DEEP_SLEEP };
```

---

## 4. Communication Protocols

* CAN states
* UART states
* SPI states

---

## 5. RTOS Tasks

* Task states (READY, RUNNING, BLOCKED)

---

# Advantages

✔ Improves code readability
✔ Reduces magic numbers
✔ Easier debugging
✔ Helps in state machines
✔ Improves maintainability

---

# Disadvantages

✘ Fixed integer mapping
✘ Not memory efficient in some cases
✘ Debugging raw values may still be needed
✘ Limited flexibility compared to structs

---

# Interview Questions

## What is enum?

`enum` is a user-defined data type used to assign names to integer constants.

---

## Why do we use enum?

To improve readability and replace magic numbers with meaningful names.

---

## What is default value of enum?

Starts from 0 and increments by 1 unless explicitly assigned.

---

## Difference between enum and #define?

| enum             | #define       |
| ---------------- | ------------- |
| Type-safe        | Not type-safe |
| Debug-friendly   | Hard to debug |
| Compiler feature | Preprocessor  |

---

## Can we change enum values?

Yes, we can assign custom integer values.

---

## Why enum is used in embedded systems?

For defining states, modes, and error codes in a readable and maintainable way.

---

# Most Asked Interview Question

## Explain enum with example.

`enum` is a user-defined data type in C used to assign meaningful names to integer constants. It improves readability and reduces the use of magic numbers. For example, `enum State { IDLE, RUNNING, STOPPED };` defines system states where IDLE = 0, RUNNING = 1, and STOPPED = 2. It is widely used in embedded systems for state machines, error handling, and device modes.

---

# Quick Revision

```text id="e19"
enum = Named integer constants

Example:
enum State { IDLE, RUN, STOP };

Used in:
- State Machines
- Error Codes
- Embedded Firmware
- RTOS
```

---

# Conclusion

`enum` is a powerful feature in C that improves readability and structure in embedded systems. It is widely used in firmware development for state machines, error handling, and device control logic. Understanding enums is essential for writing clean, professional embedded software and is frequently asked in interviews.
