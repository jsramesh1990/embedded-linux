
# Embedded C Interview Questions - Patch 2


[← Back to Main README](README.md) | [← Previous Patch](patch-1.md) | [Next Patch →](patch-3.md)

---

## Q34: #pragma pack vs Attributes

### 📚 Definition

Both `#pragma pack` and `__attribute__((packed))` control structure padding, but with different scope and portability characteristics.

### 📝 Syntax

```c
// #pragma pack method (MSVC, GCC, IAR)
#pragma pack(push, 1)  // Save current alignment, set to 1 byte

struct PackedStruct {
    uint8_t a;
    uint32_t b;
    uint16_t c;
};

#pragma pack(pop)  // Restore previous alignment

// GCC attribute method (GCC, ARMCC)
struct __attribute__((packed)) PackedStructGCC {
    uint8_t a;
    uint32_t b;
    uint16_t c;
};

// Per-member packing (GCC)
struct MixedPacking {
    uint8_t a;
    uint32_t b __attribute__((packed));
    uint16_t c;
};

// Specify exact alignment (not just packing)
struct __attribute__((aligned(16))) CacheAligned {
    uint32_t data[4];
};  // Structure aligned to 16-byte cache line

// Portable macro for multiple compilers
#ifdef __GNUC__
    #define PACKED __attribute__((packed))
#elif defined(__ARMCC_VERSION)
    #define PACKED __packed
#elif defined(_MSC_VER)
    #define PACKED __pragma(pack(push,1)) __pragma(pack(pop))
#else
    #error "Unknown compiler"
#endif

struct PACKED MyStruct {
    uint8_t a;
    uint32_t b;
};
```

### 💡 Explanation

| Feature | `#pragma pack` | `__attribute__((packed))` |
|---------|---------------|---------------------------|
| Scope | Region (push/pop) | Per-structure |
| Nesting | Yes | N/A |
| Portability | Wide (MSVC, GCC, IAR) | GCC/Clang only |
| Control | More flexible | Simpler |

### 📋 Interview Tip

```c
// WRONG - forgetting to restore packing
#pragma pack(1)
struct FileHeader { ... };
// Missing #pragma pack(pop) - affects all following structs!

// CORRECT - always restore
#pragma pack(push, 1)
struct FileHeader { ... };
#pragma pack(pop)
```

### 🎭 Visual Comedy

```
Without packing:
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ a │ P │ P │ P │ b │ b │ b │ b │  (8 bytes)
└───┴───┴───┴───┴───┴───┴───┴───┘

With packing:
┌───┬───┬───┬───┬───┐
│ a │ b │ b │ b │ b │  (5 bytes)
└───┴───┴───┴───┴───┘

"Packing: Squeezing structs like a suitcase.
More space, but slower access!" 🧳
```

---

## Q35: Volatile Effect on Optimization

### 📚 Definition

The `volatile` keyword forces the compiler to generate actual memory accesses instead of optimizing them away, even when they appear redundant.

### 📝 Syntax

```c
// Without volatile - compiler "optimizes"
void delay_bad(void) {
    uint32_t i;
    for(i = 0; i < 100000; i++);  // May be removed entirely!
}

// With volatile - compiler respects it
void delay_good(void) {
    volatile uint32_t i;
    for(i = 0; i < 100000; i++);  // Actually runs the loop
}

// See what compiler generates
// Without volatile (arm-none-eabi-gcc -O2):
// delay_bad:
//     bx lr     (function returns immediately!)

// With volatile:
// delay_good:
//     mov  r1, #100000
//     subs r1, #1
//     bne  loop
//     bx   lr

// Multiple accesses optimization
void status_check_bad(void) {
    uint32_t status = *((uint32_t*)0x40000000);
    if(status & 0x01) {
        if(status & 0x02) {  // Compiler may reuse first read!
            // status hasn't changed according to compiler
        }
    }
}

void status_check_good(void) {
    volatile uint32_t status = *((uint32_t*)0x40000000);
    // Or better:
    uint32_t status = *((volatile uint32_t*)0x40000000);
    // Forces new read each time
}

// Demonstrate optimization effect
int test_volatile_optimization(void) {
    uint32_t normal = 0;
    volatile uint32_t vol = 0;
    
    // Normal: compiler may count at compile time
    for(int i = 0; i < 100; i++) normal++;
    
    // Volatile: compiler must generate 100 increments
    for(int i = 0; i < 100; i++) vol++;
    
    return normal + vol;
}
```

