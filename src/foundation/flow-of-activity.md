# Flow of Activity

> "Concurrency is not an option in firmware; it is the environment. If you don't manage the flow, the hardware will manage it for you, usually at the worst possible time."

## The Hook: The Illusion of Parellelism
Embedded systems appear to do everything at once. They blink LEDs, read sensors, and talk to the cloud simultaneously. But unless you have a multi-core beast, your processor is a serial actor. The **Flow of Activity** is the art of choosing what to do *now* and what to defer.

## The Foundation: Schedulers & Tasks
In a bare-metal system, you are the scheduler.

*   **Tasks:** Discrete units of work.
*   **The Super-Loop:** The simplest scheduler. A `while(1)` loop that executes tasks in order. 
*   **Latency:** The time between an event and its response. In a super-loop, latency is the sum of all other task execution times.

### Concurrency Hazards: The Mid-Read Update
Many software-only solutions fail because they ignore hardware reality. The most dangerous hazard is the **Mid-Read Update**. 

If you are reading a 32-bit timestamp on an 8-bit or 16-bit processor, the read takes multiple instructions. If an interrupt updates that timestamp in the middle of your read, the resulting value will be nonsense (e.g., halfway between the old and new time).

**The Protection Ladder:**
1.  **Volatile:** Tells the compiler "this value changes behind your back." Essential, but does *not* provide atomicity.
2.  **Interrupt Masking:** Turn off interrupts during the read. Effective, but increases system latency.
3.  **Atomic Load/Store:** Use specialized CPU instructions that guarantee a single-cycle read or write.
4.  **Double Buffering:** Let the producer write to one buffer while the consumer reads from another.

## Mutexes & Semaphores
In a modern RTOS like **Zephyr**, you don't toggle interrupts manually.

1.  **Mutex (Mutual Exclusion):** A "token" for a resource. Only one task can hold it. If Task A has the UART, Task B must wait.
2.  **Semaphore:** A counter for resources. Useful for producer-consumer patterns (e.g., a buffer with 10 slots).

## State Machines: The Logic Logic
If your code is full of nested `if-else` blocks and flags, you need a state machine. It turns spaghetti code into a verifiable map.

*   **States:** Where the system is (e.g., `IDLE`, `BUSY`, `ERROR`).
*   **Events:** What happened (e.g., `BUTTON_PRESSED`, `TIMER_EXPIRED`).
*   **Transitions:** The logic that moves the system from one state to another.

### Statechart Maturity Patterns
Two common state machine pitfalls lead to "ghost in the machine" bugs:
1.  **The Initialization Void:** Forgetting to define an explicit initial state. Every state machine must start in a narrow, safe configuration (e.g., `POWER_ON`).
2.  **Guard Collisions:** Arcing from a state with two different conditions that might be true at the same time. This leads to non-deterministic behavior. Ensure every transition guard is mutually exclusive.

## Implementation Patterns: How to Code the Flow

Choosing the right structure for your state machine depends on whether your system is driven by **What it is** (State) or **What happened** (Event).

### 1. State-Centric (The Switch Statement)
The most common approach. A giant `switch(state)` handles events within each case.
*   **Best for:** Systems where behavior is highly dependent on internal context (e.g., a device that behaves differently when "Connected" vs "Idle").
*   **The Pro:** Easy to follow the logic of a single state.
*   **The Con:** Each state must "know" about its sibling states to trigger transitions.

### 2. Event-Centric (The Event Dispatcher)
The logic is driven by a `switch(event)`. 
*   **Best for:** Systems with global events that must be handled regardless of state (e.g., an "Emergency Stop" button).
*   **The Pro:** Consolidates common event logic in one place.
*   **The Con:** Can be difficult to manage state-specific variables or timeouts.

### 3. The State Pattern (Function Pointers)
An object-oriented approach in C. Each state is a `struct` of function pointers (e.g., `enter`, `exit`, `handle_event`).
*   **Best for:** Complex modular systems where states are added or removed frequently.
*   **The Pro:** High encapsulation. No giant switch statements.
*   **The Con:** Small overhead for the pointer indirection.

---

## Concurrency Hazards: The Race to the Bit

A **Race Condition** occurs when the timing or order of events affects the system's correctness.

### Anatomy of a Failure: The Shared Counter
Consider a simple global variable `button_press_count` accessed by an Interrupt Service Routine (ISR) and the `main` loop.
1.  **Main Loop:** Reads `count` (Value = 10).
2.  **Interrupt Fires:** Increments `count` to 11.
3.  **Main Loop:** Increments its *local* copy to 11 and writes it back to memory.
4.  **The Result:** The increment from the interrupt is lost. The value should be 12, but it is 11.

