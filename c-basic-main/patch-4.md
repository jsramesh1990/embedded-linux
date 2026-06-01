
# Embedded C Interview Questions - Patch 4 (Final)

[← Back to Main README](README.md) | [← Previous Patch](patch-3.md)

---

## Q51: Custom assert() Implementation

### 📚 Definition

A custom assert macro that works in embedded environments where standard assert may not be available or needs special handling (logging, LED patterns, safe shutdown).

### 📝 Syntax

```c
// Custom assert with different behaviors per build
#ifdef DEBUG
    #define ASSERT(expr) \
        do { \
            if(!(expr)) { \
                assert_failed(__FILE__, __LINE__, __FUNCTION__); \
            } \
        } while(0)
#else
    #define ASSERT(expr) ((void)0)  // Removed in release
#endif

// Production-safe assert (always enabled)
#define SAFE_ASSERT(expr) \
    do { \
        if(!(expr)) { \
            safe_assert_handler(__FILE__, __LINE__); \
        } \
    } while(0)

// Assert handler with logging
void assert_failed(const char *file, int line, const char *func) {
    // Disable interrupts to prevent corruption
    __disable_irq();
    
    // Log to non-volatile memory
    error_log_t error = {
        .type = ERROR_ASSERT,
        .file = file,
        .line = line,
        .func = func,
        .timestamp = get_system_time()
    };
    save_error_log(&error);
    
    // Blink error code on LED (5 long, 5 short = assert)
    blink_error_code(5, 5);
    
    // Enter safe state
    system_safe_shutdown();
    
    // Infinite loop - watchdog will reset
    while(1);
}

// Assert with expression string
#define ASSERT_MSG(expr, msg) \
    do { \
        if(!(expr)) { \
            assert_msg_failed(__FILE__, __LINE__, msg); \
        } \
    } while(0)

// Memory-safe assert (no printf)
void assert_msg_failed(const char *file, int line, const char *msg) {
    // Use RTT or semihosting (not printf)
    SEGGER_RTT_printf(0, "Assert: %s at %s:%d\n", msg, file, line);
    
    // Break into debugger
    __BKPT();
    
    while(1);
}

// Assert with recovery
#define ASSERT_RECOVER(expr, recovery) \
    do { \
        if(!(expr)) { \
            recovery(); \
            return ERROR_CODE; \
        } \
    } while(0)

// Parameter validation
int uart_write_buffer(const uint8_t *data, uint32_t len) {
    ASSERT_RECOVER(data != NULL, uart_recover);
    ASSERT_RECOVER(len > 0, uart_recover);
    ASSERT_RECOVER(len <= UART_MAX_BUFFER, uart_recover);
    
    // Actual function code
    return uart_do_write(data, len);
}

// Compile-time assert (no runtime cost)
#define STATIC_ASSERT(expr, msg) \
    typedef char static_assert_##msg[(expr) ? 1 : -1]

// Usage
STATIC_ASSERT(sizeof(struct Config) == 64, config_size_wrong);

// Assert with context data
typedef struct {
    uint32_t pc;        // Program counter
    uint32_t lr;        // Link register
    uint32_t sp;        // Stack pointer
    uint32_t registers[8]; // r0-r7
} assert_context_t;

assert_context_t g_assert_context;

void assert_save_context(void) {
    __asm volatile(
        "mov %0, r0\n"
        "mov %1, lr\n"
        "mov %2, sp\n"
        "stm %3, {r0-r7}\n"
        : "=r"(g_assert_context.pc),
          "=r"(g_assert_context.lr),
          "=r"(g_assert_context.sp)
        :
        : "memory"
    );
}
```

### 💡 Explanation

**Assert levels:**
| Level | Enabled | Use case |
|-------|---------|----------|
| DEBUG | Debug builds | Development |
| RELEASE | Never | Production (size) |
| SAFE | Always | Safety-critical |
| RECOVER | Always | Error recovery |

### 📋 Interview Tip

```c
// Lightweight assert (no strings, no heap)
#define LIGHT_ASSERT(expr) \
    ((expr) ? (void)0 : assert_light_failed(__LINE__))

void assert_light_failed(int line) {
    // Store line number in retention register
    RTC->BKP0R = line;
    NVIC_SystemReset();
}
```

### 🎭 Visual Comedy

```
Regular code: "I'm sure this pointer is valid..."
Assert: "ARE YOU SURE? Let me check..."
Pointer: NULL
Assert: "I KNEW IT! *crash*"

"Assert: Your code's conscience that won't let you get away with anything!" 👮
```

---

## Q52: Watchpoint Using DWT (Data Watchpoint and Trace)

### 📚 Definition

The Data Watchpoint and Trace (DWT) unit in ARM Cortex-M can monitor memory addresses and trigger a debug event when variables are read or written, catching memory corruption.

### 📝 Syntax