### 💡 Explanation

**What volatile prevents:**
- Removal of unused variables
- Reordering of accesses
- Caching values in registers
- Dead code elimination

**Cost of volatile:**
- 2-10x slower (forces memory access)
- Prevents many optimizations
- Can't be optimized away

### 📋 Interview Tip

```c
// Common pattern: ISR flag
volatile uint8_t event_occurred = 0;

void ISR(void) {
    event_occurred = 1;  // Must be volatile
}

void main_loop(void) {
    while(!event_occurred);  // Without volatile, this could be infinite!
    event_occurred = 0;
}
```

### 🎭 Visual Comedy

```
Compiler without volatile:
    "I see you set 'status' to 0, then read it twice"
    "Since it never changes, I'll just use 0 both times"
    "Your code is now wrong, but faster!" 

Compiler with volatile:
    "You want me to actually READ memory? Every time?"
    "Fine, but this is going to be slower"
    "Happy now, Mr. Hardware Engineer?"

"Volatile: Making compilers do what you actually asked for!" 🔒
```

---

## Q36: Static Assert Implementation

### 📚 Definition

`static_assert` (C11) or compile-time assertion checks conditions at compile time, preventing compilation if the condition is false. Useful for validating sizes, offsets, and constants.

### 📝 Syntax

```c
#include <assert.h>  // For static_assert (C11)

// Check structure size at compile time
struct PackedConfig {
    uint8_t version;
    uint32_t baud_rate;
    uint16_t timeout;
};

static_assert(sizeof(struct PackedConfig) == 7, 
              "PackedConfig size is not 7 bytes! Check packing");

// Check offset of member
static_assert(offsetof(struct PackedConfig, baud_rate) == 1,
              "baud_rate at wrong offset!");

// C99-compatible static assert (if C11 not available)
#define STATIC_ASSERT(cond, msg) \
    typedef char static_assert_##msg[(cond) ? 1 : -1]

// Usage
STATIC_ASSERT(sizeof(uint32_t) == 4, uint32_t_size_wrong);

// Check bitfield sizes
#define MAX_DEVICES 16
static_assert(MAX_DEVICES <= 32, "Device count exceeds bitfield size");

// Check array bounds
#define BUFFER_SIZE 64
static_assert(BUFFER_SIZE > 0 && BUFFER_SIZE <= 1024,
              "Invalid buffer size");

// Hardware register validation
#define UART_BASE_ADDR 0x40004000
static_assert(UART_BASE_ADDR % 1024 == 0,
              "UART not aligned to 1KB boundary");

// Complex compile-time checks
static_assert((__STDC_VERSION__ >= 201112L),
              "C11 or later required");
```

### 💡 Explanation

**When static_assert fails:**
```
error: static assertion failed: "PackedConfig size is not 7 bytes! Check packing"
```

**Use cases:**
- Verify struct sizes match hardware expectations
- Ensure enums fit in intended storage
- Validate configuration constants
- Detect architecture assumptions

### 📋 Interview Tip

```c
// Protect against compiler differences
static_assert(CHAR_BIT == 8, "8-bit bytes required");

// Ensure integer sizes
static_assert(sizeof(void*) == 4, "32-bit platform required");

// Prevent buffer overflows
#define ARRAY_SIZE 10
uint32_t buffer[ARRAY_SIZE];
static_assert(ARRAY_SIZE * sizeof(uint32_t) <= 1024,
              "Buffer exceeds memory region");
```

### 🎭 Visual Comedy

```
Compiler: "Compiling..."
static_assert: "Is condition true?"
Condition: "No"
static_assert: "STOP! ERROR!"
Compiler: "But I want to continue..."
static_assert: "I SAID STOP!"

"Static assert: The bouncer at the compiler club.
Condition fails? YOU DON'T GET IN!" 🚫
```

---

## Q37: Custom Section Placement (__attribute__((section)))

### 📚 Definition

The `section` attribute places variables or functions in specific memory sections defined in the linker script, giving fine control over memory placement.

### 📝 Syntax

