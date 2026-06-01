
# Embedded C Interview Questions - Patch 3

[← Back to Main README](README.md) | [← Previous Patch](patch-2.md) | [Next Patch →](patch-4.md)

---

## Q43: Memory-Mapped Register Access

###  Definition

Memory-mapped I/O (MMIO) uses regular memory addresses to access hardware registers. The CPU reads/writes to these addresses like normal memory, but the hardware intercepts them for peripheral control.

###  Syntax

```c
// Hardware register definition (from vendor header)
typedef struct {
    __IO uint32_t CR1;      // Control register 1, offset 0x00
    __IO uint32_t CR2;      // Control register 2, offset 0x04
    __IO uint32_t SR;       // Status register, offset 0x08
    __IO uint32_t DR;       // Data register, offset 0x0C
} USART_TypeDef;

// Peripheral base addresses
#define PERIPH_BASE       0x40000000UL
#define APB1PERIPH_BASE   PERIPH_BASE
#define USART2_BASE       (APB1PERIPH_BASE + 0x4400)

// Cast to pointer
#define USART2            ((USART_TypeDef *)USART2_BASE)

// Usage
void uart_init(void) {
    // Enable clock
    RCC->APB1ENR |= RCC_APB1ENR_USART2EN;
    
    // Configure
    USART2->CR1 = 0;
    USART2->CR2 = 0;
    USART2->BRR = 0x1D4C;  // 115200 baud @ 16MHz
    USART2->CR1 |= USART_CR1_TE | USART_CR1_RE;  // Enable TX/RX
    USART2->CR1 |= USART_CR1_UE;  // Enable USART
}

// Volatile is critical!
// Wrong - compiler may optimize
uint32_t *status = (uint32_t*)0x40000408;
*status = 1;  // Might be optimized away

// Correct - volatile prevents optimization
volatile uint32_t *status = (volatile uint32_t*)0x40000408;
*status = 1;  // Always performed

// Bit-band alias for atomic access
#define GPIOA_ODR_ADDR   0x40020014
#define GPIOA_ODR_BIT5   5
#define BITBAND(addr, bit) \
    ((volatile uint32_t*)((addr & 0xF0000000) + 0x2000000 + \
    ((addr & 0xFFFFF) << 5) + (bit << 2)))

volatile uint32_t *led = BITBAND(GPIOA_ODR_ADDR, GPIOA_ODR_BIT5);
*led = 1;  // Atomic bit set!

// Register access macros
#define REG_SET_BIT(reg, bit)   ((reg) |= (1UL << (bit)))
#define REG_CLR_BIT(reg, bit)   ((reg) &= ~(1UL << (bit)))
#define REG_GET_BIT(reg, bit)   (((reg) >> (bit)) & 1)

// Safe register modification (read-modify-write)
void set_register_field(uint32_t *reg, uint32_t mask, uint32_t shift, uint32_t value) {
    uint32_t tmp = *reg;
    tmp &= ~mask;
    tmp |= (value << shift) & mask;
    *reg = tmp;
}
```

### 💡 Explanation

**Common register access patterns:**

| Operation | Pattern | When to use |
|-----------|---------|-------------|
| Write-only | `REG = value` | Command registers |
| Read-modify-write | `REG |= mask` | Setting bits |
| Clear bits | `REG &= ~mask` | Clearing bits |
| Read status | `value = REG` | Status checks |

### 📋 Interview Tip

```c
// Atomic read-modify-write on ARM
// Use bit-banding for guaranteed atomicity
// OR disable interrupts for critical updates
__disable_irq();
REG |= (1 << 5);
__enable_irq();
```

### 🎭 Visual Comedy

```
CPU: "I'll write to address 0x40020014"
Hardware: "That's GPIO Port A Output Data Register"
CPU: "But it looks like memory..."
Hardware: "Surprise! You just turned on an LED!"

"Memory-mapped registers: Where writing to memory
actually blinks lights and spins motors!" 💡
```

---

## Q44: DMA and Scatter-Gather

### 📚 Definition

Direct Memory Access (DMA) transfers data between peripherals and memory without CPU intervention. Scatter-gather DMA uses a linked list of transfer descriptors for non-contiguous memory transfers.

### 📝 Syntax