```c
// Configure DWT watchpoint
typedef enum {
    WATCHPOINT_READ = 0x1,
    WATCHPOINT_WRITE = 0x2,
    WATCHPOINT_ACCESS = 0x3
} watchpoint_access_t;

void dwt_watchpoint_set(uint32_t address, watchpoint_access_t access) {
    // Check if DWT is present
    if(!(CoreDebug->DEMCR & CoreDebug_DEMCR_TRCENA_Msk)) {
        CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
    }
    
    // Configure comparator 0
    DWT->COMP0 = address;           // Address to watch
    DWT->MASK0 = 0;                 // Exact address (no mask)
    DWT->FUNCTION0 = (access << 0)  // Read/Write/Access
                    | (1 << 4);     // Enable
}

// Watch a variable
volatile uint32_t critical_variable;

void watch_critical_variable(void) {
    dwt_watchpoint_set((uint32_t)&critical_variable, WATCHPOINT_WRITE);
}

// DWT interrupt handler
void DWT_IRQHandler(void) {
    if(DWT->FUNCTION0 & DWT_FUNCTION0_WATCHED) {
        uint32_t pc = __builtin_return_address(0);
        
        printf("Watchpoint hit at address 0x%08X\n", pc);
        printf("PC = 0x%08X, LR = 0x%08X\n", pc, __builtin_return_address(1));
        
        // Save debug info
        save_corruption_log(pc);
        
        // Clear flag
        DWT->FUNCTION0 &= ~DWT_FUNCTION0_WATCHED;
        
        // Optional: break into debugger
        __BKPT();
    }
}

// Watch range of addresses
void watch_memory_range(uint32_t start, uint32_t end, watchpoint_access_t access) {
    // Calculate mask (size must be power of 2)
    uint32_t size = end - start + 1;
    uint32_t mask = 0;
    
    while(size > 1) {
        mask++;
        size >>= 1;
    }
    
    DWT->COMP0 = start;
    DWT->MASK0 = mask;
    DWT->FUNCTION0 = (access << 0) | (1 << 4);
}

// Watch a specific bit within a word
void watch_bit(uint32_t address, uint8_t bit, watchpoint_access_t access) {
    // Use bitmask to ignore other bits
    uint32_t bitmask = 1 << bit;
    
    // DWT can't watch bits directly, but can use comparator with mask
    DWT->COMP0 = address;
    DWT->FUNCTION0 = (access << 0) | (1 << 4);
    // Will trigger on any access to the word
}

// Data trace (record all writes to variable)
typedef struct {
    uint32_t timestamp;
    uint32_t value;
    uint32_t pc;
} write_history_t;

write_history_t write_history[64];
volatile uint32_t history_index = 0;

void DWT_IRQHandler(void) {
    if(DWT->FUNCTION0 & DWT_FUNCTION0_WATCHED) {
        uint32_t idx = history_index++;
        write_history[idx].timestamp = DWT->CYCCNT;
        write_history[idx].value = *(uint32_t*)DWT->COMP0;
        write_history[idx].pc = __builtin_return_address(0);
        
        if(history_index >= 64) history_index = 0;  // Wrap
    }
}

// Multiple watchpoints
void setup_multiple_watchpoints(void) {
    // Use multiple comparators (0-3 available)
    DWT->COMP0 = (uint32_t)&var1;
    DWT->FUNCTION0 = WATCHPOINT_WRITE | (1 << 4);
    
    DWT->COMP1 = (uint32_t)&var2;
    DWT->FUNCTION1 = WATCHPOINT_READ | (1 << 4);
    
    DWT->COMP2 = (uint32_t)&array[0];
    DWT->MASK2 = 2;  // 4-byte range
    DWT->FUNCTION2 = WATCHPOINT_ACCESS | (1 << 4);
}
```

### 💡 Explanation

**DWT features:**
- 4 comparators (Cortex-M3/M4/M7)
- Can watch data or PC (hardware breakpoints)
- Minimal performance impact (0 cycles when not triggered)
- Works in real-time (no halting needed for trace)

### 📋 Interview Tip

```c
// Detect stack corruption
void watch_stack_canary(void) {
    extern uint32_t __StackCanary;
    dwt_watchpoint_set((uint32_t)&__StackCanary, WATCHPOINT_WRITE);
}
```

### 🎭 Visual Comedy

```
Debugger: "Something corrupted my variable!"
Watchpoint: "I'll watch it like a hawk"
Variable: *gets written*
Watchpoint: "HALT! I saw that!"
Debugger: "Gotcha! You were at 0x08001234!"

"Watchpoints: Security cameras for your variables!" 📹
```

---

## Q53: JTAG vs SWD vs Semihosting

### 📚 Definition

Debug interfaces for ARM microcontrollers: JTAG (5-pin, older), SWD (2-pin, modern), and semihosting (uses debugger for I/O).

### 📝 Syntax

```c
// Semihosting example
#include <stdio.h>

void semihosting_example(void) {
    // These go to debugger console (not actual UART)
    printf("Hello from semihosting!\n");
    
    // Read character from debugger
    char c = getchar();
    
    // Works even if no UART is configured!
}

// Disable semihosting in release builds
#ifdef RELEASE
    #pragma import(__use_no_semihosting)
    
    // Provide dummy implementations
    void _sys_exit(int x) { while(1); }
    FILE __stdout;
#endif

// Custom semihosting calls
#define SYS_WRITE0 0x04  // Write string
#define SYS_READC  0x07  // Read character

void semihost_write(const char *str) {
    #ifdef DEBUG
    __asm volatile (
        "mov r0, %0\n"
        "mov r1, %1\n"
        "bkpt 0xAB\n"
        : : "r"(SYS_WRITE0), "r"(str) : "r0", "r1"
    );
    #endif
}

// Auto-detect debugger presence
int is_debugger_connected(void) {
    // Check DHCSR register
    return (CoreDebug->DHCSR & CoreDebug_DHCSR_C_DEBUGEN_Msk) != 0;
}

// Conditional semihosting
void debug_output(const char *msg) {
    if(is_debugger_connected()) {
        semihost_write(msg);  // Only if debugger attached
    }
    // Otherwise do nothing (no crash)
}

// SWD vs JTAG detection
void detect_debug_interface(void) {
    uint32_t idcode = DBGMCU->IDCODE;
    uint8_t revision = (idcode >> 28) & 0xF;
    uint8_t device_id = idcode & 0xFFF;
    
    if(idcode != 0) {
        // Debug interface active
        printf("Debugger connected via ");
        // Check SWDIO pin (typical detection)
        if(GPIOA->IDR & GPIO_PIN_13) {
            printf("SWD\n");
        } else {
            printf("JTAG\n");
        }
    }
}
```