```c
// Place in custom sections
uint32_t __attribute__((section(".sdram"))) framebuffer[1024*768];
const char __attribute__((section(".external_flash"))) lookup_table[256];

// Fast IRAM for critical code
void __attribute__((section(".itcm_ram"))) fast_irq_handler(void) {
    // Critical interrupt code - runs from TCM
}

// Non-initialized section (no startup zero)
uint32_t __attribute__((section(".noinit"))) boot_counter;

// Multiple attributes combined
uint32_t __attribute__((section(".dma_buffer"), aligned(32))) 
    dma_rx_buffer[256];

// Linker script snippet (custom.ld)
/*
SECTIONS {
    .sdram (NOLOAD) : {
        *(.sdram)
    } > SDRAM
    
    .itcm_ram : {
        *(.itcm_ram)
    } > ITCM
    
    .noinit (NOLOAD) : {
        *(.noinit)
    } > RAM
}
*/

// Function pointers in specific section
typedef void (*handler_t)(void);
handler_t __attribute__((section(".vectors"))) vector_table[16];

// C++ constructors in custom section
__attribute__((section(".init_array"))) 
void (*constructor_array[])(void) = { constructor1, constructor2 };
```

### 💡 Explanation

**Common section uses:**
- `.noinit`: Variables that survive reset (boot counter, error logs)
- `.sdram`: Large buffers in external memory
- `.itcm`: Performance-critical code
- `.dma_buffer`: Cache-aligned buffers for DMA

### 📋 Interview Tip

```c
// Verify placement with map file
// arm-none-eabi-objdump -t firmware.elf | grep framebuffer
// Or use __attribute__((used)) to prevent removal
uint32_t __attribute__((section(".special"), used)) magic = 0xDEADBEEF;
```

### 🎭 Visual Comedy

```
Default placement:
"Where should I put this variable?"
Linker: "In .bss, I guess"
Developer: "But I want it in SDRAM!"
Linker: "Too bad"

With section attribute:
Developer: "Put this in .sdram"
Linker: "Yes sir, right away sir!"

"Custom sections: Giving your variables their own apartments
instead of a shared dorm room!" 🏢
```

---

## Q38: Tail Call Optimization

### 📚 Definition

Tail call optimization (TCO) reuses the current function's stack frame for a called function when the call is the last action, preventing stack growth for recursive functions.

### 📝 Syntax

```c
// Without tail call optimization (recursive factorial)
uint32_t factorial_recursive(uint32_t n) {
    if (n <= 1) return 1;
    return n * factorial_recursive(n - 1);  // NOT tail call (multiplication after)
}
// Stack usage: O(n) - crashes at n=1000

// Tail recursive version
uint32_t factorial_tail(uint32_t n, uint32_t accumulator) {
    if (n <= 1) return accumulator;
    return factorial_tail(n - 1, n * accumulator);  // Tail call!
}
// Stack usage: O(1) with TCO enabled

// Enable TCO
// arm-none-eabi-gcc -O2 -foptimize-sibling-calls

// State machine using tail calls
typedef enum { STATE_INIT, STATE_RUN, STATE_ERROR } state_t;

void state_machine(state_t state, uint32_t data) {
    switch(state) {
        case STATE_INIT:
            data = initialize();
            return state_machine(STATE_RUN, data);  // Tail call
        case STATE_RUN:
            if(run_cycle(data)) {
                return state_machine(STATE_RUN, data + 1);
            } else {
                return state_machine(STATE_ERROR, data);
            }
        case STATE_ERROR:
            handle_error(data);
            return;  // Base case
    }
}

// Check if compiler optimized
// arm-none-eabi-objdump -d firmware.elf | grep factorial_tail
// Look for "b factorial_tail" (branch) not "bl factorial_tail" (call)

// Force tail call (GCC)
__attribute__((optimize("O2")))
uint32_t tail_optimized(uint32_t x) {
    if (x == 0) return 0;
    return tail_optimized(x - 1);  // Tail call
}
```

### 💡 Explanation

**Without TCO:**
```
factorial_recursive(5)
    push frame
    factorial_recursive(4)
        push frame
        factorial_recursive(3)
            push frame
            ... (stack grows)
```

**With TCO:**
```
factorial_tail(5, 1)
    jump (no push) → factorial_tail(4, 5)
    jump → factorial_tail(3, 20)
    jump → factorial_tail(2, 60)
    jump → factorial_tail(1, 120)
    return 120
```

### 📋 Interview Tip

**Limitations on ARM:**
- TCO not always enabled at -O0
- May not work with complex return types
- Debugging harder (no stack frames)

### 🎭 Visual Comedy

```
Without TCO:
Stack: [Frame1] [Frame2] [Frame3] ... [CRASH!]

With TCO:
Stack: [Frame1] (reused repeatedly)

"Tail call optimization: The 'reusable cup' of function calls.
Save the stack, save the planet!" 🌍
```