```c
// Basic DMA transfer configuration
typedef struct {
    uint32_t src_addr;
    uint32_t dst_addr;
    uint32_t size;
    uint32_t config;
} DMA_Config_t;

void dma_transfer(DMA_Config_t *cfg) {
    // Configure DMA channel
    DMA1_Channel1->CPAR = cfg->src_addr;  // Peripheral address
    DMA1_Channel1->CMAR = cfg->dst_addr;  // Memory address
    DMA1_Channel1->CNDTR = cfg->size;     // Number of transfers
    DMA1_Channel1->CCR = cfg->config;     // Configuration
    
    // Enable DMA
    DMA1_Channel1->CCR |= DMA_CCR_EN;
    
    // Wait for completion (or use interrupt)
    while(DMA1_Channel1->CCR & DMA_CCR_EN);
}

// Scatter-gather DMA (STM32 example)
typedef struct {
    uint32_t src_addr;
    uint32_t dst_addr;
    uint32_t size;
    uint32_t next_descriptor;  // Pointer to next descriptor
} SG_Descriptor_t;

SG_Descriptor_t sg_list[3] = {
    {&buffer1[0], &peripheral->DR, 100, &sg_list[1]},
    {&buffer2[0], &peripheral->DR, 150, &sg_list[2]},
    {&buffer3[0], &peripheral->DR, 80,  0}  // Last
};

void dma_scatter_gather_init(void) {
    // Set up DMA for linked list mode
    DMA1_Channel1->CCR |= DMA_CCR_CHSEL;  // Circular/scatter mode
    DMA1_Channel1->CMAR = (uint32_t)&sg_list[0];  // First descriptor
}

// Double-buffered DMA (ping-pong)
#define BUFFER_SIZE 1024
uint8_t ping_buffer[BUFFER_SIZE];
uint8_t pong_buffer[BUFFER_SIZE];
volatile uint8_t active_buffer = 0;  // 0=ping, 1=pong

void DMA_IRQ_Handler(void) {
    if(DMA1->ISR & DMA_ISR_TCIF1) {
        // Transfer complete
        if(active_buffer == 0) {
            // Ping completed, switch to pong
            DMA1_Channel1->CMAR = (uint32_t)pong_buffer;
            active_buffer = 1;
            process_ping();  // Process while DMA fills pong
        } else {
            DMA1_Channel1->CMAR = (uint32_t)ping_buffer;
            active_buffer = 0;
            process_pong();
        }
        DMA1->IFCR |= DMA_IFCR_CTCIF1;  // Clear flag
    }
}

// Memory-to-memory DMA (for large copies)
void dma_memcpy(void *dest, void *src, size_t size) {
    // Configure for memory-to-memory
    DMA1_Channel1->CCR = DMA_CCR_MEM2MEM | DMA_CCR_PL_MEDIUM;
    DMA1_Channel1->CPAR = (uint32_t)src;
    DMA1_Channel1->CMAR = (uint32_t)dest;
    DMA1_Channel1->CNDTR = size / 4;  // Word transfers
    
    DMA1_Channel1->CCR |= DMA_CCR_EN;
    while(DMA1_Channel1->CCR & DMA_CCR_EN);
}
```

### 💡 Explanation

**DMA benefits:**
- CPU free to do other work during transfer
- Lower power (DMA more efficient than CPU for data movement)
- Predictable timing (no interrupt latency)

**Scatter-gather advantages:**
- Transfer to/from multiple buffers without CPU
- Circular buffers with DMA
- Efficient packet processing

### 📋 Interview Tip

**DMA alignment requirements:**
```c
// Most DMA requires word-aligned buffers
uint32_t aligned_buffer[256];  // Good
uint8_t unaligned_buffer[256]; // May cause issues
```

### 🎭 Visual Comedy

```
Without DMA:
CPU: "Copy byte 1... copy byte 2... copy byte 3..."
      "I could be doing real work, but NOOO!"

With DMA:
CPU: "DMA, copy 1024 bytes"
DMA: "Got it boss!" *whirrr*
CPU: "I'll go do something useful"

"DMA: The intern who does all the boring data moving!" 📋
```

---

## Q45: MPU (Memory Protection Unit) Configuration

### 📚 Definition

The MPU divides memory into regions with different access permissions (read/write/execute), preventing tasks from accessing memory they shouldn't and catching memory corruption early.

### 📝 Syntax