### 💡 Explanation

**Interface comparison:**

| Feature | JTAG | SWD | Semihosting |
|---------|------|-----|-------------|
| Pins | 5 (TMS, TCK, TDI, TDO, nSRST) | 2 (SWDIO, SWCLK) | 0 (uses debug pins) |
| Speed | Up to 50MHz | Up to 50MHz | Slow (polled) |
| Features | Boundary scan, trace | Debug only | Console I/O |
| Target impact | High (more pins) | Low | Medium (polling) |
| Modern MCUs | Optional | Primary | Always |

### 📋 Interview Tip

```c
// Redirect printf to semihosting or UART automatically
int _write(int file, char *ptr, int len) {
    if(is_debugger_connected()) {
        for(int i = 0; i < len; i++) {
            semihost_write_char(ptr[i]);
        }
    } else {
        uart_write(ptr, len);
    }
    return len;
}
```

### 🎭 Visual Comedy

```
JTAG: [TAKES 5 PINS] "I'm the old standard!"
SWD:  [USES 2 PINS] "I'm the new hotness!"
Semihosting: [USES 0 PINS] "I'm cheating!"

Developer: "Can I have all three?"
MCU: "Pick one, I don't have enough pins!"

"Debug interfaces: The battle of the protocol dinosaurs!" 🦕
```

---

## Q54: Stack Canary Implementation

### 📚 Definition

A stack canary is a known value placed between stack variables and return addresses to detect stack buffer overflows before they corrupt critical data.

### 📝 Syntax

```c
// Software stack canary
typedef struct {
    uint32_t canary;
    uint32_t saved_lr;
    uint32_t saved_r4;
    // ... other saved registers
} stack_frame_t;

#define STACK_CANARY_VALUE 0xDEADBEEF

__attribute__((noinline))
void function_with_canary(void) {
    uint32_t canary = STACK_CANARY_VALUE;
    
    char buffer[64];
    
    // Dangerous operation
    gets(buffer);  // Potential overflow!
    
    // Check canary
    if(canary != STACK_CANARY_VALUE) {
        // Stack corrupted!
        stack_corruption_handler();
    }
}

// Compiler-assisted stack protector (GCC)
// Compile with: -fstack-protector-strong
void vulnerable_function(void) {
    char buffer[64];
    gets(buffer);  // Compiler adds canary automatically
}

// Custom per-task canary for RTOS
typedef struct {
    uint32_t task_canary;
    uint32_t stack_start;
    uint32_t stack_end;
    char task_name[16];
} task_protection_t;

task_protection_t tasks[MAX_TASKS];

void task_stack_init(TaskHandle_t task, const char *name) {
    task_protection_t *prot = &tasks[task_index];
    
    // Fill entire stack with canary pattern
    uint32_t *stack_top = (uint32_t*)task->pxStack;
    for(uint32_t i = 0; i < task->uxStackSize; i++) {
        stack_top[i] = 0xDEADBEEF;
    }
    
    prot->task_canary = STACK_CANARY_VALUE;
    prot->stack_start = (uint32_t)task->pxStack;
    prot->stack_end = prot->stack_start + (task->uxStackSize * 4);
    strcpy(prot->task_name, name);
}

void check_all_task_stacks(void) {
    for(int i = 0; i < task_count; i++) {
        uint32_t *stack = (uint32_t*)tasks[i].stack_start;
        uint32_t corrupted = 0;
        
        // Check each word of stack
        for(uint32_t j = 0; j < (tasks[i].stack_end - tasks[i].stack_start)/4; j++) {
            if(stack[j] != STACK_CANARY_VALUE) {
                corrupted = stack[j];
                break;
            }
        }
        
        if(corrupted) {
            printf("Task %s stack corrupted at offset %lu! Value: 0x%08X\n",
                   tasks[i].task_name, j, corrupted);
        }
    }
}

// Hardware stack limit checking (ARMv7-M)
void enable_stack_limit_check(void) {
    // Set stack limit register
    uint32_t stack_limit = (uint32_t)&__StackLimit;
    __set_SPLIM(stack_limit);
    
    // Enable stack limit checking
    // Any access below SP limit causes UsageFault
}

// Stack canary in linker script
/*
SECTIONS {
    .stack_canary : {
        __stack_canary_start = .;
        LONG(0xDEADBEEF)
        __stack_canary_end = .;
    } > RAM
}
*/

extern uint32_t __stack_canary_start;
extern uint32_t __stack_canary_end;

void check_linker_canary(void) {
    if(__stack_canary_start != 0xDEADBEEF) {
        // Stack canary overwritten!
        system_error(ERROR_STACK_CORRUPT);
    }
}

// Interrupt stack canary
uint32_t __attribute__((section(".isr_stack_canary"))) isr_stack_canary = 0xDEADBEEF;

void check_isr_stack(void) {
    if(isr_stack_canary != 0xDEADBEEF) {
        // ISR stack overflow!
        // Can't log (ISR context), just reset
        NVIC_SystemReset();
    }
}
```

### 💡 Explanation

**Canary effectiveness:**
- Stack smashing usually overwrites sequentially
- Return address comes after local variables
- Canary in between catches overflow before return

**GCC options:**
| Flag | Protection | Overhead |
|------|-----------|----------|
| `-fstack-protector` | Functions with large buffers | Low |
| `-fstack-protector-strong` | More aggressive | Medium |
| `-fstack-protector-all` | Every function | High |

