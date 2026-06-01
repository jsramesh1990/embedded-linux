# double-pointer

---

#  Introduction

In C and C++, a **double pointer** (pointer to pointer) is a pointer that stores the address of another pointer.

In embedded systems, double pointers are widely used in:

* dynamic memory allocation (2D buffers)
* driver design (UART, SPI buffers)
* array of pointers handling
* passing pointer addresses to functions
* complex data structures

---

#  Definition

A double pointer is a pointer that stores the address of another pointer.

---

#  Syntax

```c id="d1"
data_type **pointer_name;
```

---

#  Basic Concept

```text id="flow1"
Variable → Pointer → Double Pointer
```

or

```text
Value → Address → Address of Pointer
```

---

#  Memory Diagram

```text id="diag1"
Value:        10
               |
               v
Pointer p → Address of value
               |
               v
Double pointer pp → Address of p
```

---

#  Working Flow of Double Pointer

```text id="flow2"
Variable created
      |
      v
Pointer stores address of variable
      |
      v
Double pointer stores address of pointer
      |
      v
Access value using **pp
```

---

#  Example 1: Basic Double Pointer

```c id="ex1"
#include <stdio.h>

int main() {
    int a = 10;
    int *p = &a;
    int **pp = &p;

    printf("%d\n", a);
    printf("%d\n", *p);
    printf("%d\n", **pp);

    return 0;
}
```

---

#  Embedded System Use Case

---

## 1. Dynamic 2D Buffer

```c id="ex2"
int **buffer;
```

Used for:

* image processing
* sensor grids
* communication buffers

---

## Allocation Example

```c id="ex3"
buffer = (int**)malloc(3 * sizeof(int*));

for(int i = 0; i < 3; i++) {
    buffer[i] = (int*)malloc(5 * sizeof(int));
}
```

---

#  Flow of 2D Allocation

```text id="flow3"
Step 1: Allocate row pointers
        |
        v
Step 2: Allocate columns for each row
        |
        v
Step 3: Access using buffer[i][j]
```

---

#  2D Memory Representation

```text id="diag2"
buffer
  |
  v
+-----+-----+-----+
| *   | *   | *   |  → rows
+-----+-----+-----+
  |     |     |
  v     v     v
 [cols][cols][cols]
```

---

#  Real-Time Embedded Example

---

## UART Buffer Handling

```c id="ex4"
void receiveData(char **data) {
    *data = (char*)malloc(100);
}
```

---

## Usage

```c id="ex5"
char *buffer;

receiveData(&buffer);
```

---

#  Flow

```text id="flow4"
Main function
      |
      v
Pass address of pointer
      |
      v
Function modifies original pointer
      |
      v
Memory allocated dynamically
```

---

#  Why Double Pointer is Needed

* Modify pointer inside function
* Allocate dynamic 2D arrays
* Handle multiple buffers
* Driver-level programming
* Return multiple memory blocks

---

#  Example: Modify Pointer via Function

```c id="ex6"
void allocate(int **p) {
    *p = (int*)malloc(sizeof(int));
}

int main() {
    int *ptr = NULL;

    allocate(&ptr);

    *ptr = 50;

    return 0;
}
```

---

#  Common Mistakes

---

## 1. Not allocating memory properly

```c id="m1"
int **p;
*p = 10; //  crash
```

---

## 2. Forgetting second-level allocation

```c id="m2"
int **p = malloc(10); //  incomplete
```

---

## 3. Memory leak in 2D arrays

```c id="m3"
for(int i=0;i<3;i++)
    free(p[i]);

free(p);
```

---

# 🔌 Embedded System Flow

```text id="flow5"
Hardware/Buffer requirement
        |
        v
Allocate pointer array
        |
        v
Allocate sub-arrays
        |
        v
Process sensor/communication data
```

---

#  Advantages

| Advantage              | Explanation                  |
| ---------------------- | ---------------------------- |
| Dynamic structure      | Flexible memory allocation   |
| Function flexibility   | Modify pointers in functions |
| Efficient buffering    | Useful for drivers           |
| Multi-dimensional data | Image/sensor grids           |

---

#  Disadvantages

| Issue             | Explanation           |
| ----------------- | --------------------- |
| Complex usage     | Hard to debug         |
| Memory leaks      | Easy to forget free   |
| Overhead          | Multiple allocations  |
| Pointer confusion | Difficult readability |

---

