
# Embedded C Interview Questions - Patch 1 (Continuation)

**Continuation from main README.md - Questions 25-40**

[← Back to Main README](README.md)

---

## Q25: Critical Sections with Interrupt Disable

###  Definition

A critical section is a block of code that must execute atomically without interruption. On bare-metal systems, this means disabling interrupts temporarily.

###  Syntax

```c
// Simple critical section (bare metal)
void critical_section_example(void) {
    // Save current interrupt state
    uint32_t primask = __get_PRIMASK();
    
    // Disable interrupts (set PRIMASK)
    __disable_irq();
    
    // === CRITICAL SECTION START ===
    shared_variable++;
    if (shared_variable == 100) {
        trigger_event();
    }
    // === CRITICAL SECTION END ===
    
    // Restore previous interrupt state
    __set_PRIMASK(primask);
}

// Macro version for cleaner code
#define CRITICAL_SECTION_START() \
    do { \
        uint32_t _primask = __get_PRIMASK(); \
        __disable_irq(); \
        do {} while(0)

#define CRITICAL_SECTION_END() \
        do {} while(0); \
        __set_PRIMASK(_primask); \
    } while(0)

// Usage
void example(void) {
    CRITICAL_SECTION_START();
    // Critical code here
    CRITICAL_SECTION_END();
}
```

###  Explanation

**Why not just `__disable_irq()` / `__enable_irq()`?**
- If interrupts were already disabled, `__enable_irq()` would wrongly re-enable them
- Always save/restore state, never force-enable

**On ARM Cortex-M:**
- `__disable_irq()` sets PRIMASK (disables all exceptions except HardFault and NMI)
- `__set_PRIMASK(primask)` restores previous state

### 📋 Interview Tip

```c
// WRONG - destroys previous state
__disable_irq();
// critical code
__enable_irq();  // Forces interrupts ON even if they were OFF!

// CORRECT - restores previous state
uint32_t state = __get_PRIMASK();
__disable_irq();
// critical code
__set_PRIMASK(state);
```

### 🎭 Visual Comedy

```
Interrupts:    [ENABLED]  →  [DISABLED]  →  [RESTORED]
                    ↓            ↓              ↓
Main:           "I'm running"  "Critical!"   "Done"
ISR:            "Can I run?"   "No!"         "Thanks!"

"Critical sections: The only time your ISRs
get put on hold. They'll call back later!" 📞
```

---

## Q26: Priority Inversion & Inheritance

### 📚 Definition

Priority inversion occurs when a high-priority task is blocked waiting for a resource held by a low-priority task, while a medium-priority task preempts the low-priority task, causing the high-priority task to wait indefinitely.

### 📝 Syntax

```c
// FreeRTOS mutex with priority inheritance
SemaphoreHandle_t mutex;

void low_priority_task(void *pvParams) {
    while(1) {
        xSemaphoreTake(mutex, portMAX_DELAY);  // Takes mutex
        // Long operation with mutex held
        vTaskDelay(pdMS_TO_TICKS(100));
        xSemaphoreGive(mutex);
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

void medium_priority_task(void *pvParams) {
    while(1) {
        // CPU-bound task that preempts low priority
        for(int i = 0; i < 100000; i++) {
            // Computations
        }
        vTaskDelay(pdMS_TO_TICKS(5));
    }
}

void high_priority_task(void *pvParams) {
    while(1) {
        xSemaphoreTake(mutex, portMAX_DELAY);  // Waits for low task!
        // Critical operation
        xSemaphoreGive(mutex);
    }
}

// Solution: Use mutex with priority inheritance
// In FreeRTOS config file:
#define configUSE_MUTEXES 1
#define configUSE_PRIORITY_INHERITANCE 1

// Create mutex:
mutex = xSemaphoreCreateMutex();  // Automatically has inheritance
```

### 💡 Explanation

**Without inheritance:**
1. Low task takes mutex
2. Medium task preempts low task (higher priority)
3. High task tries to take mutex → blocked forever
4. Medium task keeps running → **PRIORITY INVERSION**