### 📋 Interview Tip

```c
// Canary with random value
uint32_t get_random_canary(void) {
    // Use hardware RNG if available
    if(RNG->SR & RNG_SR_DRDY) {
        return RNG->DR;
    }
    // Fallback: XOR of various sources
    return (uint32_t)&get_random_canary ^ DWT->CYCCNT ^ SCB->CPUID;
}
```

### 🎭 Visual Comedy

```
Stack layout:
[Local vars] → [CANARY] → [Return address]
                      ↑
                      Guard dog

Buffer overflow: "I'll fill everything!"
[Local vars]→[*overflow*]→[CANARY: 0xDEAD??]
Canary: "BARK BARK!"
Handler: "Who's a good canary? You are!"

"Stack canary: The guard dog that bites back at buffer overflows!" 🐕
```

---

## Q55: Weak Attribute and Overriding

### 📚 Definition

The `weak` attribute allows symbols (functions, variables) to be overridden by strong definitions elsewhere, enabling default implementations that can be replaced.

### 📝 Syntax

```c
// Default weak implementation
__attribute__((weak))
void Default_Handler(void) {
    // Default: infinite loop
    while(1);
}

// Override in user code (strong symbol)
void USART1_IRQHandler(void) {
    // User implementation - overrides weak default
    uart_process();
}

// Weak variable
__attribute__((weak)) uint32_t config_value = 0;

// Strong override in another file
uint32_t config_value = 1000;  // Overrides weak

// Multiple weaks - last one wins
__attribute__((weak, alias("default_implementation")))
void optional_function(void);

void default_implementation(void) {
    printf("Using default\n");
}

// Weak function with fallback
__attribute__((weak))
uint32_t get_temperature(void) {
    // Default: return fake value
    return 25;
}

// Application can override
uint32_t get_temperature(void) {
    return read_sensor();
}

// Weak in library - allows customization
// lib.c
__attribute__((weak))
void system_init_hook(void) {
    // Weak - does nothing by default
}

void system_init(void) {
    // Hardware init
    clock_init();
    gpio_init();
    
    // Hook for user code
    system_init_hook();  // Can be overridden
}

// user.c
void system_init_hook(void) {
    // User initialization runs automatically
    my_custom_init();
}

// Weak for interrupt vectors (common in startup files)
typedef void (*vector_t)(void);

__attribute__((weak)) vector_t vectors[] = {
    (vector_t)&__stack_end,  // Stack pointer
    Reset_Handler,           // Reset handler
    NMI_Handler,             // Strong version exists
    HardFault_Handler,       // Strong version exists
    MemManage_Handler,       // Weak
    BusFault_Handler,        // Weak
    UsageFault_Handler,      // Weak
};

// Override for memory management
void MemManage_Handler(void) {
    // Custom handler instead of infinite loop
    log_error("MemManage fault at PC=0x%08X", __builtin_return_address(0));
    system_recover();
}

// Conditional override
#ifdef USE_CUSTOM_HANDLER
    void Timer_IRQHandler(void) {
        // Custom implementation
    }
#else
    // Weak default exists in library
#endif

// Weak for test mocking
__attribute__((weak))
uint32_t hardware_read(void) {
    // Real hardware read
    return *((volatile uint32_t*)0x40000000);
}

// Test code (override)
uint32_t hardware_read(void) {
    // Mock value for testing
    return 0x42;
}
```

### 💡 Explanation

**Weak symbol rules:**
1. Multiple weak symbols allowed (linker picks one)
2. One strong symbol overrides all weak
3. No strong symbol → linker picks first weak
4. Useful for library default implementations

### 📋 Interview Tip

```c
// Check if symbol is weak or strong
// arm-none-eabi-nm --defined-only firmware.elf | grep " W "
// 'W' = weak, 'T' = strong

// Override weak with attribute
__attribute__((weak)) void debug_printf(const char *fmt, ...) {
    // Default: no output
}

// Strong override in application
void debug_printf(const char *fmt, ...) {
    // Send to UART
    uart_vprintf(fmt, args);
}
```

### 🎭 Visual Comedy

```
Linker: "I need a symbol 'uart_handler'"
        "Found weak version... but also strong version!"
        "Strong wins! Weak gets discarded"

Weak symbol: "I was here first!"
Strong: "I'm stronger!"
Linker: "Strong wins. Sorry weak."

"Weak symbols: The backup dancers of the linker world!" 💃
```

---

## Q56: Backtrace on ARM Cortex-M

### 📚 Definition

A backtrace reconstructs the call stack after a fault, showing which functions were called leading to the crash. Essential for post-mortem debugging.

### 📝 Syntax

