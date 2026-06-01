# dangling-pointer

---

#  Introduction

In C and C++, a **dangling pointer** is a pointer that still points to a memory location that has already been freed or is no longer valid.

In embedded systems, dangling pointers are dangerous because they can cause:

* system crashes
* unpredictable behavior
* memory corruption
* hardware faults
* real-time system failure

---

#  Definition

A dangling pointer is a pointer that:

* points to deallocated memory
* or points to a variable that is out of scope
* or points to invalid hardware/software memory

---

#  Syntax Example (Simple Case)

```c id="d1"
int *ptr = (int*)malloc(sizeof(int));
free(ptr);

// ptr is now a dangling pointer
```

---

#  Working Flow of Dangling Pointer

```text id="flow1"
Memory allocated (malloc)
        |
        v
Pointer points to valid memory
        |
        v
Memory freed (free)
        |
        v
Pointer still holds old address
        |
        v
Invalid memory access (DANGLING POINTER)
```

---

#  Dangling Pointer Diagram

```text id="diag1"
Before free():
+-------------------+
| Heap Memory       |
| value = 100       |
+-------------------+
        ^
        |
       ptr

After free():
+-------------------+
| Heap Memory freed |
+-------------------+
        ^
        |
       ptr  (DANGLING)
```

---

#  Types of Dangling Pointers

---

# 1. Heap Dangling Pointer

## Definition

Occurs when dynamically allocated memory is freed.

## Example

```c id="h1"
#include <stdlib.h>

int main() {
    int *ptr = (int*)malloc(sizeof(int));
    *ptr = 10;

    free(ptr);

    *ptr = 20; // ❌ dangling pointer
}
```

---

# 2. Stack Dangling Pointer

## Definition

Occurs when pointer points to local variable after function return.

## Example

```c id="s1"
int* func() {
    int x = 10;
    return &x; //  dangerous
}
```

---

## Flow

```text id="flow2"
Function starts
   |
   v
Local variable created (stack)
   |
   v
Function ends → stack destroyed
   |
   v
Pointer still refers to invalid memory
```

---

# 3. Double Free Dangling Pointer

## Example

```c id="d2"
int *ptr = (int*)malloc(sizeof(int));

free(ptr);
free(ptr); //  undefined behavior
```

---

#  Embedded System Example

```c id="e1"
int* buffer;

void init() {
    buffer = (int*)malloc(100);
}

void deinit() {
    free(buffer);
}

void useBuffer() {
    buffer[0] = 10; //  if deinit called earlier → dangling pointer
}
```

---

#  Real-Time Embedded Flow Issue

```text id="flow3"
Peripheral allocates memory
        |
        v
Data processing uses pointer
        |
        v
Memory freed unexpectedly
        |
        v
ISR or task still uses pointer
        |
        v
SYSTEM CRASH / UNDEFINED BEHAVIOR
```

---

#  Why Dangling Pointers are Dangerous in Embedded Systems

* No operating system safety (bare-metal systems)
* Direct hardware interaction
* Real-time constraints
* No memory protection (in many MCUs)
* Hard to debug runtime faults

---

#  Real-Time Example (UART Buffer)

```c id="ex1"
char *uartBuffer;

void setup() {
    uartBuffer = (char*)malloc(128);
}

void process() {
    free(uartBuffer);

    uartBuffer[0] = 'A'; //  dangerous access
}
```

---

# ⚖ Difference: Valid Pointer vs Dangling Pointer

| Feature       | Valid Pointer | Dangling Pointer |
| ------------- | ------------- | ---------------- |
| Memory status | Active        | Freed            |
| Safety        | Safe          | Unsafe           |
| Behavior      | Defined       | Undefined        |
| Embedded use  | OK            | Dangerous        |

---

#  How to Avoid Dangling Pointers

---

## 1. Set pointer to NULL after free

```c id="safe1"
free(ptr);
ptr = NULL;
```

---

## 2. Check before use

```c id="safe2"
if (ptr != NULL) {
    *ptr = 10;
}
```

---

## 3. Avoid returning address of local variable

```c id="safe3"
int* func() {
    static int x = 10;
    return &x; // safe
}
```

---

## 4. Use static memory in embedded systems

```c id="safe4"
static int buffer[100];
```

---

# ⚙ Safe Flow Design

```text id="safe_flow"
Allocate memory
      |
      v
Use pointer safely
      |
      v
Free memory
      |
      v
Set pointer = NULL
      |
      v
Check before reuse
```

---

# ⚠ Common Mistakes

---

## 1. Using pointer after free

```c id="m1"
free(ptr);
*ptr = 10; // 
```

---

## 2. Returning local variable address

```c id="m2"
return &localVar; // 
```

---

## 3. Not resetting pointer

```c id="m3"
free(ptr);
// ptr still holds old address 
```

---

---

#  Final Summary

A dangling pointer is a pointer that points to invalid memory after it has been freed or gone out of scope.

In embedded systems, it is extremely dangerous because it can lead to:

* system crashes
* unpredictable behavior
* real-time failures

---


