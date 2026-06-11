# Embedded Linux

This repository is a structured capture of embedded system behavior across firmware, kernel, and hardware interaction layers. It focuses on system-level relationships rather than isolated concepts, emphasizing execution flow, constraints, and failure modes in real embedded environments.

---

Embedded systems operate under strict coupling between hardware state and software execution. Unlike general-purpose systems, behavior is governed by deterministic timing, memory-mapped interactions, and platform-specific initialization sequences. System correctness is therefore defined not only by logic, but by ordering, latency, and side effects across hardware boundaries.

Boot flow is the first point of system determinism. Execution begins at reset vector → ROM code → bootloader stages (SPL / U-Boot or equivalent) → kernel entry → init system. Each stage progressively transitions from hardware initialization to abstraction layers. At the kernel boundary, control shifts from firmware-defined behavior to scheduler-driven execution, where timing and concurrency are no longer static but policy-driven.

Device tree acts as a declarative hardware contract between silicon and kernel. It removes hardware descriptions from executable code paths and relocates them into structured metadata consumed at runtime. Misalignment between DT and actual hardware state results in silent failures, often observed as probe-time inconsistencies, missing IRQ bindings, or incorrect clock tree propagation.

---

At the kernel level, execution is dominated by interrupt context, process context, and deferred work mechanisms. Interrupt handlers operate under strict constraints: no blocking, minimal stack usage, and deterministic exit paths. Deferred execution mechanisms such as softirqs, tasklets, and workqueues exist to move non-critical work out of atomic context while preserving responsiveness.

Memory management introduces another layer of system constraint. Physical memory is abstracted into zones and pages, with allocation paths bifurcating into atomic and non-atomic contexts. Cache behavior, alignment constraints, and DMA coherence requirements define correctness boundaries. In many embedded platforms, memory is non-uniform, requiring explicit handling of contiguous regions for hardware-accessible buffers.

---

Driver architecture is fundamentally a contract between kernel subsystems and hardware registers. Character drivers expose sequential access semantics, while block drivers abstract storage-level random access. Network drivers operate on packet-based abstractions with interrupt-driven RX/TX rings. Regardless of type, all drivers converge on a common pattern: initialization → resource acquisition → registration → interrupt binding → operational state machine.

Memory-mapped I/O remains the primary mechanism for hardware interaction. Registers are accessed through volatile-qualified pointers or kernel abstraction helpers. Ordering constraints between read/write operations are enforced through memory barriers, without which hardware state transitions may become non-deterministic on weakly ordered architectures such as ARM.

---

Concurrency in embedded systems is not an abstraction but a physical contention model. Race conditions emerge not only from thread scheduling but from interrupt preemption, SMP execution, and deferred execution paths. Synchronization primitives such as spinlocks, mutexes, and atomics operate at different layers of this contention model. The choice of primitive is constrained by execution context (atomic vs process) and latency requirements.

Priority inversion is treated as a system-level anomaly where scheduling assumptions are violated due to resource locking chains. Mitigation strategies rely on priority inheritance or ceiling protocols, but in practice require careful system design to prevent unbounded blocking in critical paths.

---

Communication subsystems introduce timing and framing constraints across electrical and logical layers. UART operates on asynchronous sampling assumptions where baud alignment defines correctness. SPI introduces clocked full-duplex transfer with explicit chip select arbitration. I2C operates under shared-bus arbitration with address-based transaction control and clock stretching behavior. CAN extends this model into deterministic arbitration with message priority encoded at the physical layer using dominant/recessive bit evaluation.

Signal integrity and timing margins become first-order concerns at higher speeds. Bus contention, rise/fall times, and sampling edges directly influence protocol correctness, often requiring hardware-level validation rather than software debugging.

---

Firmware update systems introduce controlled mutation of persistent system state. A reliable update chain requires verification (CRC/signature), atomic write strategies, and rollback capability. Dual-partition schemes isolate runtime execution from update staging. Bootloader logic becomes critical in enforcing integrity before handoff to kernel execution. Watchdog systems operate as external correctness validators, enforcing recovery when software control flow diverges from expected liveness conditions.

---

Debugging embedded systems requires correlation across software execution traces and hardware signal states. Software-level observability (logs, traces, kernel debugfs) is insufficient without hardware-level validation (JTAG halt states, bus analysis, register inspection). Failure analysis typically involves reconstructing system state at the moment of divergence rather than observing runtime behavior alone.

Common failure classes include uninitialized hardware state, race-induced memory corruption, incorrect device tree bindings, DMA misconfiguration, and interrupt storm conditions. Each failure class manifests differently across kernel logs, hardware signals, and system responsiveness.

---

This repository serves as a continuously evolving capture of system-level embedded behavior, focusing on correctness boundaries, execution ordering, and hardware-software coupling rather than isolated implementation patterns.
