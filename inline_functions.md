# Inline Functions

## Overview

An **inline function** is a function where the compiler replaces the function call with the actual function code at compile time (if possible).

This reduces function call overhead and improves performance.

---

# 1. What is an Inline Function?

### Normal Function
```

function call → stack push → jump → return

```

### Inline Function
```

function code copied directly at call site

````

---

# 2. Basic Syntax

```c
inline int add(int a, int b) {
    return a + b;
}
``` id="inl1"

---

# 3. How It Works

### Code Example

```c
inline int square(int x) {
    return x * x;
}

int main() {
    int a = square(5);
}
````

### After Inlining (Conceptually)

````c
int main() {
    int a = (5 * 5);
}
``` id="inl2"

---

# 4. Why Use Inline Functions?

## Advantages

- Faster execution (no function call overhead)
- Useful for small, frequently used functions
- Better performance in embedded systems

---

## Disadvantages

- Code size increases (code duplication)
- Compiler may ignore inline request
- Not suitable for large functions

---

# 5. Inline vs Macro

| Feature | Inline Function | Macro |
|--------|----------------|-------|
| Type checking | Yes | No |
| Debugging | Easy | Hard |
| Evaluation safety | Safe | Risky |
| Performance | High | High |

---

## Macro Problem Example

```c
#define SQUARE(x) x * x
````

### Wrong output:

````
SQUARE(2+3) → 2+3*2+3 = 11
``` id="err1"

---

## Inline Solution

```c
inline int square(int x) {
    return x * x;
}
``` id="fix1"

---

# 6. Compiler Behavior

Important:

> `inline` is only a **request**, not a command.

Compiler may:
- Inline the function
- Ignore inlining
- Decide based on optimization level

---

## Example Compiler Flags

````

-O0 → no optimization (likely no inline)
-O2 → automatic inlining
-O3 → aggressive inlining

````id="opt1"

---

# 7. Static Inline Functions

Used in embedded systems to avoid multiple definitions.

```c
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
``` id="st1"

---

# 8. Inline in Header Files

### Example (important for embedded)

```c
// utils.h
static inline int square(int x) {
    return x * x;
}
``` id="hdr1"

### Why?

- Avoid linker errors
- Allow multiple inclusion safely
- Common in kernel code

---

# 9. Inline vs Function Call (Performance)

## Function Call

````

CALL → STACK PUSH → RETURN

```

## Inline Function

```

DIRECT CODE REPLACEMENT

````

---

# 10. When to Use Inline Functions

## Good Use Cases

- Small utility functions
- Bit manipulation
- Math helpers
- Hardware register access

### Example:

```c
static inline void set_bit(int *reg, int bit) {
    *reg |= (1 << bit);
}
``` id="emb1"

---

## Bad Use Cases

- Large functions
- Recursive functions
- Heavy loops
- Complex logic

---

# 11. Inline in Embedded Linux

Used heavily in:

- Kernel code
- Device drivers
- Hardware register access
- Performance-critical paths

---

## Example (Kernel-style)

```c
static inline int read_gpio(int pin) {
    return *(volatile int *)(GPIO_BASE + pin);
}
``` id="ker1"

---

# 12. Compiler Optimization Impact

| Optimization | Inline Behavior |
|-------------|----------------|
| -O0 | Mostly ignored |
| -O1 | Some inlining |
| -O2 | Recommended |
| -O3 | Aggressive inlining |

---

# 13. Common Mistakes

## 13.1 Large Inline Functions

❌ Bad:
```c
inline void big_function() {
    // 100 lines of code
}
````

---

## 13.2 Missing static in headers

Can cause multiple definition errors.

---

# 14. Debugging Inline Functions

Inline functions may not appear in debugger stack traces.

Use:

````
gcc -fno-inline
``` id="dbg1"

---

# 15. Summary

- Inline functions replace function calls with code
- Improve performance in embedded systems
- Compiler may ignore inline keyword
- Best for small, frequently used functions
- Common in Linux kernel and drivers

---