```c
// Stack frame structure
typedef struct {
    uint32_t r4;   // Register 4
    uint32_t r5;   // Register 5
    uint32_t r6;   // Register 6
    uint32_t r7;   // Register 7
    uint32_t r8;   // Register 8
    uint32_t r9;   // Register 9
    uint32_t r10;  // Register 10
    uint32_t r11;  // Register 11
    uint32_t lr;   // Link register (return address)
} stack_frame_t;

// Hard fault handler with backtrace
void HardFault_Handler(void) {
    // Get stack pointer
    uint32_t sp = __get_MSP();
    uint32_t psp = __get_PSP();
    
    // Get fault address
    uint32_t fault_addr = SCB->BFAR;
    uint32_t fault_status = SCB->CFSR;
    
    printf("Hard fault at PC=0x%08X\n", __builtin_return_address(0));
    printf("Fault address: 0x%08X\n", fault_addr);
    printf("Fault status: 0x%08X\n", fault_status);
    
    // Print backtrace
    print_backtrace(sp);
    
    while(1);
}

// Backtrace printer
void print_backtrace(uint32_t current_sp) {
    uint32_t *sp = (uint32_t*)current_sp;
    uint32_t frame_count = 0;
    
    printf("Backtrace:\n");
    
    while(sp && frame_count < 20) {
        // SP points to stacked registers (r0-r3, r12, lr, pc, xpsr)
        uint32_t pc = sp[6];  // Program counter (stacked)
        uint32_t lr = sp[5];  // Link register (return address)
        
        if(pc == 0 || lr == 0) break;
        
        printf("  #%lu: PC=0x%08X (LR=0x%08X)\n", frame_count, pc, lr);
        
        // Find function name (requires symbol table)
        const char *func = find_function_name(pc);
        if(func) printf("       %s\n", func);
        
        // Move to next frame
        sp = (uint32_t*)lr;  // Previous frame
        frame_count++;
    }
}

// Symbol lookup (requires .elf file)
typedef struct {
    uint32_t addr;
    const char *name;
} symbol_t;

extern symbol_t __symbol_start[];
extern symbol_t __symbol_end[];

const char* find_function_name(uint32_t addr) {
    for(symbol_t *s = __symbol_start; s < __symbol_end; s++) {
        if(s->addr == addr) return s->name;
    }
    return NULL;
}

// Automatic backtrace on any fault
void fault_handler_with_backtrace(void) {
    uint32_t sp;
    uint32_t frame_ptr;
    
    // Try both MSP and PSP
    __asm volatile("mov %0, sp" : "=r"(sp));
    __asm volatile("mov %0, r7" : "=r"(frame_ptr));  // Frame pointer
    
    // Walk frames using frame pointer
    while(frame_ptr && frame_count < 20) {
        uint32_t return_addr = *(uint32_t*)(frame_ptr + 4);
        printf("0x%08X\n", return_addr);
        
        // Move to previous frame
        frame_ptr = *(uint32_t*)frame_ptr;
        frame_count++;
    }
}

// Store backtrace in memory for post-mortem
typedef struct {
    uint32_t pc[20];
    uint32_t lr[20];
    uint32_t count;
    uint32_t fault_pc;
    uint32_t fault_lr;
    uint32_t fault_sp;
} crash_dump_t;

crash_dump_t crash_dump;

void save_crash_context(void) {
    crash_dump.fault_pc = __builtin_return_address(0);
    crash_dump.fault_lr = __builtin_return_address(1);
    crash_dump.fault_sp = __get_MSP();
    
    // Walk stack
    uint32_t *sp = (uint32_t*)crash_dump.fault_sp;
    crash_dump.count = 0;
    
    while(sp && crash_dump.count < 20) {
        crash_dump.pc[crash_dump.count] = sp[6];
        crash_dump.lr[crash_dump.count] = sp[5];
        crash_dump.count++;
        sp = (uint32_t*)sp[5];
    }
    
    // Save to backup registers
    for(int i = 0; i < crash_dump.count && i < 10; i++) {
        RTC->BKP0R + i*4 = crash_dump.pc[i];
    }
}

// Python script to decode backtrace (external)
/*
import subprocess

def decode_backtrace(pcs):
    for pc in pcs:
        result = subprocess.run(
            ['arm-none-eabi-addr2line', '-e', 'firmware.elf', hex(pc)],
            capture_output=True, text=True
        )
        print(f"{hex(pc)}: {result.stdout.strip()}")
*/
```

### 💡 Explanation

**Stack frame layout (ARM AAPCS):**
```
Higher address
+------------------+
| Previous frame   |
+------------------+
| r4-r11 (saved)   | ← Frame pointer points here
+------------------+
| Return address   |
+------------------+
| Local variables  |
+------------------+ ← Stack pointer
Lower address
```

### 📋 Interview Tip

```c
// Use frame pointer for reliable backtrace
// Compile with: -fno-omit-frame-pointer
// Without frame pointer, backtrace may fail
```

### 🎭 Visual Comedy

```
Crash: BOOM!
Backtrace:
  #0: main() at main.c:42
  #1: uart_handler() at uart.c:128
  #2: process_buffer() at buffer.c:256
  #3: system_init() at startup.s:15
  #4: Reset_Handler()

Developer: "Aha! Found the bug!"

"Backtrace: The breadcrumb trail back to your bug!" 🍞
```

---

## Q57: Duff's Device (Historical)

### 📚 Definition

Duff's Device is a famous C optimization that combines loop unrolling with switch statement fall-through for copying memory. Educational curiosity in modern embedded systems.

### 📝 Syntax