> [!WARNING]
> This is a **Heisenbug**: it only appears when the timing is "just wrong." You cannot find this by stepping through code in a debugger. You can only fix it by design.

---

## Real-Time: Deadlines vs. Speed
Real-time firmware is not about being "fast." It is about being **predictably fast enough**.

*   **Worst-Case Execution Time (WCET):** You must know the longest possible time a task takes to run. Measuring "average" time is useless for safety-critical systems.
*   **The Deadline Rule:** A task must complete before its next period starts. If a task executes every 10ms, but occasionally takes 11ms, your system is broken regardless of its "average" speed.

### Scheduling Approaches
1.  **Pure Static Loop:** Call every task in a fixed order. Predictable, but inflexible.
2.  **Rate Monotonic Scheduling (RMS):** Prioritize tasks by their frequency. The most frequent task gets the highest priority. This is the mathematical gold standard for RTOS scheduling.

## The Modern Shift: Event-Driven Architecture
The "Zero to One" shift in flow is moving from **Polling** (asking "is it done yet?") to **Events** (waiting for "I'm done"). 

*   **Traditional:** CPU burns cycles in a loop checking a flag.
*   **Modern:** The CPU sleeps (`WFI` - Wait For Interrupt). The hardware wakes the kernel only when there is work to do. This is the foundation of battery-powered endurance.

## RTOS Internals: What the Scheduler Actually Does

Understanding what a scheduler does under the hood prevents you from making incorrect assumptions about task behavior.

### The Context Switch

When the scheduler preempts Task A to run Task B, it must save the entire CPU state of Task A: general-purpose registers, the Program Counter, the Stack Pointer. This is the **task block** (also called TCB). On ARM Cortex-M, the hardware pushes eight registers to the stack automatically on every interrupt entry. The `PendSV` handler (the kernel's context-switch mechanism) saves the rest.

The cost? Roughly 50-200 CPU cycles. On a 64 MHz MCU, that is microseconds not milliseconds.

### Priority Inversion

This is the bug that killed the Mars Pathfinder mission in 1997.

1.  Task T1 (low priority) locks a mutex on a shared resource.
2.  Task T3 (high priority) tries to acquire the same mutex and blocks.
3.  Task T2 (medium priority) preempts T1 and runs indefinitely.
4.  T3 (the high-priority real-time task) is now starved by T2 (the medium-priority task), an indirect effect.

**Solution: Priority Inheritance.** When a high-priority task blocks on a mutex held by a lower-priority task, the OS temporarily elevates the lower-priority task's priority to the higher level. T1 finishes quickly, releases the mutex, and T3 unblocks.

Zephyr implements priority inheritance by default. Without it, any system with multiple priority levels and shared mutexes is vulnerable.

### Privilege Separation (MSP vs. PSP)

The Cortex-M has two stack pointers:
*   **MSP (Master Stack Pointer):** Used by the kernel and all interrupt handlers.
*   **PSP (Process Stack Pointer):** Used by user-level tasks.

A real OS switches the PSP register on each context switch and configures the CPU to run application tasks at "user" privilege. This way, a misbehaving task cannot corrupt the kernel stack or directly access peripherals; it must go through a **system call (SVC)**.

This is the hardware-enforced boundary between user space and kernel space.

---

## Real-World Case Studies: Flow and Data Integrity

### The Cost of a Data Handshake (Negative Example)
In 1999, the **Mars Climate Orbiter** was lost as it entered the Martian atmosphere. The disaster was caused by a simple unit mismatch at a component interface.

*   **The Error:** One software module (provided by Lockheed Martin) calculated the thruster impulse in English units (Pound-seconds). A second module (provided by JPL) expected these values in Metric units (Newton-seconds). 
*   **The Data:** The discrepancy was a factor of **4.45**. This caused the orbiter to enter the atmosphere much lower than intended, resulting in its total destruction. The mission cost was **$327.6 million**.
*   **The Lesson:** System "Flow" is not just about timing; it is about the absolute integrity of data across every software boundary. Strong typing and explicit unit enforcement (e.g., using `std::chrono` in C++ or specialized types in Rust) are not optional for high-assurance systems.

---

---

## References & Further Reading
*   *Making Embedded Systems*, 2nd Edition by Elecia White (Chapter 5)
*   [Zephyr Project: Kernel Services](https://docs.zephyrproject.org)
*   [Statecharts: A Visual Formalism for Complex Systems](https://statecharts.dev)