---

## Q39: restrict Pointer Optimization

### 📚 Definition

The `restrict` keyword tells the compiler that pointers do not alias (point to overlapping memory), enabling aggressive optimizations.

### 📝 Syntax

```c
// Vector addition with restrict
void vector_add(float *restrict dest, 
                const float *restrict src1,
                const float *restrict src2, 
                int n) {
    for(int i = 0; i < n; i++) {
        dest[i] = src1[i] + src2[i];
    }
}

// Without restrict - compiler must assume overlap
void vector_add_slow(float *dest, const float *src1, const float *src2, int n) {
    for(int i = 0; i < n; i++) {
        dest[i] = src1[i] + src2[i];
        // Compiler: "dest might be same as src1, must reload each iteration"
    }
}

// With restrict - compiler can use SIMD
// Generates NEON instructions on ARM:
// vld1.32 {q0}, [r1]!    (load 4 floats)
// vld1.32 {q1}, [r2]!    (load 4 floats)
// vadd.f32 q0, q0, q1    (add them)
// vst1.32 {q0}, [r0]!    (store result)

// Matrix multiplication (restrict helps a lot)
void matmul(float *restrict c,
            const float *restrict a,
            const float *restrict b,
            int n) {
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            float sum = 0;
            for(int k = 0; k < n; k++) {
                sum += a[i*n + k] * b[k*n + j];
            }
            c[i*n + j] = sum;
        }
    }
}
```

### 💡 Explanation

**Performance difference:**
- Without restrict: 1-2 operations per iteration
- With restrict: 4-16 operations per iteration (SIMD vectorization)

**Danger of restrict:**
```c
// BROKEN - violates restrict promise
float data[10];
vector_add(data, data + 1, data + 2, 8);  // Overlap! Undefined behavior!
```

### 📋 Interview Tip

**When to use restrict:**
- ✅ Performance-critical loops
- ✅ DSP processing
- ✅ Known independent buffers (DMA buffers)
- ❌ When pointers might alias (defeats purpose)

### 🎭 Visual Comedy

```
Compiler without restrict:
"These pointers might point to same memory"
"I'll play it safe - one element at a time"
*pessimized code*

Compiler with restrict:
"You PROMISE they don't overlap?"
"Great! I'll use SIMD, unroll loops, do magic!"
*optimized code*

"restrict: The compiler's permission slip to go fast!" 🏎️
```

---

## Q40: Optimized memcpy for Aligned Data

### 📚 Definition

Custom memcpy optimized for aligned data can be much faster than standard memcpy by using word-sized copies instead of byte-by-byte.

### 📝 Syntax

```c
#include <stdint.h>

// Optimized for 32-bit aligned, 32-bit copies
void* memcpy_aligned32(void *dest, const void *src, size_t n) {
    uint32_t *d = (uint32_t*)dest;
    const uint32_t *s = (const uint32_t*)src;
    size_t words = n / 4;
    
    // Copy 32-bit words
    for(size_t i = 0; i < words; i++) {
        d[i] = s[i];
    }
    
    // Handle remaining bytes
    size_t bytes = n % 4;
    if(bytes) {
        uint8_t *d8 = (uint8_t*)&d[words];
        const uint8_t *s8 = (const uint8_t*)&s[words];
        for(size_t i = 0; i < bytes; i++) {
            d8[i] = s8[i];
        }
    }
    
    return dest;
}

// Even faster - loop unrolling
void* memcpy_aligned32_fast(void *dest, const void *src, size_t n) {
    uint32_t *d = (uint32_t*)dest;
    const uint32_t *s = (const uint32_t*)src;
    size_t words = n / 4;
    
    // Unroll 4x
    while(words >= 4) {
        d[0] = s[0];
        d[1] = s[1];
        d[2] = s[2];
        d[3] = s[3];
        d += 4;
        s += 4;
        words -= 4;
    }
    
    // Handle remainder
    while(words--) {
        *d++ = *s++;
    }
    
    // Handle remaining bytes...
    return dest;
}

// ARM Cortex-M7 optimized with prefetch
void* memcpy_arm_optimized(void *dest, const void *src, size_t n) {
    // Use built-in for best performance
    __builtin_memcpy(dest, src, n);
    
    // Or use PLD (prefetch) instruction
    #ifdef __ARM_ARCH_7EM__
    __asm volatile (
        "pld [%0, #64]\n"  // Prefetch 64 bytes ahead
        : : "r"(src)
    );
    #endif
    
    return __builtin_memcpy(dest, src, n);
}

// Compile-time check alignment
static_assert(__alignof(uint32_t) == 4, "Unaligned uint32_t");

void safe_memcpy(void *dest, const void *src, size_t n) {
    // Check alignment
    if(((uintptr_t)dest & 3) == 0 && ((uintptr_t)src & 3) == 0) {
        memcpy_aligned32(dest, src, n);  // Fast path
    } else {
        memcpy(dest, src, n);  // Slow but safe
    }
}
```