```c
// ARM Cortex-M MPU configuration
typedef struct {
    uint32_t base_addr;
    uint32_t size;
    uint8_t permissions;  // RW, RO, NO_ACCESS
    uint8_t executable;   // EXECUTE, NO_EXECUTE
} MPU_Region_t;

void mpu_init(void) {
    // Enable MPU
    MPU->CTRL = MPU_CTRL_PRIVDEFENA_Msk | MPU_CTRL_ENABLE_Msk;
    
    // Configure regions
    // Region 0: Flash (read-only, executable)
    MPU->RBAR = 0x08000000;  // Base address
    MPU->RASR = (0x1F << 1)  // Size: 2MB
              | (0x01 << 24) // Enable
              | (0x0 << 19)  // No XN (executable)
              | (0x1 << 16); // Full access (read-only)
    
    // Region 1: SRAM (read-write, executable)
    MPU->RBAR = 0x20000000 | (1 << 4);  // Region 1
    MPU->RASR = (0x1D << 1)  // Size: 512KB
              | (0x01 << 24) // Enable
              | (0x0 << 19)  // No XN (executable)
              | (0x3 << 16); // Full access (read-write)
    
    // Region 2: Peripherals (read-write, non-executable)
    MPU->RBAR = 0x40000000 | (2 << 4);  // Region 2
    MPU->RASR = (0x1F << 1)  // Size: 2MB
              | (0x01 << 24) // Enable
              | (0x1 << 19)  // XN (no execute!)
              | (0x3 << 16); // Full access
    
    // Region 3: Stack guard page (no access)
    MPU->RBAR = (STACK_END - 32) | (3 << 4);
    MPU->RASR = (0x9 << 1)   // Size: 32 bytes
              | (0x01 << 24) // Enable
              | (0x0 << 19)  // No XN
              | (0x0 << 16); // No access
}

// FreeRTOS MPU support
void task_mpu_config(void) {
    // Define MPU regions for task
    MPU_REGION_ACCESS region[] = {
        {.base = FLASH_BASE, .length = FLASH_SIZE, 
         .access = mpuRO},
        {.base = SRAM_BASE, .length = SRAM_SIZE, 
         .access = mpuRW},
        {0, 0, mpuPROHIBITED}
    };
    
    xTaskCreateRestricted(region, task_function, 
                          "protected_task", &task_handle);
}

// MPU fault handler
void MemManage_Handler(void) {
    uint32_t fault_addr = SCB->MMFAR;  // Fault address
    uint32_t fault_status = SCB->CFSR; // Fault status
    
    if(fault_status & SCB_CFSR_MSTKERR_Msk) {
        // Stack corruption
        log_error("MPU stack fault at 0x%08X", fault_addr);
    }
    
    if(fault_status & SCB_CFSR_MMARVALID_Msk) {
        // Invalid access
        log_error("MPU access fault at 0x%08X", fault_addr);
    }
    
    // Safe shutdown
    system_safe_state();
    while(1);
}
```

### 💡 Explanation

**MPU region alignment requirements:**
- Size must be power of 2 (32, 64, 128, ... 4GB)
- Base address must be aligned to size
- Minimum region size: 32 bytes (Cortex-M)

### 📋 Interview Tip

**Debug MPU faults:**
```c
// In fault handler
uint32_t mpu_region = (SCB->MMAR & 0x1F) >> 4;
printf("Fault in region %lu\n", mpu_region);
```

### 🎭 Visual Comedy

```
Without MPU:
Task A: "I'll just write to Task B's stack" 
Task B: *crashes mysteriously* "Who did this?"

With MPU:
Task A: "I'll write to Task B's..."
MPU: "STOP RIGHT THERE CRIMINAL SCUM!"
Task A: *trapped in fault handler*

"MPU: The bouncer that keeps tasks out of each other's business!" 🛡️
```

---

## Q46: Clock Gating and Power Management

### 📚 Definition

Clock gating disables clocks to unused peripherals to save power. Modern MCUs have multiple power modes (run, sleep, stop, standby) with different wake-up latencies.

### 📝 Syntax