```c
// Duff's Device - original version
void duff_copy(uint32_t *to, const uint32_t *from, size_t count) {
    size_t n = (count + 7) / 8;  // How many groups of 8
    
    switch(count % 8) {
        case 0: do { *to++ = *from++;
        case 7:      *to++ = *from++;
        case 6:      *to++ = *from++;
        case 5:      *to++ = *from++;
        case 4:      *to++ = *from++;
        case 3:      *to++ = *from++;
        case 2:      *to++ = *from++;
        case 1:      *to++ = *from++;
                } while(--n > 0);
    }
}

// Modern equivalent (better optimized)
void modern_copy(uint32_t *to, const uint32_t *from, size_t count) {
    // Compiler will unroll and optimize
    memcpy(to, from, count * sizeof(uint32_t));
    
    // Or let compiler decide
    #pragma GCC unroll 8
    for(size_t i = 0; i < count; i++) {
        to[i] = from[i];
    }
}

// Why Duff's Device is obsolete:
// 1. Modern compilers do better unrolling
// 2. Cache behavior is more important
// 3. ARM SIMD instructions (NEON) copy 16 bytes at once
// 4. Harder to read/maintain
// 5. May confuse optimizers

// Use modern optimized memcpy
void arm_optimized_copy(uint32_t *to, const uint32_t *from, size_t count) {
    // Use builtin memcpy with optimization
    __builtin_memcpy(to, from, count * 4);
    
    // Or use DMA for large copies
    if(count > 256) {
        dma_copy(to, from, count);
    } else {
        __builtin_memcpy(to, from, count * 4);
    }
}

// Educational: showing the trick
void duff_explanation(void) {
    // The magic: switch falls through into do-while
    // count % 8 determines entry point
    // Then each iteration copies 8 items
    //
    // For count = 10:
    // switch(2) -> entry at case 2
    // Copies 2 items, then loop copies 8 more = 10 total
}

// Modern ARM copy using LDM/STM (load/store multiple)
void asm_copy(uint32_t *to, const uint32_t *from, size_t count) {
    __asm volatile (
        "1:\n"
        "ldm %[from]!, {r4-r7}\n"
        "stm %[to]!, {r4-r7}\n"
        "subs %[count], #4\n"
        "bne 1b\n"
        : [to] "+r"(to), [from] "+r"(from), [count] "+r"(count)
        :
        : "r4", "r5", "r6", "r7", "cc"
    );
}
```

### 💡 Explanation

**Why historical:**
- Written in 1983 for early C compilers
- Modern compilers produce better code
- Cache behavior more important than instruction count
- ARM's LDM/STM instructions make it obsolete

### 📋 Interview Tip

**If asked about Duff's Device:**
1. Explain what it does (unrolled loop)
2. Acknowledge cleverness for its time
3. Explain why you wouldn't use it today
4. Show modern alternatives

### 🎭 Visual Comedy

```
Duff's Device:
      switch/do-while hybrid
            ↓
"What am I looking at?"
            ↓
"This is C code??"

Modern memcpy:
      __builtin_memcpy()
            ↓
"Simple. Clean. Fast."

"Duff's Device: The code that makes you question
everything you know about C programming!" 🤯
```

---

## Q58: March C- Memory Test

### 📚 Definition

March C- is an algorithm for testing RAM that detects stuck-at faults, stuck-open faults, and coupling faults with predictable time complexity.

### 📝 Syntax

```c
// March C- memory test algorithm
typedef enum {
    TEST_PASS,
    TEST_FAIL
} test_result_t;

// March elements
#define MARCH_UP    0  // Increasing addresses
#define MARCH_DOWN  1  // Decreasing addresses

test_result_t march_c_test(volatile uint32_t *start, uint32_t size_words) {
    uint32_t *mem = (uint32_t*)start;
    uint32_t i;
    
    // March element 1: Write all 0s (addresses increasing)
    for(i = 0; i < size_words; i++) {
        mem[i] = 0;
    }
    
    // March element 2: Read 0, write 1 (increasing)
    for(i = 0; i < size_words; i++) {
        if(mem[i] != 0) return TEST_FAIL;  // Read 0
        mem[i] = 1;                        // Write 1
    }
    
    // March element 3: Read 1, write 0 (increasing)
    for(i = 0; i < size_words; i++) {
        if(mem[i] != 1) return TEST_FAIL;  // Read 1
        mem[i] = 0;                        // Write 0
    }
    
    // March element 4: Read 0, write 1 (decreasing)
    for(i = size_words; i > 0; i--) {
        if(mem[i-1] != 0) return TEST_FAIL;  // Read 0
        mem[i-1] = 1;                        // Write 1
    }
    
    // March element 5: Read 1, write 0 (decreasing)
    for(i = size_words; i > 0; i--) {
        if(mem[i-1] != 1) return TEST_FAIL;  // Read 1
        mem[i-1] = 0;                        // Write 0
    }
    
    // March element 6: Read 0 (increasing)
    for(i = 0; i < size_words; i++) {
        if(mem[i] != 0) return TEST_FAIL;   // Read 0
    }
    
    return TEST_PASS;
}

// Optimized March C- with background operation
typedef struct {
    volatile uint32_t progress;
    volatile uint32_t fails;
    volatile uint8_t running;
} test_state_t;

void march_c_test_background(volatile uint32_t *start, uint32_t size_words) {
    uint32_t *mem = (uint32_t*)start;
    uint32_t i;
    
    test_state.running = 1;
    
    // March 1: Write 0
    for(i = 0; i < size_words && test_state.running; i++) {
        mem[i] = 0;
        test_state.progress = i * 100 / size_words;
    }
    if(!test_state.running) return;
    
    // March 2: Read 0, write 1
    for(i = 0; i < size_words && test_state.running; i++) {
        if(mem[i] != 0) test_state.fails++;
        mem[i] = 1;
        test_state.progress = 25 + (i * 25 / size_words);
    }
    // ... continue for other marches
    
    test_state.running = 0;
}

// Memory test with error logging
typedef struct {
    uint32_t address;
    uint32_t expected;
    uint32_t actual;
    uint8_t operation;
} memory_error_t;

memory_error_t errors[256];
uint32_t error_count = 0;

void log_error(uint32_t addr, uint32_t expected, uint32_t actual, uint8_t op) {
    if(error_count < 256) {
        errors[error_count].address = addr;
        errors[error_count].expected = expected;
        errors[error_count].actual = actual;
        errors[error_count].operation = op;
        error_count++;
    }
}

// Self-test on startup
void ram_self_test(void) {
    // Test only critical sections first
    if(march_c_test((uint32_t*)0x20000000, 1024) != TEST_PASS) {
        // RAM failure - can't continue
        blink_error_code(ERROR_RAM);
        while(1);
    }
    
    // Test remaining RAM in background
    march_c_test_background((uint32_t*)0x20000400, 1024*64);
}

// Periodic memory test (safety-critical)
void periodic_memory_test(void) {
    static uint32_t test_patterns[] = {0x55555555, 0xAAAAAAAA, 0xFFFFFFFF, 0x00000000};
    static uint32_t current_pattern = 0;
    
    // Test a small block each iteration
    uint32_t *test_block = (uint32_t*)0x20001000;
    
    // Write pattern
    for(int i = 0; i < 64; i++) {
        test_block[i] = test_patterns[current_pattern];
    }
    
    // Read back
    for(int i = 0; i < 64; i++) {
        if(test_block[i] != test_patterns[current_pattern]) {
            // Memory degradation detected!
            safe_shutdown(ERROR_MEMORY_CORRUPT);
        }
    }
    
    current_pattern = (current_pattern + 1) % 4;
}
```