### 💡 Explanation

**Performance comparison (Cortex-M7, 216MHz):**
| Method | Speed (MB/s) | Size (bytes) |
|--------|--------------|--------------|
| Byte loop | 15 MB/s | 32 |
| Word loop | 60 MB/s | 48 |
| Unrolled word | 85 MB/s | 96 |
| `__builtin_memcpy` | 120 MB/s | Variable |
| DMA memcpy | 216 MB/s | Large overhead |

### 📋 Interview Tip

```c
// Use DMA for large copies
void dma_memcpy(void *dest, const void *src, size_t n) {
    if(n > 256) {  // Threshold for DMA overhead
        start_dma_transfer(dest, src, n);
        wait_dma_complete();
    } else {
        __builtin_memcpy(dest, src, n);  // Small: CPU is faster
    }
}
```

### 🎭 Visual Comedy

```
Byte copy:
[1][2][3][4] → 4 operations

Word copy:
[1,2,3,4] → 1 operation

Unrolled word copy:
[1,2,3,4][5,6,7,8] → 2 operations but pipeline happy!

"memcpy optimization: Copying data like FedEx instead of Pony Express!" 📦
```

---

## Q41: __builtin_expect Branch Prediction

### 📚 Definition

`__builtin_expect` gives the compiler a hint about which branch is more likely, allowing better instruction prefetch and pipeline optimization.

### 📝 Syntax

```c
#define likely(x)   __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)

// Error handling - unlikely path
void process_data(uint32_t *data) {
    if(unlikely(data == NULL)) {
        handle_error();  // Compiler puts this far away
        return;
    }
    
    // Normal processing - likely path (prefetched)
    do_processing(data);
}

// Loop with exit condition
uint32_t find_value(uint32_t *array, uint32_t len, uint32_t target) {
    for(uint32_t i = 0; i < len; i++) {
        if(likely(array[i] != target)) {
            continue;  // Likely not found yet
        }
        return i;  // Unlikely found
    }
    return NOT_FOUND;
}

// State machine with likely states
void state_handler(void) {
    if(likely(state == STATE_RUNNING)) {
        handle_running();  // Likely path - fast
    } else if(unlikely(state == STATE_ERROR)) {
        handle_error();    // Unlikely - moved away
    } else {
        handle_other();    // Other states
    }
}

// Check generated assembly
// With __builtin_expect:
//     cmp r0, #0
//     beq .error_path  (branch to far away)
//     bl do_processing (fall through - fast)
//
// Without:
//     cmp r0, #0
//     bne .process     (branch likely)
//     bl handle_error
// .process:
//     bl do_processing (missed branch)

// Complex conditions
int validate_input(int x, int y) {
    // Normal inputs: x between 0-100, y between 0-50
    if(unlikely(x < 0 || x > 100)) return -1;
    if(unlikely(y < 0 || y > 50)) return -2;
    
    // Likely path
    return x * y;
}
```

### 💡 Explanation

**How it works:**
- Compiler arranges code so likely path has no branch
- Likely path follows sequentially (prefetched)
- Unlikely path is moved to end of function

**When to use:**
- Error handling
- Loop exit conditions
- State machines with dominant states
- Initialization code (unlikely after first run)

### 📋 Interview Tip

```c
// Overuse hurts performance
if(likely(a == 1)) { ... }  // Only if really 99% true

// Measure before using
// Add counters, verify branch ratio
```

### 🎭 Visual Comedy

```
CPU pipeline with __builtin_expect:
                  [Likely path] → prefetched, ready!
Unlikely path:    [Far away] → "I'll get there if needed"

CPU pipeline without hints:
                  [Branch?] → "Hmm, which way?"
                  "I'll guess... WRONG!"
                  *pipeline flush* *stall*

"Branch prediction hints: GPS for your CPU's instruction fetcher!" 🧭
```