**With inheritance:**
- When high task waits, low task temporarily inherits high priority
- Low task runs faster (can't be preempted by medium)
- Low task releases mutex, priority restored

### 📋 Interview Tip

**Priority inversion in real life (Mars Pathfinder - 1997):**
- Had a priority inversion bug that caused system resets
- Fixed by adding priority inheritance mutex

### 🎭 Visual Comedy

```
Without Inheritance:
HIGH:    "I need mutex!" → [BLOCKED] → [STILL BLOCKED] → "I'm starving!"
MEDIUM:  "I'm running"  → "Still running" → "More running" → "Done"
LOW:     [Has mutex]    → [PREEMPTED]  → [PREEMPTED]   → "Finally!"

With Inheritance:
HIGH:    "Need mutex" → [BLOCKED] → "Got it!"
MEDIUM:  "Want CPU"    → [BLOCKED by inherited priority] → "Fine..."
LOW:     [Has mutex]   → "I'm high priority now!" → "Released"

"Mutex inheritance: Giving the little guy a temporary promotion!" 🚀
```

---

## Q27: Watchdog Feeder Anti-Spoofing

### 📚 Definition

Anti-spoofing prevents a stuck task from feeding the watchdog while the rest of the system is hung. Instead of a simple "kick" function, use a counter that must be updated by multiple independent tasks.

### 📝 Syntax

```c
// Anti-spoofing watchdog implementation
typedef struct {
    volatile uint32_t task1_counter;
    volatile uint32_t task2_counter;
    volatile uint32_t task3_counter;
    uint32_t last_known_good[3];
} watchdog_state_t;

static watchdog_state_t wdt = {0};

// Each task calls its own feed function
void task1_watchdog_feed(void) {
    wdt.task1_counter++;
}

void task2_watchdog_feed(void) {
    wdt.task2_counter++;
}

void task3_watchdog_feed(void) {
    wdt.task3_counter++;
}

// Main watchdog monitor task (runs every 100ms)
void watchdog_monitor_task(void) {
    static uint32_t last_task1 = 0, last_task2 = 0, last_task3 = 0;
    
    // Check if ALL tasks have progressed
    if (wdt.task1_counter == last_task1 ||
        wdt.task2_counter == last_task2 ||
        wdt.task3_counter == last_task3) {
        // At least one task is stuck!
        system_reset();  // Or enter safe state
    }
    
    // Update last known values
    last_task1 = wdt.task1_counter;
    last_task2 = wdt.task2_counter;
    last_task3 = wdt.task3_counter;
    
    // Feed the actual hardware watchdog
    IWDG->KR = 0xAAAA;  // Reload watchdog (STM32)
}

// Even better: Use random nonce
typedef struct {
    uint32_t nonce;
    uint32_t expected_response;
} watchdog_challenge_t;

void watchdog_challenge(void) {
    wdt.nonce = rand();
    wdt.expected_response = calculate_response(wdt.nonce);
}

void task_feed_response(uint32_t response) {
    if (response == wdt.expected_response) {
        IWDG->KR = 0xAAAA;  // Valid response - feed watchdog
    }
    // Generate new challenge for next cycle
    watchdog_challenge();
}
```

### 💡 Explanation

**Simple approach (spoofable):**
```c
// Any stuck task can still call this:
void feed_watchdog(void) {
    IWDG->KR = 0xAAAA;  // Just feeds - no verification!
}
```

**Anti-spoofing approach:**
- Multiple independent tasks must all report progress
- Or use challenge-response (task must compute correct value)
- Watchdog only feeds if ALL system components are healthy

### 📋 Interview Tip

**In safety-critical systems (ISO 26262):**
- Watchdog must monitor program sequence, not just timing
- Use "question-answer" mechanism
- Store expected answers in ROM, compute on the fly

### 🎭 Visual Comedy

```
Simple Watchdog:
Task: "I'm fine!" *is actually on fire*
Watchdog: "OK, I'll stay alive" 🔥

Anti-spoofing:
Task 1: "I did work" ✓
Task 2: "I did work" ✓  
Task 3: ... [silence]
Watchdog: "Task 3 is dead! RESET!" 🚨

"Your watchdog should be suspicious of every task.
Trust no one, verify everyone!" 🕵️
```

---

## Q28: __disable_irq() vs taskENTER_CRITICAL()

### 📚 Definition

- `__disable_irq()`: Disables ALL interrupts globally (bare metal)
- `taskENTER_CRITICAL()`: RTOS-aware critical section that only disables scheduler (not necessarily all interrupts)

### 📝 Syntax

```c
// Bare metal / simple RTOS
void bare_metal_critical(void) {
    __disable_irq();
    // Access shared resource
    __enable_irq();
}

// FreeRTOS critical section
void rtos_critical(void) {
    taskENTER_CRITICAL();
    // Access shared resource
    // Interrupts above configMAX_SYSCALL_INTERRUPT_PRIORITY still run!
    taskEXIT_CRITICAL();
}

// Nested critical sections (FreeRTOS)
void nested_critical(void) {
    taskENTER_CRITICAL();  // Nested count = 1
    taskENTER_CRITICAL();  // Nested count = 2
    // Still in critical section
    taskEXIT_CRITICAL();   // Nested count = 1
    taskEXIT_CRITICAL();   // Nested count = 0, re-enable
}

// Choosing which interrupts to disable
#define configMAX_SYSCALL_INTERRUPT_PRIORITY 5

// Interrupt with priority 3 (higher than 5) - will still run!
void HighPriority_IRQ(void) {
    // Can't call FreeRTOS functions!
}

// Interrupt with priority 6 (lower than 5) - disabled in critical section
void LowPriority_IRQ(void) {
    // Can call FreeRTOS functions from ISR
    xSemaphoreGiveFromISR(...);
}
```

### 💡 Explanation

| Feature | `__disable_irq()` | `taskENTER_CRITICAL()` |
|---------|-------------------|------------------------|
| Disables interrupts | ALL (PRIMASK) | Only those below threshold |
| RTOS scheduler | Stops (if timer interrupts disabled) | Explicitly suspended |
| Nesting support | Manual | Automatic |
| ISRs can run | None | High-priority only |
| Use case | Bare metal, short sections | RTOS, medium sections |

### 📋 Interview Tip

**In FreeRTOS:**
- Use `taskENTER_CRITICAL()` for protecting shared data between tasks
- Use `__disable_irq()` only for hardware-critical operations (e.g., flash programming)
- Never call `__disable_irq()` in an RTOS task without restoring before blocking

### 🎭 Visual Comedy

```
__disable_irq():
    "ALL INTERRUPTS STOP NOW!"
    [Complete silence]
    "You may continue..."
    "That was dramatic!"

taskENTER_CRITICAL():
    "Scheduler, please pause"
    "But let high-priority hardware through"
    "Thanks, resume when done"

"One is a sledgehammer, one is a scalpel.
Choose wisely for your surgery!" 🔨🔪
```

---

## Q29: Software Interrupt (PendSV) Implementation

### 📚 Definition

PendSV (Pended Service Call) is a software-triggerable interrupt on ARM Cortex-M used for RTOS context switching. It's configurable to pend (trigger) when no other interrupts are active.

### 📝 Syntax

```c
// Trigger PendSV from software
void trigger_context_switch(void) {
    // Set PendSV pending bit
    SCB->ICSR |= SCB_ICSR_PENDSVSET_Msk;
}

// PendSV handler (lowest priority)
void PendSV_Handler(void) {
    // Save current task context
    __asm volatile (
        "mrs r0, psp\n"          // Get current PSP
        "stmdb r0!, {r4-r11}\n"  // Save remaining registers
        "msr psp, r0\n"
        "isb\n"
    );
    
    // Switch to next task
    current_task = next_task;
    
    // Restore new task context
    __asm volatile (
        "mrs r0, psp\n"
        "ldmia r0!, {r4-r11}\n"
        "msr psp, r0\n"
        "isb\n"
        "bx lr\n"
    );
}

// Usage: Yield CPU to next task
void taskYIELD(void) {
    trigger_context_switch();
    __asm volatile("dsb\n isb\n");
}

// Set PendSV to lowest priority
void init_pendsv(void) {
    NVIC_SetPriority(PendSV_IRQn, 0xFF);  // Lowest priority
}
```

### 💡 Explanation

Why PendSV for context switching?
- Can be pended (triggered) at any time
- Executes only when no other interrupts are active
- Priority can be set to lowest (so background tasks don't block ISRs)
- Hardware automatically saves half the context (r0-r3, r12, LR, PC, xPSR)

### 📋 Interview Tip

**PendSV workflow in FreeRTOS:**
1. Task calls `taskYIELD()`
2. PendSV is pended (set bit in ICSR)
3. Current instruction completes
4. Any higher priority interrupts run first
5. When idle, PendSV runs and switches context

### 🎭 Visual Comedy

```
Task A: "I yield!" (Pends PendSV)
  ↓
ISR: "Emergency! Handle me first!" (Higher priority)
  ↓
ISR finishes
  ↓
PendSV: "Finally, my turn. Context switch time!"
  ↓
Task B: "Hello world!"

"PendSV: The polite interrupt that waits its turn.
Unlike that pushy UART interrupt!" 🎩
```

---

## Q30: Interrupt Nesting Stack Calculation

### 📚 Definition

Calculating the maximum stack usage when interrupts can nest (higher priority interrupts preempt lower ones). Each nested interrupt adds its own stack frame.

### 📝 Syntax

```c
// Calculate worst-case stack usage
typedef struct {
    uint32_t main_stack;      // Main program stack
    uint32_t isr1_stack;      // Lowest priority ISR
    uint32_t isr2_stack;      // Medium priority ISR
    uint32_t isr3_stack;      // Highest priority ISR
} stack_calc_t;

// Function to analyze stack
void calculate_max_stack(void) {
    // Assumptions:
    // - ISR3 (highest) can preempt ISR2 and ISR1
    // - ISR2 can preempt ISR1
    // - Main can be preempted by any ISR
    
    uint32_t worst_case = 
        main_stack_max +
        isr1_stack_max +
        isr2_stack_max +
        isr3_stack_max;
    
    printf("Worst case stack: %u bytes\n", worst_case);
    
    // Add 64 bytes for safety margin
    uint32_t total_stack = worst_case + 64;
}

// Runtime stack monitoring
void check_stack_overflow(void) {
    extern uint32_t __StackTop;
    extern uint32_t __StackLimit;
    
    uint32_t current_sp = __get_MSP();
    
    if (current_sp < (uint32_t)&__StackLimit) {
        // Stack overflow!
        system_error(ERROR_STACK_OVERFLOW);
    }
}

// GCC attribute to set stack canary
uint32_t __attribute__((section(".stack_canary"))) stack_canary = 0xDEADBEEF;

void check_canary(void) {
    if (stack_canary != 0xDEADBEEF) {
        // Stack corruption detected
        enter_safe_state();
    }
}
```

### 💡 Explanation

**Stack frame sizes (ARM Cortex-M):**
- Main program: Local vars + return address (8-100+ bytes)
- ISR prologue: 8 registers saved automatically (32 bytes)
- ISR local vars: Depends on function
- Nested ISR: Another full frame

**Example calculation:**
```
Main stack:     256 bytes
ISR1 (UART):    64 bytes
ISR2 (Timer):   96 bytes (calls functions)
ISR3 (DMA):     48 bytes
--------------------------------
Worst case:     256 + 64 + 96 + 48 = 464 bytes
```

### 📋 Interview Tip

**Tools to measure:**
- `--fstack-usage` (GCC): Reports function stack usage
- `arm-none-eabi-size -A firmware.elf`: Shows section sizes
- Static analysis: Add 50% safety margin

### 🎭 Visual Comedy

```
Stack growth with nesting:
Address    Content
0x20001FFF ┌─────────────┐
           │ Main stack  │ ← 256 bytes
0x20001EFF ├─────────────┤
           │ ISR1 frame  │ ← 64 bytes (preempted)
0x20001EBF ├─────────────┤
           │ ISR2 frame  │ ← 96 bytes (preempted)
0x20001E5F ├─────────────┤
           │ ISR3 frame  │ ← 48 bytes (running)
0x20001E2F └─────────────┘
           ↑
        Stack pointer

"Watch your stack like a hawk.
One wrong move and *crash*!" 🦅
```

---

## Q31: Deferred Handling with Task Notifications

### 📚 Definition

Task notifications are a lightweight (faster than semaphores) mechanism in FreeRTOS for tasks to send events to each other without creating a separate queue or semaphore object.

### 📝 Syntax

```c
// Receiver task - waits for notification
void processing_task(void *pvParams) {
    uint32_t notification_value;
    
    while(1) {
        // Wait for notification with timeout
        if (xTaskNotifyWait(0x00,           // Clear bits on entry
                            ULONG_MAX,      // Clear all bits on exit
                            &notification_value,
                            portMAX_DELAY)) {
            // Process based on notification value
            switch(notification_value) {
                case 1: handle_uart_data(); break;
                case 2: handle_timer_expiry(); break;
                case 3: handle_button_press(); break;
            }
        }
    }
}

// ISR sends notification (deferred handling)
void UART_ISR(void) {
    uint32_t data = UART->DR;
    
    // Store data in global buffer
    uart_rx_buffer[uart_index++] = data;
    
    // Notify task (instead of processing directly)
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    vTaskNotifyGiveFromISR(processing_task_handle,
                           &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// Another way: Send value
void Timer_ISR(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    xTaskNotifyFromISR(timer_task_handle,
                      2,                    // Notification value
                      eSetBits,             // Set bits (OR)
                      &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// Simple notification (binary semaphore style)
void Button_ISR(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // Give notification (increments counter)
    vTaskNotifyGiveFromISR(button_task_handle,
                          &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

void button_task(void *pvParams) {
    while(1) {
        // Wait for notification (decrements counter)
        ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        handle_button();
    }
}
```

### 💡 Explanation

**Benefits over semaphores:**
- No separate object needed (uses task's own notification)
- 45% faster than binary semaphore
- Uses less RAM (8 bytes vs 20+ bytes)
- Can send 32-bit values

**Comparison:**
| Feature | Semaphore | Task Notification |
|---------|-----------|-------------------|
| Object needed | Yes | No (per task) |
| Max count | Configurable | 255 (or unlimited) |
| Value passing | No | Yes (32-bit) |
| Multiple waiters | Yes | No (one per task) |

### 📋 Interview Tip

**Limitations:**
- Only one task can wait on notifications
- Notifications can be lost if task isn't waiting (use queue if needed)
- Not available in all RTOSes (FreeRTOS-specific)

### 🎭 Visual Comedy

```
ISR: "Task! Wake up! I have data!"
      ↓ (notification)
Task: "Zzz... Oh! What is it?"
      ↓ (value 42)
Task: "42? That's the answer!"

"Task notifications: Like poking your friend
instead of writing a formal letter. Much faster!" 📨
```

---

## Q32: NMI (Non-Maskable Interrupt) Uses

### 📚 Definition

NMI is the highest priority interrupt that cannot be disabled (`__disable_irq()` has no effect). Used for hardware-critical events that must be handled immediately.

### 📝 Syntax

```c
// NMI handler (cannot be masked)
void NMI_Handler(void) {
    // Check what caused NMI
    if (RCC->CSR & RCC_CSR_LPWRRSTF) {
        // Low-power reset - recover state
        recover_from_sleep();
    }
    
    if (BKP->DR1 != MAGIC_NUMBER) {
        // Backup RAM corrupted
        restore_backup();
    }
    
    // Hardware failure - attempt safe shutdown
    if (PWR->CSR & PWR_CSR_SBUF) {
        // Standby flag - power issue
        emergency_shutdown();
    }
    
    // Log error and reset
    log_error(ERROR_NMI);
    NVIC_SystemReset();
}

// Software trigger NMI (on some MCUs)
void trigger_nmi(void) {
    // STM32: Write to NMI control register
    DBGMCU->CR |= DBGMCU_CR_TRACE_IOEN;
    // Or cause hardware fault that escalates to NMI
    *(uint32_t*)0xE000EDF0 = 0x05FA0004;  // AIRCR VECTKEY
}

// Example: Watchdog NMI before reset
void IWDG_IRQHandler(void) {
    // This runs BEFORE watchdog reset
    if (IWDG->SR & IWDG_SR_PVU) {
        // Save critical data to backup registers
        BKP->DR1 = system_state;
        BKP->DR2 = error_code;
        
        // Flash LED pattern for debugging
        for(int i = 0; i < error_code; i++) {
            LED_On();
            delay_ms(100);
            LED_Off();
            delay_ms(100);
        }
    }
    // Then reset
}
```

### 💡 Explanation

**Common NMI sources:**
- Hardware watchdog expiration (pre-reset)
- Clock security system (CSS) - crystal failure
- Power supply brown-out detection
- Memory parity error
- Bus fault escalation (configurable)

### 📋 Interview Tip

**NMI vs HardFault:**
- **NMI**: External hardware event, always highest priority
- **HardFault**: Internal CPU error (divide by zero, unaligned access, etc.)
- Both are non-maskable, but caused by different sources

### 🎭 Visual Comedy

```
Other Interrupts: "We can be disabled!"
NMI: "DISABLE ME? I DON'T KNOW HER"
      ↓
Developer: "__disable_irq()"
NMI: "That's cute. Anyway..."
      ↓
Hardware: "Crystal failed!"
NMI: "I GOT THIS. Emergency shutdown!"

"NMI: The interrupt that doesn't take 'no' for an answer.
Like a fire alarm - you WILL handle it!" 🚨
```

---

## Q33: Link-Time Optimization (LTO)

### 📚 Definition

LTO (Link-Time Optimization) allows the compiler to optimize across file boundaries by postponing optimizations until link time, potentially inlining functions from different .c files.

### 📝 Syntax

```makefile
# In Makefile
CFLAGS += -flto -Os
LDFLAGS += -flto

# Or for specific files
file1.o: CFLAGS += -flto
file2.o: CFLAGS += -flto

# Check LTO status in output
# arm-none-eabi-objdump -d firmware.elf | less

// Without LTO - function defined in another file
// uart.c
void uart_send_byte(uint8_t c) {
    while(!(UART->SR & UART_SR_TXE));
    UART->DR = c;
}

// main.c
void send_message(void) {
    uart_send_byte('H');  // Can't inline across files without LTO
    uart_send_byte('i');
}

// With LTO, compiler can inline uart_send_byte into main
// Result: Smaller and faster code
```

### 💡 Explanation

**What LTO enables:**
- Cross-file inlining
- Dead code elimination across files
- Constant propagation across compilation units
- Whole-program optimization

**Trade-offs:**
- Increases link time significantly (2-10x slower)
- Memory usage during link (RAM for whole program IR)
- May increase binary size for some code patterns
- Debugging harder (function names may disappear)

###  Interview Tip

**When to use LTO:**
- ✅ Production builds for size/speed
- ✅ Release builds
- ❌ Debug builds (slows compilation, hurts debugging)
- ❌ When using weak symbols (unpredictable)

### 🎭 Visual Comedy

```
Without LTO:
main.c: "I call uart_send_byte()"
uart.c: "Here's uart_send_byte()"
Linker: "Connect them"
Code: [Call] → [Jump] → [Return]

With LTO:
Compiler: "I see main.c calls uart_send_byte()"
Compiler: "I know what uart_send_byte() does"
Compiler: "I'll just put the code directly here"
Code: [uart_send_byte instructions inline]

"LTO: Breaking down file barriers since 2005.
Code unification through compiler communism!" 
```

---

## Quick Reference Card 2

### Critical Section Patterns

| Pattern | Use Case | Performance |
|---------|----------|-------------|
| `__disable_irq()` | Bare metal, short ops | Fastest |
| `taskENTER_CRITICAL()` | RTOS task protection | Medium |
| Mutex | Long operations | Slowest |
| Lock-free | High-performance | Complex |

### Watchdog Patterns

| Pattern | Security Level | Complexity |
|---------|---------------|------------|
| Simple kick | Low | Simple |
| Multi-task counters | Medium | Medium |
| Challenge-response | High | High |
| External watchdog | Hardware | Simple |

### ISR Latency Optimization

| Technique | Latency Reduction | Complexity |
|-----------|-------------------|------------|
| Naked functions | 10-20% | Medium |
| Tail-chaining | 30-40% | None (hardware) |
| Short ISRs | 50%+ | Low |
| Deferred processing | 80%+ | Medium |

---