```c
// Peripheral clock gating (STM32)
void peripheral_power_management(void) {
    // Enable only needed peripherals at startup
    RCC->AHB1ENR = RCC_AHB1ENR_GPIOAEN |   // GPIO A
                   RCC_AHB1ENR_GPIOCEN;    // GPIO C
    
    // Disable unused peripherals
    RCC->APB1ENR = 0;  // Disable all APB1 peripherals
    
    // Dynamic clock enable/disable
    void uart_send(const char *data) {
        __disable_irq();
        RCC->APB1ENR |= RCC_APB1ENR_USART2EN;  // Enable clock
        __enable_irq();
        
        // Send data (200us typically)
        
        __disable_irq();
        RCC->APB1ENR &= ~RCC_APB1ENR_USART2EN;  // Disable clock
        __enable_irq();
    }
}

// Sleep modes
void enter_sleep_mode(void) {
    // Set sleep mode
    SCB->SCR &= ~SCB_SCR_SLEEPDEEP_Msk;
    
    // Wait for interrupt instruction
    __WFI();  // CPU sleeps until interrupt
}

void enter_stop_mode(void) {
    // Configure wake-up source
    EXTI->IMR |= EXTI_IMR_MR0;  // Wake on pin
    
    // Enter stop mode
    PWR->CR |= PWR_CR_LPDS;      // Low-power sleep
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();  // Wake on interrupt
}

void enter_standby_mode(void) {
    // Reset all peripherals
    RCC->CSR |= RCC_CSR_RMVF;
    
    // Configure wake-up pin
    PWR->CSR |= PWR_CSR_EWUP;
    
    // Enter standby
    PWR->CR |= PWR_CR_PDDS;
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();  // Wake only on reset or wake-up pin
}

// Dynamic voltage/frequency scaling (DVFS)
void set_cpu_frequency(uint32_t freq_mhz) {
    if(freq_mhz <= 16) {
        // Low power mode
        FLASH->ACR &= ~FLASH_ACR_LATENCY;
        RCC->CFGR = (RCC->CFGR & ~RCC_CFGR_SW) | RCC_CFGR_SW_HSI;
    } else if(freq_mhz <= 84) {
        // Medium performance
        FLASH->ACR = FLASH_ACR_LATENCY_2WS;
        RCC->CFGR = (RCC->CFGR & ~RCC_CFGR_SW) | RCC_CFGR_SW_PLL;
    }
}

// Power domain management
typedef enum {
    POWER_DOMAIN_VDD,   // Always on
    POWER_DOMAIN_VBAT,  // Backup battery
    POWER_DOMAIN_OFF    // Can be shut down
} power_domain_t;

void manage_backup_domain(void) {
    // Enable backup domain access
    PWR->CR |= PWR_CR_DBP;
    
    // Preserve RTC during standby
    RCC->BDCR |= RCC_BDCR_RTCEN;
    
    // Prevent backup write
    PWR->CR &= ~PWR_CR_DBP;
}
```

### 💡 Explanation

**Power modes comparison:**

| Mode | Current | Wake-up | Wake Latency |
|------|---------|---------|--------------|
| Run | 20-100mA | N/A | N/A |
| Sleep | 5-20mA | Any interrupt | 1-2 cycles |
| Stop | 100-500µA | Selected interrupts | 5-10µs |
| Standby | 1-10µA | Reset/Wake pins | 50-100µs |

### 📋 Interview Tip

```c
// Measure power impact
void power_measurement(void) {
    uint32_t start = DWT->CYCCNT;
    
    // Enable peripheral clock
    RCC->APB1ENR |= RCC_APB1ENR_USART2EN;
    
    // Do operation
    uart_send("Hello");
    
    uint32_t cycles = DWT->CYCCNT - start;
    // Enable time = ~10 cycles, not worth it for single char!
    // Better to enable once, send multiple chars
}
```

### 🎭 Visual Comedy

```
MCU Power States:
Run:    [████████████] "I'M AWAKE!"
Sleep:  [████______]   "Just resting my eyes..."
Stop:   [██____]       "Shh, I'm sleeping"
Standby:[_]            "DON'T. WAKE. ME."

"Power management: Your MCU's version of
Monday morning vs Friday afternoon!" 😴
```

---

## Q47: Software Delay Without Optimization

### 📚 Definition

Software delays that survive compiler optimization require volatile variables or inline assembly to prevent the delay loop from being removed.

### 📝 Syntax