---

## Q42: RAM Functions for Flash Programming

### 📚 Definition

Functions that run from RAM instead of Flash, needed when programming the Flash memory itself (can't execute from Flash while erasing/programming it).

### 📝 Syntax

```c
// Place function in RAM section
__attribute__((section(".ramfunc")))
void flash_program_word(uint32_t address, uint32_t data) {
    // This code runs from RAM
    
    // Unlock flash
    FLASH->KEYR = FLASH_KEY1;
    FLASH->KEYR = FLASH_KEY2;
    
    // Wait for ready
    while(FLASH->SR & FLASH_SR_BSY);
    
    // Program word
    *(volatile uint32_t*)address = data;
    
    // Wait for completion
    while(FLASH->SR & FLASH_SR_BSY);
    
    // Lock flash
    FLASH->CR |= FLASH_CR_LOCK;
}

// Copy to RAM at startup
extern uint32_t __ramfunc_start;
extern uint32_t __ramfunc_end;
extern uint32_t __ramfunc_load_start;

void copy_ramfunc_to_ram(void) {
    uint32_t *src = &__ramfunc_load_start;
    uint32_t *dst = &__ramfunc_start;
    
    while(dst < &__ramfunc_end) {
        *dst++ = *src++;
    }
}

// Linker script entries:
/*
.ramfunc : {
    . = ALIGN(4);
    __ramfunc_start = .;
    *(.ramfunc)
    . = ALIGN(4);
    __ramfunc_end = .;
} > RAM AT> FLASH
__ramfunc_load_start = LOADADDR(.ramfunc);
*/

// Function pointer to RAM function
typedef void (*flash_func_t)(uint32_t, uint32_t);
flash_func_t flash_func = (flash_func_t)&flash_program_word;

// Call from RAM
flash_func(0x08002000, 0x12345678);

// Alternative: Copy on demand
void execute_ram_function(void (*func)(void)) {
    // Copy just this function to RAM
    uint32_t func_size = get_function_size(func);
    uint32_t *ram_copy = malloc(func_size);
    memcpy(ram_copy, func, func_size);
    
    // Execute from RAM
    ((void(*)(void))ram_copy)();
    
    free(ram_copy);
}
```

### 💡 Explanation

**Why RAM functions needed:**
- Flash controller can't read while writing
- Executing from Flash while programming causes prefetch abort
- Must relocate code to RAM temporarily

**Typical use cases:**
- Flash programming
- Bootloader updates
- Critical interrupt handlers (lower latency from RAM)

### 📋 Interview Tip

```c
// Check if function is in RAM
int is_function_in_ram(void *func) {
    extern uint32_t __ram_start, __ram_end;
    return (func >= &__ram_start && func < &__ram_end);
}

// Debug: Print function location
printf("flash_program_word at %p\n", flash_program_word);
```

### 🎭 Visual Comedy

```
Normal execution:
CPU: "Execute from Flash"
Flash: "Reading... here's your instruction"

Flash programming:
CPU: "Erase Flash block!"
Flash: "I'm busy erasing..."
CPU: "I need the next instruction!"
Flash: "Can't read, I'm erasing!"
CPU: *crash*

RAM function:
CPU: "Run from RAM"
RAM: "Here's the special code"
CPU: "Now erase Flash!"
Flash: "Erasing..."
CPU: "Still have RAM for next instruction!"

"RAM functions: The designated driver for flash operations!" 🚗
```

---

## Quick Reference Card 3

### Optimization Attributes

| Attribute | Purpose | Trade-off |
|-----------|---------|-----------|
| `packed` | Remove padding | Speed for size |
| `aligned` | Force alignment | Size for speed |
| `section` | Custom placement | Manual management |
| `restrict` | No aliasing promise | Safety for speed |
| `likely/unlikely` | Branch hints | Complexity |

### Memory Copy Methods

| Method | Best for | Speed |
|--------|----------|-------|
| Byte loop | Small, unaligned | Slow |
| Word loop | Aligned <256 bytes | Medium |
| Unrolled word | Aligned 256-1KB | Fast |
| Builtin memcpy | Any aligned | Fastest |
| DMA | >1KB | Hardware |

### Branch Prediction Hints

```c
#define likely(x)   __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)

// Use for:
if(unlikely(error))           // Errors
if(likely(success))           // Success paths  
if(unlikely(loop_exit))       // Loop breaks
if(likely(main_state))        // Dominant states
```

---

