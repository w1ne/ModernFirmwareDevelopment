# Observability and Telemetry

A device that fails in the field and gives you no data is worse than one that fails in the lab. Observability is your window into production. Without it, you are debugging blind.

## The Three Pillars

Observability borrows from distributed systems engineering:

1.  **Logs:** What happened, in sequence.
2.  **Metrics:** How the system is behaving over time.
3.  **Traces:** Where time is being spent across the call stack.

Most embedded systems only implement the first. Modern firmware implements all three.

## Crash Reporting: Coredump on Embedded

When a device faults in the field, the MCU state at the moment of failure is priceless. Zephyr's **coredump subsystem** captures the CPU registers, stack, and selected memory regions into flash or a dedicated logging backend.

On your next OTA update retrieval, the device uploads the coredump to your backend. You get:
*   The faulting instruction address.
*   The call stack at the time of fault.
*   The values of all local variables.

This eliminates the "could not reproduce" class of bugs.

## Remote Logging

`printk` to UART is fine for a desk prototype. For 10,000 fielded devices, you need a logging backend that routes to your infrastructure.

*   **Zephyr's logging subsystem** supports backends: UART, RTT, Bluetooth, network. Switch backends with a Kconfig flag. No code changes needed.
*   **Structured logging:** Log in JSON or CBOR instead of plain text. This lets you query logs in tools like Elasticsearch or Grafana Loki.

---

## Run-Time Error Logs: The Persistence Policy
Logs are only useful if they survive a system crash or power cycle.

### The Essential Error Log Format
Every error log entry should be a structured "record of fact":
1.  **Timestamp:** Relative or wall-time of the event.
2.  **Tracking ID:** A unique identifier for the specific instance of the issue.
3.  **Severity:** Error, Warning, or Info.
4.  **Error Code:** A machine-readable ID (e.g., `0xDEADBEEF`).
5.  **Context Data:** Snapshot of relevant state (e.g., register values or sensor readings).

### Logging Best Practices
*   **Persistent Storage:** Dedicate a section of non-volatile memory (NOR Flash or EEPROM) as a circular buffer for error logs. 
*   **Overflow Behavior:** When the log is full, the system must decide whether to stop logging or overwrite the oldest entries. For debugging, overwriting the oldest is usually preferred.
*   **Log on Boot:** The first thing a system should do after a reset is check the error log. If a crash occurred, log the **Reset Reason** and any available **Coredump** before starting the main application logic.

## Metrics Collection

Track the health indicators that matter before the device starts failing:

*   **Stack high-water mark:** Are any tasks approaching their stack limit?
*   **Heap fragmentation:** Is heap usage growing without being freed?
*   **Task CPU utilization:** Which task is consuming more CPU than expected?
*   **Error counters:** How often is a peripheral returning an error before succeeding?

Expose these via a lightweight protocol (MQTT, CoAP, or a custom BLE characteristic) and visualize them in Grafana or a similar dashboard.

---

## The Post-Market Health Shift

Modern firmware is never "done" just because it shipped. It is a **Software-Defined Asset** that must be monitored throughout its lifecycle.

### Fleet-Scale Metrics
To maintain architectural verticality, monitor your fleet using high-level health indicators:
*   **Crash-Free Hours:** The primary stability metric. Track the mean time between software failures (MTBSF) across the entire fleet.
*   **Battery Lifecycle Degradation:** Monitor internal resistance and depth-of-discharge profiles. This data allows you to push power-optimization updates *before* the physical hardware reaches its end-of-life.
*   **Shadow Mode Verification:** Run new algorithms in the background on fielded devices. Compare the "predicted" output of the new code against the "actual" output of the current production code.

---

---

## Mental Model: The Always-On Auditor (Tracing vs. Logging)

To understand a complex multi-tasking system, you must distinguish between the "Facts" and the "Flow."

*   **Logging (The Facts):** Logging tells you *what* happened (e.g., "Sensor connection failed"). It is essential for auditing and post-mortem analysis.
*   **Tracing (The Flow):** Tracing tells you *where* time was spent and *how* the system moved between states. It captures the latency between an interrupt triggering and a task responding.
*   **The Auditor Strategy:** An elite system treats observability as an "Auditor" that is always on. Do not wait for a bug to enable tracing. Maintain a circular trace buffer in RAM that is always running. When a fault occur, this buffer provides the "Black Box" flight data recorder of the final milliseconds before the crash.

---

## The Observability Rule
> "If you cannot tell whether your system is healthy without physical access, it is not production-ready."

Design observability in from day one. Adding it after is always more expensive.