### 💡 Explanation

**March C- complexity:**
- Time: O(n) - 6n operations (good)
- Fault coverage: Stuck-at, coupling, stuck-open
- 6 March elements (write/read/read+write)

### 📋 Interview Tip

```c
// Validate before and after (endianness might affect test patterns)
static_assert(sizeof(uint32_t) == 4, "32-bit words required");
static_assert(UINT32_MAX == 0xFFFFFFFF, "2's complement required");
```

### 🎭 Visual Comedy

```
RAM cells before test: [????]
March 1: Write 0 → [0000]
March 2: Read 0, Write 1 → [1111]
March 3: Read 1, Write 0 → [0000]
March 4: Read 0, Write 1 → [1111]
March 5: Read 1, Write 0 → [0000]
March 6: Read 0 → [0000] ✓

Broken cell: [???1] fails at March 2
"The March test marches through memory like a drill sergeant!" 🪖
```

---

## Q59: setjmp/longjmp in Embedded

### 📚 Definition

`setjmp` saves the execution context (stack pointer, program counter), and `longjmp` restores it, allowing non-local jumps (like exceptions) in C.

### 📝 Syntax

```c
#include <setjmp.h>

// Error handling without exceptions
jmp_buf error_handler;

int risky_operation(void) {
    if(setjmp(error_handler) == 0) {
        // First time - normal execution
        dangerous_code();
        return 0;
    } else {
        // Returning from longjmp - error path
        handle_error();
        return -1;
    }
}

void dangerous_code(void) {
    // Something goes wrong
    longjmp(error_handler, 1);  // Jump back, return value 1
}

// Nested error handling
jmp_buf top_level;
jmp_buf mid_level;

void deep_function(void) {
    if(setjmp(mid_level) == 0) {
        // Normal
        deeper_function();
    } else {
        // Handle mid-level error
        longjmp(top_level, 2);
    }
}

void reset_state(void) {
    // Restore registers, stack, etc.
    // Not recommended in embedded!
}

// Reset-proof error recovery (store jmp_buf in noinit section)
jmp_buf __attribute__((section(".noinit"))) reset_jmp;

void check_for_reset(void) {
    if(get_reset_cause() == RESET_WATCHDOG) {
        // Recover after watchdog reset
        longjmp(reset_jmp, 1);
    }
}

// Alternative: never use setjmp/longjmp in embedded
// Why?
// 1. Undefined behavior with volatile
// 2. Doesn't restore signal mask
// 3. Can corrupt floating point state
// 4. Hard to debug
// 5. Better patterns exist

// Better: State machine
typedef enum {
    STATE_NORMAL,
    STATE_ERROR,
    STATE_RECOVER
} error_state_t;

error_state_t g_error_state = STATE_NORMAL;

int safe_operation(void) {
    if(g_error_state != STATE_NORMAL) {
        return -1;
    }
    
    if(error_condition()) {
        g_error_state = STATE_ERROR;
        return -1;
    }
    
    return 0;
}

void error_recovery(void) {
    g_error_state = STATE_RECOVER;
    // Clean up resources
    clean_state();
    g_error_state = STATE_NORMAL;
}

// Stack unwinding manually
#define TRY do { jmp_buf _buf; if(setjmp(_buf) == 0) {
#define CATCH } else {
#define END_TRY }} while(0)
#define THROW longjmp(_buf, 1)

TRY {
    // Code that might fail
    if(error) THROW;
}
CATCH {
    // Handle error
    printf("Caught!\n");
}
END_TRY;
```

### 💡 Explanation

**Why avoid in embedded:**
- Stack unwinding unpredictable
- Can corrupt RTOS task contexts
- Doesn't restore interrupt state
- Debugging nightmare

### 📋 Interview Tip

**Better alternatives:**
- State machines
- Error return codes
- RTOS task notifications
- Watchdog + recovery

### 🎭 Visual Comedy

```
setjmp: "I'll save your place!"
longjmp: "Go back to saved place!"
Program: *jumps*
Debugger: "Where did you go?"

Without setjmp:
Error: "Fix it properly"
Program: *handles error* "Done"

"setjmp/longjmp: The 'teleport' of C programming.
Amazing, until something goes wrong!" 🌀
```

---

## Q60: Therac-25 Race Condition Bug

### 📚 Definition

The Therac-25 radiation therapy machine had a race condition that allowed high-power mode without proper safety checks, leading to patient deaths. Classic embedded safety lesson.

### 📝 Syntax - The Bug

