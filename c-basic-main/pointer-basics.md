# Pointers

---

#  Introduction

In C and C++, **pointers** are variables that store the memory address of another variable.

In embedded systems, pointers are extremely important because they are used for:

* hardware register access
* memory-mapped I/O
* efficient array handling
* driver development
* interrupt handling
* RTOS communication
* direct memory control

---

#  Definition

A pointer is a variable that stores the **address of another variable** instead of storing a direct value.

---

#  Syntax of Pointer

## Declaration

```c id="p1"
data_type *pointer_name;
```

## Initialization

```c id="p2"
pointer_name = &variable;
```

## Example

```c id="p3"
int a = 10;
int *p = &a;
```

---

#  Basic Pointer Concepts

---

# 1. Address Operator (&)

Used to get memory address of a variable.

```c id="p4"
int a = 10;
printf("%p", &a);
```

---

# 2. Dereference Operator (*)

Used to access value stored at address.

```c id="p5"
int a = 10;
int *p = &a;

printf("%d", *p);
```

---

#  Pointer Working Flow

```text id="flow1"
Variable (a = 10)
      |
      v
Memory Address (0x1000)
      |
      v
Pointer stores address (p = 0x1000)
      |
      v
Dereference (*p) → value = 10
```

---

#  Pointer Diagram

```text id="diag1"
   Variable Memory
   ----------------
   | a = 10      |
   | Address=1000|
   ----------------
         |
         |  &
         v
   Pointer p = 1000
         |
         |  *
         v
   Value = 10
```

---

#  Types of Pointers

---

# 1. NULL Pointer

## Definition

Pointer that points to nothing.

## Syntax

```c id="n1"
int *p = NULL;
```

---

## Use

* Safety initialization
* Avoid garbage memory access

---

# 2. Void Pointer

## Definition

Generic pointer that can point to any data type.

## Syntax

```c id="v1"
void *p;
```

## Example

```c id="v2"
int a = 10;
void *p = &a;
```

---

# 3. Wild Pointer

## Definition

Uninitialized pointer (dangerous).

```c id="w1"
int *p;  // wild pointer
```

---

# 4. Dangling Pointer

## Definition

Pointer pointing to freed memory.

```c id="d1"
int *p = malloc(sizeof(int));
free(p);
// p is now dangling
```

---

#  Embedded Pointer Example (GPIO Register Access)

```c id="e1"
#define GPIO (*(volatile unsigned int*)0x40020000)

int main() {
    GPIO = 1;        // Write to hardware register
    GPIO |= (1 << 2);
    GPIO &= ~(1 << 2);

    return 0;
}
```

---

#  Memory-Mapped I/O Flow

```text id="flow2"
CPU Instruction
      |
      v
Pointer Address (0x40020000)
      |
      v
Hardware Register (GPIO)
      |
      v
LED / Motor / Peripheral Action
```

---

#  Pointer Arithmetic

## Definition

Pointers can be incremented or decremented based on data type size.

---

## Example

```c id="pa1"
int arr[3] = {10, 20, 30};
int *p = arr;

printf("%d\n", *p);
p++;
printf("%d\n", *p);
```

---

## Flow

```text id="pa2"
p → arr[0]
p++ → arr[1]
p++ → arr[2]
```

---

#  Pointer with Arrays

```c id="arr1"
int arr[3] = {1, 2, 3};
int *p = arr;

for(int i = 0; i < 3; i++) {
    printf("%d\n", *(p + i));
}
```

---

#  Pointer with Functions (Call by Reference)

## Definition

Pointers allow functions to modify original variables.

---

## Example

```c id="fn1"
void update(int *p) {
    *p = 100;
}

int main() {
    int a = 10;
    update(&a);
    printf("%d", a);
}
```

---

#  Function Flow

```text id="fnflow"
main()
  |
  v
pass address of variable
  |
  v
function receives pointer
  |
  v
modifies original memory
  |
  v
value updated in main
```

---

#  Embedded Use Cases

* ADC buffer access
* UART data handling
* ISR flag control
* Memory-mapped registers
* RTOS task communication

---

#  Pointer in Embedded Systems

```text id="emb1"
Firmware Code
     |
     v
Pointers
     |
     v
Direct Memory Access
     |
     v
Hardware Registers
     |
     v
Peripheral Control
```

---

#  Advantages

| Advantage            | Explanation             |
| -------------------- | ----------------------- |
| Direct memory access | Faster hardware control |
| Efficient memory use | No copying of data      |
| Hardware mapping     | Register access         |
| Dynamic structures   | Linked lists            |
| Fast execution       | Minimal overhead        |

---

#  Disadvantages

| Issue              | Explanation                |
| ------------------ | -------------------------- |
| Memory corruption  | Wrong address access       |
| Segmentation fault | Invalid pointer            |
| Hard debugging     | Low-level errors           |
| Security risks     | Direct memory manipulation |
| Complexity         | Hard to maintain           |

---