```c
// BROKEN - optimizer removes loop
void delay_bad(uint32_t ms) {
    uint32_t i, j;
    for(i = 0; i < ms; i++) {
        for(j = 0; j < 1000; j++);  // May be removed!
    }
}

// FIX 1: Volatile loop counter
void delay_volatile(uint32_t ms) {
    volatile uint32_t i, j;
    for(i = 0; i < ms; i++) {
        for(j = 0; j < 1000; j++);  // Now safe
    }
}

// FIX 2: Inline assembly (most precise)
void delay_cycles(uint32_t cycles) {
    __asm volatile (
        "1: subs %0, #1\n"  // Subtract 1
        "bne 1b\n"          // Branch if not zero
        : "+r"(cycles)      // Input/output
        :                   // No inputs
        : "cc"              // Clobbers condition codes
    );
}

// FIX 3: NOP loop
void delay_nop(uint32_t us) {
    // ARM: 1 NOP = 1 cycle at 1MHz = 1us
    // Adjust for CPU frequency
    uint32_t nops = us * (SystemCoreClock / 1000000);
    
    while(nops--) {
        __asm volatile("nop");
    }
}

// FIX 4: Use SysTick (most accurate)
volatile uint32_t systick_ms = 0;

void SysTick_Handler(void) {
    systick_ms++;
}

void delay_systick(uint32_t ms) {
    uint32_t start = systick_ms;
    while((systick_ms - start) < ms);
}

// Calibrated delay
void delay_us(uint32_t us) {
    // Assuming 100MHz CPU, 1 instruction = 10ns
    // Need ~100 cycles per microsecond
    
    // Use DWT cycle counter
    uint32_t start = DWT->CYCCNT;
    uint32_t target = start + (us * (SystemCoreClock / 1000000));
    
    while(DWT->CYCCNT < target);
}

// Initialize DWT for cycle counting
void init_cycle_counter(void) {
    CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
    DWT->CYCCNT = 0;
    DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
}
```

### 💡 Explanation

**Why loops get optimized:**
```c
// Compiler sees:
for(i = 0; i < 1000; i++);  // No side effects
// Compiler thinks: "This does nothing, remove it"
```

**Precision comparison:**
| Method | Precision | Portability | Power |
|--------|-----------|-------------|-------|
| Volatile | ±30% | High | Medium |
| NOP loop | ±10% | Medium | Medium |
| Assembly | ±5% | Low | Medium |
| SysTick | ±0.1% | High | Low (wakes CPU) |
| DWT | ±0.01% | Low | Low |

### 📋 Interview Tip

```c
// Use for short delays only (< 1ms)
// For longer delays, use RTOS delay or timer
void delay_short_us(uint32_t us) {
    if(us > 1000) {
        vTaskDelay(pdMS_TO_TICKS(us / 1000));  // RTOS handles long
    } else {
        delay_cycles(us * (SystemCoreClock / 1000000));
    }
}
```

### 🎭 Visual Comedy

```
Compiler optimization:
Loop: "I do nothing useful"
Compiler: "Then you don't exist" *poof*

Volatile variable:
Loop: "I do nothing useful... but I'm SPECIAL!"
Compiler: "Fine, you can stay"

"Software delays: Convincing the compiler your busy loop
is actually doing something important!" ⏱️
```

---

## Q48: Push-Pull vs Open-Drain

### 📚 Definition

GPIO output modes determine how the pin driver behaves. Push-pull actively drives high and low, while open-drain only drives low (high is high-impedance).

### 📝 Syntax

```c
// Push-pull configuration (default)
void gpio_push_pull_init(void) {
    GPIO_InitTypeDef gpio = {
        .Pin = GPIO_PIN_5,
        .Mode = GPIO_MODE_OUTPUT_PP,  // Push-pull
        .Pull = GPIO_NOPULL,
        .Speed = GPIO_SPEED_FREQ_LOW
    };
    HAL_GPIO_Init(GPIOA, &gpio);
}

// Open-drain configuration
void gpio_open_drain_init(void) {
    GPIO_InitTypeDef gpio = {
        .Pin = GPIO_PIN_5,
        .Mode = GPIO_MODE_OUTPUT_OD,  // Open-drain
        .Pull = GPIO_PULLUP,           // External pull-up needed
        .Speed = GPIO_SPEED_FREQ_LOW
    };
    HAL_GPIO_Init(GPIOA, &gpio);
}

// I2C bus (requires open-drain)
void i2c_gpio_config(void) {
    // SDA and SCL must be open-drain
    gpio_open_drain_init();
    
    // Release bus (high impedance)
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);  // Actually high-Z
}

// Multi-device wired-OR (open-drain required)
typedef struct {
    GPIO_TypeDef *port;
    uint16_t pin;
} open_drain_bus_t;

open_drain_bus_t error_bus = {GPIOA, GPIO_PIN_5};

void signal_error(uint32_t error_code) {
    // Drive low to indicate error
    HAL_GPIO_WritePin(error_bus.port, error_bus.pin, GPIO_PIN_RESET);
    
    // Wait for others
    delay_ms(100);
    
    // Release bus
    HAL_GPIO_WritePin(error_bus.port, error_bus.pin, GPIO_PIN_SET);
}

// Level shifting with open-drain
// 3.3V MCU to 5V device
void level_shifted_io_init(void) {
    // Open-drain with external 5V pull-up
    gpio_open_drain_init();
    // Connect pin to 5V device via 4.7k pull-up to 5V
}

// Bi-directional pin (software-selectable)
typedef enum {
    PIN_INPUT,
    PIN_OUTPUT_PP,
    PIN_OUTPUT_OD
} pin_mode_t;

void set_pin_mode(GPIO_TypeDef *port, uint16_t pin, pin_mode_t mode) {
    switch(mode) {
        case PIN_INPUT:
            port->MODER &= ~(3 << (pin*2));
            break;
        case PIN_OUTPUT_PP:
            port->MODER = (port->MODER & ~(3 << (pin*2))) | (1 << (pin*2));
            port->OTYPER &= ~(1 << pin);
            break;
        case PIN_OUTPUT_OD:
            port->MODER = (port->MODER & ~(3 << (pin*2))) | (1 << (pin*2));
            port->OTYPER |= (1 << pin);
            break;
    }
}
```

