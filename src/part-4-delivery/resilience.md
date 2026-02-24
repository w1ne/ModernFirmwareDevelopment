# Watchdogs and System Resilience

A watchdog timer is not a reliability feature. It is a last-resort recovery mechanism. If your watchdog trips regularly in the field, you have a bug. The watchdog just prevents that bug from turning into a permanently dead device.

## How a Watchdog Works

A hardware counter starts at a preset value and counts down. The application must write a specific unlock sequence to a watchdog register before the counter reaches zero. If it fails to do so, the MCU resets.

The counter reaching zero triggers a reset regardless of what the CPU is doing. An infinite loop, a deadlocked RTOS task, or a corrupted stack pointer, it does not matter. The WDT fires.

**On STM32:** IWDG (Independent WDT) runs on the LSI oscillator, independent of the main clock. Even if the main clock hangs, IWDG fires.

**On nRF52:** Use the `WDT` peripheral with a dedicated run register. Requires writing `0x6E524635` to `WDTRR` to kick it.

**In Zephyr:** The `wdt_feed()` API abstracts the hardware:

```c
// In initialization:
wdt = device_get_binding("WDT_0");
struct wdt_timeout_cfg wdt_config = {
    .window.min = 0,
    .window.max = 5000, // 5 second timeout
    .callback = wdt_callback,
};
wdt_install_timeout(wdt, &wdt_config);
wdt_setup(wdt, WDT_OPT_PAUSE_HALTED_BY_DBG);

// In each task's main loop:
wdt_feed(wdt, channel_id);
```

## The Supervised Watchdog Pattern

Kicking the watchdog from a single timer ISR is the most common and most dangerous watchdog anti-pattern. A timer ISR runs even when every other task is dead. The result: a completely hung system that still kicks its watchdog every 100 ms while doing nothing useful.

**Correct pattern: task voting.** Each task sets a bit in a shared bitmask when it completes its work. A dedicated supervisor task checks the bitmask periodically and only kicks the hardware watchdog when every required task has reported in. Then it resets the bitmask.

```c
// Each task:
atomic_or(&task_alive_mask, BIT(MY_TASK_ID));

// Supervisor task (lowest priority):
void supervisor_task(void *a, void *b, void *c) {
    while (1) {
        k_sleep(K_MSEC(1000));
        uint32_t alive = atomic_get(&task_alive_mask);
        if ((alive & REQUIRED_TASKS) == REQUIRED_TASKS) {
            wdt_feed(wdt, channel_id);
            atomic_set(&task_alive_mask, 0);
        }
        // If not all tasks reported: watchdog fires on next cycle
    }
}
```

## Use a Windowed Watchdog

A standard watchdog fires if you kick it too late. A **windowed watchdog** fires if you kick it too early or too late. This catches a task that loops so fast it kicks the watchdog thousands of times per second while ignoring all other work.

STM32's WWDG supports windowed mode. Enable it when resources allow.

## Tracking Resets: The Boot Counter

A watchdog that fires and recovers silently is a bug that looks like normal operation. Track every reset and its reason.

The MCU reset reason register (e.g., `RCC->CSR` on STM32, `RESETREAS` on nRF52) tells you whether the last reset was from:
- Power-on
- IWDG (watchdog)
- Software (intentional reboot)
- Pin reset

Write the reset reason to a non-volatile counter in retained RAM or flash at every boot. If the IWDG counter increments in the field, you have a real problem.

MCUboot's boot counter integration takes this further: if the application fails to confirm itself within a boot window, MCUboot rolls back to the previous firmware image automatically.

---

## Real-World Case Studies: System Resilience

### The Cost of a Priority Paradox (Negative Example)
In 1997, the **Mars Pathfinder** began resetting itself sporadically during meteorological data collection. The cause was a **Priority Inversion** in the VxWorks RTOS.

*   **The Error:** A low-priority task held a mutex for a shared data bus. A medium-priority task preempted it. A high-priority bus management task then needed the bus and became blocked. Since the medium task wouldn't stop, the low-priority task couldn't finish to release the bus. The high-priority task stalled.
*   **The Data:** The watchdog timer detected the high-priority task hang and reset the system. Engineers spent **3 weeks** replicating the bug on Earth before patching the spacecraft.
*   **The Lesson:** Use priority inheritance on all shared mutexes. Never assume that a watchdog will fix a systemic architectural flaw. 

### The Success of Extreme Remote Resilience (Positive Example)
In 2023, the **Voyager 1** spacecraft began transmitting garbled data from 15 billion miles away. A single memory chip in the Flight Data Subsystem (FDS) had failed.

*   **The Resilience:** The engineering team could not repair the hardware. Instead, they segmented the affected software code and relocated it to scattered, unused areas of the remaining memory.
*   **The Data:** After **5 months** of analysis and remote patching using 1970s assembly, the team restored all engineering and science data. The mission continues after **46 years** in space.
*   **The Lesson:** Resilient software treats hardware as a volatile resource. Design for relocation. Build the capability to patch Every byte of the system remotely.

---

## System Reset: Safety During Recovery

The MCU resetting does not reset the physical world. A motor commanded to full speed before the MCU reset is still at full speed during the reboot.

Rules for well-behaved resets:

1. **Safe outputs on reset.** GPIO output reset state is configurable. Set actuator outputs to their safe default (off, neutral, braked) in hardware before the boot sequence completes.
2. **Read before write.** On startup, read all sensor states before commanding any output. Never command based on stale internal state.
3. **Initialize history buffers from reality.** If your control loop uses a moving average, pre-fill the history with the current measured value, not zero. A zero-initialized integrator will spike the output on first loop execution.
4. **Detect repeated resets.** If the system resets 3 times within 30 seconds, something is systemically wrong. Enter a safe/degraded mode rather than looping through the same crash repeatedly. Log the condition.

---

## Mental Model: Silent vs. Loud Failures

How a system fails determines its safety.

*   **Silent Failure (The Danger):** The system encounters an error but continues to operate in an undefined or unsafe state. It does not log the error or alert the user. This is common with "swallowed" exceptions and ignored return codes.
*   **Loud Failure (The Safe State):** The system identifies the error and immediately transitions to a predefined "Safe State." This may involve shutting down actuators, lighting an error LED, and broadcasting a telemetry alert.
*   **The Principle:** In a high-assurance system, there are no "Minor" errors. Every error that moves the system out of its verified operational range must be treated as a "Loud Failure" requiring immediate recovery or safe-state transition.

---

> [!IMPORTANT]
> Never disable the watchdog once it is running. If a long operation would exceed the timeout, kick the watchdog at checkpoints within that operation, or use Zephyr's `wdt_setup()` with `WDT_OPT_PAUSE_IN_SLEEP` to pause during low-power modes only.
