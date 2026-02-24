# The Memory Safety Shift

In traditional firmware, memory is a flat, unprotected arena. A buffer overflow on the stack doesn't just crash a program; it silent corrupts adjacent variables or redirects execution. The modern approach treats memory as a structured resource with hardware-enforced boundaries.

## The End of the "Wild West" Heap

Dynamic memory (`malloc`/`free`) is the primary source of sporadic, hard-to-reproduce bugs: memory leaks, double-frees, and fragmentations.

1.  **"No-Heap" Architectures:** Many safety-critical systems (MISRA-C compliant) forbid the use of any dynamic allocation after the initialization phase. All memory must be allocated statically at compile time.
2.  **Memory Pools and Slabs:** Instead of a global heap, modern RTOSs like Zephyr use fixed-size memory pools. This eliminates fragmentation and provides deterministic allocation times. If a pool is empty, the system knows immediately, rather than waiting for the heap to slowly starve.

## The Execution Stack: Guarding the Growth

The stack grows backward. When it overflows into the `.bss` or `.data` sections, it silently overwrites application state.

### Modern Mitigation: Stack Guards
Modern microcontrollers use a **Memory Protection Unit (MPU)** to place an "inaccessible" region at the bottom of the stack. 
*   **Old Way:** Calculate stack usage manually and hope for the best.
*   **Modern Way:** Configure the MPU to trigger a hard fault the microsecond the stack pointer touches the guard region. This transforms a silent corruption into a synchronous, debuggable exception.

## The Safety Sovereign: Rust's Ownership Model

While C depends on developer discipline, **Rust** enforces memory safety at compile time. 

*   **Ownership:** Every piece of memory has a unique owner. When the owner goes out of scope, the memory is cleaned up.
*   **Borrowing:** The compiler ensures that you can never have multiple mutable references to the same data, preventing race conditions and stale pointers by design.

## Hardware-Enforced Isolation (MPU)

The MPU is not just for stack guards. Use it to:
*   **Prevent Execution from RAM:** Ensure that data buffers can never be executed as code, neutralizing a massive class of security vulnerabilities.
*   **Read-Only Sections:** Mark the flash and configuration registers as read-only for the application logic, allowing only privileged drivers to modify them.

---

## Real-World Case Studies: Memory Integrity

### The Cost of an Insecure Buffer (Negative Example)
In 2014, the **Heartbleed** vulnerability (CVE-2014-0160) exposed the secret keys and data of millions of servers. The cause was a simple missing bounds check in the OpenSSL library.

*   **The Error:** A client could send a "Heartbeat" request and specify a payload length up to 64 KB. The server code trusted this length without verifying that the actual request payload was that large.
*   **The Code (Conceptual C):**
```c
// Heartbleed vulnerability: trusts payload_length from the client
memcpy(response_buffer, request_payload, payload_length); 
// If the actual payload was 1 byte but length was 65535, 
// the server copied 64 KB of internal memory to the response.
```
*   **The Data:** Estimations place the global cleanup cost and market impact at over **$10 billion**. 
*   **The Lesson:** Never trust external input length. Always perform bounds checks on every memory operation.

### The Success of Language-Enforced Safety (Positive Example)
Between 2019 and 2022, the **Android** engineering team shifted from C++ to Rust for new systems development.

*   **The Shift:** By using a language that prevents buffer overflows and use-after-free bugs at compile time, the team eliminated the human error inherent in manual memory management.
*   **The Data:** Memory safety vulnerabilities in Android dropped from **76% to 24%** of total vulnerabilities within three years. Overall critical vulnerabilities decreased by **70%**.
*   **The Lesson:** Security is an engineering problem. Move the burden of safety from the developer's discipline to the language compiler.

---

---

## The 80% Threshold: The Cost of Fullness
Software development costs rise exponentially as hardware resources become full. 

**The Rule of Slack:** Avoid using more than **80% of any resource** (CPU cycles, Flash, or RAM) during the initial release. 
*   **Growth Room:** A system with slack can absorb emergency patches and unexpected requirements without requiring a complete redesign.

### Optimization Timing: Flow Before Speed
Premature optimization is a primary driver of technical debt. 
1.  **Phase 1: Implement & Verify.** Focus on correct logic and code clarity.
2.  **Phase 2: Measure.** Use profilers or GPIO toggling to identify actual bottlenecks.
3.  **Phase 3: Tune.** Optimize only the 3% of code that consumes 90% of resources.

### Resource Trading: The Flex Factor
Embedded resources are often exchangeable. Design your system with the flexibility to trade:
*   **Flash for RAM:** Store large lookup tables in Flash (constant memory) to save precious RAM.
*   **RAM for Cycles:** Buffer sensor data in RAM to process it in batches, reducing CPU wake-ups and power consumption.
*   **Cycles for Flash:** Use compression algorithms to store more data in Flash at the cost of higher CPU usage during decompression.

---

---

## Global Variables: The Memory Safety Hazard
Global variables cause implicit coupling and are a prime source of concurrency bugs.

### The Mitigation Ladder
If a variable absolutely must retain state across function calls, use these patterns from least to most risky:
1.  **Static Local Variables:** Scope the variable inside the function that needs it. It remains in memory but is invisible to the rest of the system.
2.  **Quasi-Globals (File Static):** Keep the variable visible only within a single `.c` or `.rs` file. This prevents far-flung modules from touching it.
3.  **Encapsulated Objects:** Hide the variable behind a set of access functions (e.g., `get_temp()`, `set_temp()`). This provides a single point for mutex protection and debugging hooks.
4.  **No Pointers to Globals:** Never pass the address of a global variable around. This makes it impossible to track who is modifying the state.

## The Rule of Memory Safety
> "Static is safer. Hardware-enforced is better. Compile-time checked is best."

---

## Mental Model: The Blast Radius (MPU Isolation)

A vulnerability or bug in one software module should not lead to a total system failure. 

*   **The Inhabitable Compartment:** Think of your firmware like a ship. A leak in the hull is only catastrophic if it can flood the entire vessel. By using the Memory Protection Unit (MPU), you divide your firmware into "Inhabitable Compartments." You can define these boundaries during the [Abstraction Shift](abstraction-shift.md) design phase.
*   **Defining the Radius:** The Blast Radius is the maximum extent of damage a single module can do. If the Bluetooth stack has its own MPU-protected region, a buffer overflow in that stack cannot touch the motor control logic.
*   **Hardware Enforcement:** Unlike "Software Rules," the MPU is a hardware-enforced boundary. If a module tries to write outside its blast radius, the CPU triggers a fault instantly. This contains the "explosion" of a bug and allows the rest of the system to enter a safe state.

---