### 💡 Explanation

**Comparison:**

| Feature | Push-Pull | Open-Drain |
|---------|-----------|------------|
| Drives high? | Yes (active) | No (pull-up needed) |
| Drives low? | Yes | Yes |
| Multiple devices | No (conflicts) | Yes (wired-OR) |
| Speed | Fast (ns) | Slower (RC limited) |
| Power | Lower | Higher (pull-up current) |
| Level shifting | No | Yes |

### 📋 Interview Tip

**When to use open-drain:**
- I2C bus (required)
- Multiple interrupt sharing (wired-OR)
- Bidirectional lines
- Mixed voltage systems

### 🎭 Visual Comedy

```
Push-pull:
  3.3V ----[FET]---- Pin ----[FET]---- GND
           High        |        Low
  "I control everything!"

Open-drain:
  ┌─[Pull-up]─ 3.3V
  │
  Pin ----[FET]---- GND
           Low only
  "I can only pull down. Up to you!"

"Open-drain: The polite driver who only pulls low
and lets others pull high!" 🤝
```

---

## Q49: Ring Buffer with DMA

### 📚 Definition

Combining circular buffers with DMA creates an efficient, zero-CPU-overhead data streaming system. DMA fills/empties the buffer while the CPU processes data at its own pace.

### 📝 Syntax

```c
// DMA double-buffer ring buffer
#define RX_BUFFER_SIZE 1024
#define DMA_BUFFER_SIZE (RX_BUFFER_SIZE / 2)

typedef struct {
    uint8_t buffer[RX_BUFFER_SIZE];
    volatile uint32_t write_index;  // Updated by DMA
    volatile uint32_t read_index;   // Updated by CPU
    uint8_t dma_buffer1[DMA_BUFFER_SIZE];
    uint8_t dma_buffer2[DMA_BUFFER_SIZE];
    volatile uint32_t active_buffer;  // 0 or 1
} dma_ring_buffer_t;

dma_ring_buffer_t uart_rx;

// DMA half-transfer and full-transfer interrupts
void DMA1_Channel1_IRQHandler(void) {
    if(DMA1->ISR & DMA_ISR_HTIF1) {
        // Half transfer complete - buffer1 is full
        memcpy(&uart_rx.buffer[uart_rx.write_index], 
               uart_rx.dma_buffer1, DMA_BUFFER_SIZE);
        uart_rx.write_index = (uart_rx.write_index + DMA_BUFFER_SIZE) 
                               % RX_BUFFER_SIZE;
        DMA1->IFCR |= DMA_IFCR_CHTIF1;
    }
    
    if(DMA1->ISR & DMA_ISR_TCIF1) {
        // Full transfer complete - buffer2 is full
        memcpy(&uart_rx.buffer[uart_rx.write_index], 
               uart_rx.dma_buffer2, DMA_BUFFER_SIZE);
        uart_rx.write_index = (uart_rx.write_index + DMA_BUFFER_SIZE) 
                               % RX_BUFFER_SIZE;
        DMA1->IFCR |= DMA_IFCR_CTCIF1;
    }
}

// Circular DMA (no interrupts for continuous streaming)
void uart_dma_circular_init(void) {
    // Configure DMA for circular mode
    DMA1_Channel1->CCR = DMA_CCR_CIRC |     // Circular mode
                         DMA_CCR_MINC |     // Memory increment
                         DMA_CCR_PSIZE_8 |  // 8-bit peripheral
                         DMA_CCR_MSIZE_8 |  // 8-bit memory
                         DMA_CCR_PL_MEDIUM; // Medium priority
    
    DMA1_Channel1->CMAR = (uint32_t)uart_rx.dma_buffer1;
    DMA1_Channel1->CNDTR = DMA_BUFFER_SIZE;
    DMA1_Channel1->CCR |= DMA_CCR_EN;
}

// Zero-copy DMA read
uint32_t dma_ring_read(uint8_t *dest, uint32_t max_bytes) {
    uint32_t available = (uart_rx.write_index - uart_rx.read_index) 
                         % RX_BUFFER_SIZE;
    uint32_t to_read = (available < max_bytes) ? available : max_bytes;
    
    uint32_t space_until_wrap = RX_BUFFER_SIZE - uart_rx.read_index;
    uint32_t first_chunk = (to_read < space_until_wrap) ? 
                           to_read : space_until_wrap;
    
    // Copy without buffer (direct from ring)
    if(first_chunk) {
        memcpy(dest, &uart_rx.buffer[uart_rx.read_index], first_chunk);
        uart_rx.read_index = (uart_rx.read_index + first_chunk) % RX_BUFFER_SIZE;
    }
    
    uint32_t second_chunk = to_read - first_chunk;
    if(second_chunk) {
        memcpy(dest + first_chunk, &uart_rx.buffer[0], second_chunk);
        uart_rx.read_index = second_chunk;
    }
    
    return to_read;
}

// DMA circular buffer with index tracking
typedef struct {
    volatile uint32_t head;  // DMA writes here
    volatile uint32_t tail;  // CPU reads from here
    uint8_t data[DMA_BUFFER_SIZE];
} circular_dma_t;

circular_dma_t dma_buffer;

void DMA_IRQHandler(void) {
    // DMA automatically advances head in circular mode
    // Just need to check half/full flags if needed
    
    if(DMA1->ISR & DMA_ISR_HTIF1) {
        // At half buffer - can process first half
        process_dma_data(0, DMA_BUFFER_SIZE/2);
        DMA1->IFCR |= DMA_IFCR_CHTIF1;
    }
    if(DMA1->ISR & DMA_ISR_TCIF1) {
        // At full buffer - process second half
        process_dma_data(DMA_BUFFER_SIZE/2, DMA_BUFFER_SIZE);
        DMA1->IFCR |= DMA_IFCR_CTCIF1;
    }
}
```