```c
// Simplified Therac-25 bug example
typedef struct {
    uint32_t mode;        // 0=low, 1=high, 2=low (error)
    uint32_t beam_power;
    uint8_t setup_complete;
} therapy_control_t;

therapy_control_t therac;

// The buggy code (simplified)
void set_xray_mode(void) {
    // Set to low power mode first (should be safe)
    therac.mode = 0;
    
    // Operator could edit data during this window!
    therac.beam_power = 0;  // Safe
    
    // Check if setup complete
    if(operator_typing_complete()) {
        // Later: set to high power
        therac.mode = 1;
        therac.beam_power = 25000;  // High power!
    }
}

// The race condition:
// Thread 1 (main):
void operator_input(void) {
    while(1) {
        if(key_pressed()) {
            update_display();
            // Between these, settings could be partially applied
            
            therac.setup_complete = 0;  // Not ready
            // ... update parameters ...
            therac.setup_complete = 1;  // Ready
        }
    }
}

// Thread 2 (beam control):
void beam_control(void) {
    while(1) {
        if(therac.setup_complete) {
            // Problem: Reading partially updated data!
            therac.beam_power = calculate_power();
            fire_beam();  // May fire with wrong power!
        }
    }
}

// Fixed version - atomic updates
typedef struct {
    uint32_t mode;
    uint32_t beam_power;
    uint8_t setup_complete;
    uint8_t validation_key;  // Extra safety
} safe_therac_t;

void safe_set_xray_mode(void) {
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    // Atomic update
    therac.mode = 0;
    therac.beam_power = 0;
    therac.validation_key = MAGIC;
    therac.setup_complete = 1;
    
    __set_PRIMASK(primask);
}

void safe_beam_control(void) {
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    uint32_t power = therac.beam_power;
    uint32_t mode = therac.mode;
    uint8_t valid = (therac.validation_key == MAGIC);
    
    __set_PRIMASK(primask);
    
    if(valid && mode == 1) {
        // Only fire if data is consistent
        if(power <= SAFE_LIMIT) {
            fire_beam();
        } else {
            safety_shutdown();
        }
    }
}

// Modern safety patterns
typedef struct {
    uint32_t value;
    uint32_t checksum;
} safe_variable_t;

void safe_write(safe_variable_t *var, uint32_t value) {
    var->value = value;
    var->checksum = ~value;  // Simple checksum
}

uint32_t safe_read(safe_variable_t *var) {
    if(var->checksum != ~var->value) {
        // Corrupted!
        system_error(ERROR_CORRUPTION);
        return 0;
    }
    return var->value;
}

// Double-check (vital for safety)
void fire_beam_safe(void) {
    // Double-check conditions
    if(beam_mode != HIGH_POWER) return;
    if(beam_power > MAX_SAFE) return;
    if(!patient_in_position) return;
    
    // Second confirmation from operator
    if(!confirm_fire()) return;
    
    // All checks passed
    actually_fire_beam();
}
```

### 💡 Explanation

**Therac-25 lessons:**
1. Race conditions kill
2. Single-threaded assumed, but interrupts exist
3. Always use atomic operations
4. Validate data before critical actions
5. Double-check safety conditions
6. Design for failure

### 📋 Interview Tip

**Safety-critical patterns:**
- Always use critical sections
- Validate all inputs
- Redundant checks
- Watchdog timers
- Fail-safe defaults

### 🎭 Visual Comedy

```
Buggy code: 
Thread 1: "Settings: half updated"
Thread 2: "Settings ready! Fire!"
Patient: *receives lethal dose*

Fixed code:
Thread 1: "Acquire lock"
Thread 2: "Wait for lock..."
Thread 1: "Settings fully updated"
Thread 2: "Verify checksum... OK"
Patient: "Safe treatment"

"Therac-25: When software bugs become literal life-or-death.
Always lock your shared data!" 💀
```

---

## Final Quick Reference Card 5

### Debug & Safety Patterns

| Pattern | Use | Overhead |
|---------|-----|----------|
| Assert | Development | None (release) |
| Watchpoint | Debug | Zero |
| Stack canary | Protection | Low |
| Weak symbols | Libraries | Zero |
| Backtrace | Debug | Zero |
| March C- | RAM test | 6n operations |

### Safety Critical Rules

```c
// 1. Always validate
if(ptr != NULL && ptr != INVALID_ADDRESS) {
    *ptr = value;
}

// 2. Use atomic operations
__disable_irq();
critical_operation();
__enable_irq();

// 3. Redundant checks
if(check1 && check2 && check3) {
    unsafe_operation();
}

// 4. Fail safe
if(error) {
    enter_safe_state();
    while(1);  // Watchdog resets
}

// 5. Log errors
log_error(ERROR_CODE);
save_crash_dump();
```

### Memory Test Patterns

| Pattern | Use | Value |
|---------|-----|-------|
| Walking 1 | Stuck-at | 0x00000001, 0x00000002... |
| Walking 0 | Stuck-at | 0xFFFFFFFE, 0xFFFFFFFD... |
| Checkerboard | Coupling | 0x55555555, 0xAAAAAAAA |
| March C- | All faults | Algorithmic |

---

## The End (And Beginning)

Congratulations! You've completed the Embedded C Interview Questions series.

```c
// Final advice
int main(void) {
    init_hardware();
    init_interrupts();
    
    while(1) {
        study_daily(this_repository);
        practice_coding();
        apply_for_jobs();
        
        if(interview_successful()) {
            celebrate();
            break;
        } else {
            learn_from_mistakes();
            continue_learning();
        }
    }
    
    return 0;  // Success!
}
```


[← Back to Main README](README.md)

---
