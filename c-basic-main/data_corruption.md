# data_corruption

---

#  Introduction

In embedded systems, **data corruption** happens when stored or transmitted data becomes incorrect, altered, or inconsistent due to hardware, software, or memory issues.

Since embedded systems often run without memory protection (no MMU), data corruption can directly lead to:

* wrong sensor readings
* system crashes
* unsafe actuator behavior
* communication failures

---

#  Definition

Data corruption is a condition where **valid data is unintentionally modified into incorrect or invalid data during storage, processing, or transmission**.

---

#  Why Data Corruption Happens

* race conditions
* buffer overflow
* pointer misuse
* DMA misconfiguration
* power failure during write
* interrupt conflicts
* memory corruption (heap/stack issues)

---

#  Data Corruption Flow

```text id="flow1"
Valid data stored
        |
        v
Unexpected write / overwrite
        |
        v
Data modified incorrectly
        |
        v
System reads wrong value
        |
        v
Incorrect system behavior
```

---

#  Memory-Based Corruption Diagram

```text id="diag1"
Before:
[ 10 ][ 20 ][ 30 ][ 40 ]

After corruption:
[ 10 ][ XX ][ 99 ][ 40 ]  
```

---

#  Embedded System Example

## Buffer Overflow Causing Corruption

```c id="ex1"
int buffer[5];

buffer[0] = 1;
buffer[1] = 2;

buffer[10] = 99; //  data corruption
```

---

#  Types of Data Corruption

---

## 1. Memory Corruption

* stack overflow
* heap overflow
* wild pointer access

---

## 2. Communication Corruption

* UART noise
* SPI mismatch
* CAN frame errors

---

## 3. Concurrency Corruption

* race conditions
* missing mutex protection

---

## 4. Hardware-Induced Corruption

* voltage drop
* EMI noise
* flash write failure

---

#  Real Embedded Example

## UART Buffer Corruption

```c id="ex2"
char rxBuffer[10];

void ISR_UART() {
    rxBuffer[20] = 'A'; //  overflow corruption
}
```

---

# ⚙ Data Corruption Flow in RTOS

```text id="flow2"
Task A writes data
        |
Task B interrupts and modifies same memory
        |
Data becomes inconsistent
        |
System reads wrong value
```

---

#  Race Condition → Data Corruption

```text id="flow3"
Task 1: counter = 5
Task 2: counter = 5

Task 1 writes 6
Task 2 writes 6  lost update
```

---

#  Impact in Embedded Systems

* incorrect motor control
* wrong sensor calibration
* communication failure
* safety hazards
* watchdog resets
* unpredictable system behavior

---

#  Why Embedded Systems Are More Vulnerable

* no memory protection (no MMU)
* direct hardware access
* interrupts can occur anytime
* real-time constraints
* limited debugging visibility

---

#  Data Corruption Lifecycle

```text id="flow4"
Correct data
      |
      v
Unexpected modification
      |
      v
Invalid state propagation
      |
      v
System-level failure
```

---

#  How to Prevent Data Corruption

---

## 1. Use proper synchronization

```c id="sol1"
mutex / semaphore protection
```

---

## 2. Prevent buffer overflow

```c id="sol2"
if(index < size)
```

---

## 3. Use volatile correctly

```c id="sol3"
volatile prevents compiler optimization issues
```

---

## 4. Validate inputs

```c id="sol4"
check sensor data ranges
```

---

## 5. Use memory pools

```text id="sol5"
reduces heap corruption risk
```

---

## 6. Disable interrupts in critical sections

```c id="sol6"
__disable_irq();
__enable_irq();
```

---

#  Common Mistakes

---

## 1. Ignoring bounds checking

```c id="m1"
buffer[100] = 10; //  overflow
```

---

## 2. Using shared data without protection

```text id="m2"
leads to race condition corruption
```

---

## 3. Misusing pointers

```c id="m3"
dereferencing invalid pointer
```

---

## 4. Not handling communication errors

---

#  Data Corruption vs Memory Corruption

| Feature    | Data Corruption | Memory Corruption |
| ---------- | --------------- | ----------------- |
| Scope      | Data values     | Memory regions    |
| Cause      | logic/hardware  | pointer/overflow  |
| Visibility | subtle          | severe            |
| Impact     | wrong output    | system crash      |

---

#  Real-Time Embedded Impact

* corrupted sensor readings
* incorrect control signals
* system instability
* safety-critical failures
* communication breakdown

---

#  Example: Sensor Data Corruption

```c id="ex3"
int temperature = 25;

void ISR() {
    temperature = -999; //  corrupted value
}
```