### 💡 Explanation

**DMA vs CPU copy:**
- DMA handles the data movement
- CPU only processes when buffer half/full
- Achieves 100% bus utilization for streaming data

### 📋 Interview Tip

```c
// Check for overrun
uint32_t dma_ring_available(void) {
    uint32_t avail = (uart_rx.write_index - uart_rx.read_index) 
                     % RX_BUFFER_SIZE;
    if(avail > RX_BUFFER_SIZE - 100) {
        // Near overflow!
        uart_rx.read_index = uart_rx.write_index;  // Reset
        return 0;
    }
    return avail;
}
```

### 🎭 Visual Comedy

```
CPU-only: [Copy][Copy][Copy][Copy][Process] (busy,busy)

DMA + Ring:
DMA:     [Copy][Copy][Copy][Copy] (in background)
CPU:     [Process][Process][Process] (in parallel)

"Ring buffer + DMA = Having your cake and eating it too!" 🎂
```

---

## Q50: Jitter and Precise Timing

### 📚 Definition

Jitter is the variation in timing between expected and actual events. In embedded systems, interrupts, cache misses, and bus contention cause timing uncertainty.

### 📝 Syntax

```c
// Measure jitter using DWT cycle counter
typedef struct {
    uint32_t min;
    uint32_t max;
    uint32_t sum;
    uint32_t count;
} jitter_stats_t;

jitter_stats_t isr_jitter;

void TIM2_IRQHandler(void) {
    static uint32_t last_time = 0;
    uint32_t current = DWT->CYCCNT;
    
    if(last_time != 0) {
        uint32_t delta = current - last_time;
        uint32_t expected = TIMER_PERIOD_CYCLES;
        
        int32_t jitter = (int32_t)(delta - expected);
        
        // Update stats
        if(jitter < isr_jitter.min) isr_jitter.min = jitter;
        if(jitter > isr_jitter.max) isr_jitter.max = jitter;
        isr_jitter.sum += (jitter > 0 ? jitter : -jitter);
        isr_jitter.count++;
    }
    
    last_time = current;
}

// Minimize jitter with critical section
void low_jitter_isr(void) {
    // Run with interrupts disabled
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    // Critical timing code here
    toggle_pin();  // Measure with oscilloscope
    
    __set_PRIMASK(primask);
}

// Use DWT to measure code execution time
uint32_t measure_execution_time(void (*func)(void)) {
    uint32_t start = DWT->CYCCNT;
    func();
    uint32_t end = DWT->CYCCNT;
    return end - start;  // Cycles
}

// Measure worst-case timing
typedef struct {
    uint32_t worst_case;
    uint32_t best_case;
    uint32_t total_runs;
} timing_stats_t;

timing_stats_t uart_tx_stats;

void measure_uart_tx(void) {
    uint32_t time = measure_execution_time(uart_send_byte);
    
    if(time > uart_tx_stats.worst_case) {
        uart_tx_stats.worst_case = time;
    }
    if(time < uart_tx_stats.best_case) {
        uart_tx_stats.best_case = time;
    }
    uart_tx_stats.total_runs++;
}

// Cache effects on jitter
void cache_aware_measurement(void) {
    // First run - cache cold
    uint32_t cold = measure_execution_time(critical_func);
    
    // Second run - cache hot
    uint32_t hot = measure_execution_time(critical_func);
    
    printf("Jitter due to cache: %lu cycles\n", cold - hot);
}

// Real-time deadline check
void isr_handler(void) {
    uint32_t start = DWT->CYCCNT;
    
    // Process interrupt
    process_data();
    
    uint32_t duration = DWT->CYCCNT - start;
    uint32_t deadline_cycles = 1000;  // 10us @ 100MHz
    
    if(duration > deadline_cycles) {
        // Missed deadline!
        error_counter++;
        if(error_counter > 10) {
            system_error(ERROR_RT_DEADLINE_MISS);
        }
    }
}
```

