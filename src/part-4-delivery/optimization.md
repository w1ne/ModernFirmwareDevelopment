# The Optimization Shift

Optimization in firmware has two failure modes. The first is ignoring it entirely until the product ships slow and power-hungry. The second is obsessing over it before understanding where the bottleneck is.

The modern approach: **measure, then optimize, never guess.**

## Code Size Optimization

Microcontrollers have finite flash. Every byte matters on constrained targets.

1.  **Link-Time Optimization (LTO):** Enable `--flto` in your compiler flags. LTO allows the entire program to be analyzed at link time, eliminating dead code and inlining cross-file functions that the compiler could never see before.
2.  **Size vs. Speed flag (`-Os`):** Compiling with `-Os` instead of `-O2` reduces code size with minimal performance impact. For most firmware, this is the correct default.
3.  **`__attribute__((section))`:** Place time-critical ISR routines in fast CCRAM. Place less-used init code in flash. Explicit section placement gives you control the linker alone cannot achieve.

## Speed Optimization

Before touching a single register, profile with real data.

1.  **Profiling with DWT:** ARM's Data Watchpoint and Trace (DWT) unit includes a free-running cycle counter. Use it to measure function durations with zero external tooling.
2.  **DMA over CPU Loops:** Never transfer data in a CPU polling loop if DMA is available. A CPU copying 1 MB of data to a peripheral is a CPU doing nothing useful. DMA frees it entirely.
3.  **Cache Coherency:** On Cortex-M7 and above, mismanaged caches create invisible bugs. Clean/invalidate the D-Cache before and after DMA transfers. Failing to do so is one of the most common hard-to-debug bugs in modern high-performance MCUs.

## Power Optimization

The modern expectation: devices run for years on a coin cell.

1.  **Clock Gating:** Never run a peripheral clock if the peripheral is idle. On STM32, every peripheral is controlled by an AHB/APB clock enable bit. Turn it off when done.
2.  **DVFS (Dynamic Voltage and Frequency Scaling):** Drop the CPU frequency to 8 MHz in low-activity periods. The power consumed by the CPU is roughly proportional to `V^2 * f`. Halving the frequency more than halves the dynamic power.
3.  **Tickless Idle (RTOS):** Traditional RTOS designs wake the CPU on every scheduler tick (e.g., 1 ms). A "tickless" RTOS like Zephyr suppresses ticks when no tasks are ready to run, allowing the CPU to stay in a sleep mode until the next real event.

## The Rule of Optimization
> "Premature optimization is the root of all evil." However, designing for optimization from day one (DMA, clean interfaces, profiling hooks) is engineering discipline, not premature optimization.

Measure first. Optimize the 3% that matters. Keep the 97% readable.