### 💡 Explanation

**Jitter sources:**
| Source | Typical Jitter | Mitigation |
|--------|---------------|------------|
| Interrupt priority | ±100 cycles | Use same priority |
| Cache misses | ±50 cycles | Lock code in cache |
| Bus contention | ±20 cycles | Dedicate DMA |
| Branch prediction | ±10 cycles | Likely/unlikely hints |

### 📋 Interview Tip

```c
// Use timer capture for precise jitter measurement
void timer_capture_init(void) {
    TIM2->CCER |= TIM_CCER_CC1E;   // Enable capture
    TIM2->SMCR |= TIM_SMCR_TS_TI1FP1;  // Trigger on pin
    TIM2->CR1 |= TIM_CR1_CEN;
}

uint32_t measure_external_event_jitter(void) {
    uint32_t capture1 = TIM2->CCR1;
    uint32_t capture2 = TIM2->CCR1;  // Wait for next event
    return abs(capture2 - capture1);
}
```

### 🎭 Visual Comedy

```
Ideal timing:  |---|---| (perfect)

With jitter:   |-| |---|  |--| (all over the place)

"Jitter: Your code's equivalent of caffeine jitters.
Sometimes fast, sometimes slow, always unpredictable!" ☕

Oscilloscope trace:
Expected: ______|‾‾‾‾|______|‾‾‾‾|______
Actual:   ______|‾‾‾|_______|‾‾‾‾‾|_____
           "I'm a little wobbly today!"
```

---

## Quick Reference Card 4

### DMA Configuration

```c
// Typical DMA config for peripheral-to-memory
DMA->CCR = DMA_CCR_MINC   // Memory increment
         | DMA_CCR_CIRC    // Circular
         | DMA_CCR_TCIE    // Transfer complete interrupt
         | DMA_CCR_PSIZE_8 // 8-bit peripheral
         | DMA_CCR_MSIZE_8 // 8-bit memory
         | DMA_CCR_PL_HIGH; // High priority
```

### Power Mode Macros

```c
#define enter_sleep()   (SCB->SCR &= ~SCB_SCR_SLEEPDEEP_Msk, __WFI())
#define enter_stop()    (PWR->CR |= PWR_CR_LPDS, SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk, __WFI())
#define enter_standby() (PWR->CR |= PWR_CR_PDDS, SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk, __WFI())
```

### Pin Mode Decision Tree

```
Need bidirectional? → Open-drain
Need multiple devices? → Open-drain  
Need level shifting? → Open-drain
Need maximum speed? → Push-pull
Default choice? → Push-pull
```

### Timing Measurement Tools

| Tool | Resolution | Overhead |
|------|-----------|----------|
| DWT_CYCCNT | 1 cycle | ~5 cycles |
| GPIO toggle | ~10ns | ~20 cycles |
| Timer capture | 1 cycle | Hardware |
| Logic analyzer | 1ns (external) | Zero |

---

