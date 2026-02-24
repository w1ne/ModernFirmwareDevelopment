# Dependability and System Resilience

If your system cannot tell you how reliable it is, it has no reliability goal. And without a goal, reliability is whatever happens at the end of testing.

## The Fault, Error, Failure Chain

Three distinct events happen before a user sees a problem:

1. **Fault:** A defect exists. A corrupted bit in RAM. A buggy code path.
2. **Error:** The fault activates. The corrupted bit is read and used in a computation.
3. **Failure:** The error propagates into observable misbehavior.

A dependable system tolerates faults before they become errors, and contains errors before they become failures. This framing is useful because it shows there are three distinct places to intervene.

## Reliability vs. Availability

These are not the same metric. Use the right one for your system.

**Reliability** is the probability the system operates completely failure-free for a specific mission time. It decays exponentially as mission time grows.

**Availability** is the fraction of time the system is operational. It depends on MTBF and recovery time: `Availability = MTBF / (MTBF + MTTR)`.

| Scenario | Use Metric |
|---|---|
| Aircraft engine must survive a 12-hour flight | Reliability |
| Industrial gateway must be up 99.9% of the year | Availability (five nines = 315 seconds downtime/year) |
| Medical pump must never fail during a 72-hour infusion | Reliability |
| Field sensor reports every 10 minutes | Availability |

## Software Fails Differently Than Hardware

Hardware reliability math assumes **random independent faults**. Two identical resistors in parallel fail independently. If one fails, the other keeps working.

Software has **correlated faults**. Two copies of the same software given the same inputs produce the same crash. Running two redundant controllers with identical firmware gives you hardware redundancy but zero software fault tolerance.

This is why:
- Running the same RTOS image on two cores does not eliminate software bugs
- Diverse software (different implementations of the same function) helps, but it is expensive and difficult to validate
- The most reliable path is one high-quality version, not two average ones

## Improving Software Dependability

Three strategies work:

**1. High-quality process.** Good process reduces defect density. It does not guarantee zero defects. CI/CD with static analysis, peer review, and branch coverage targets measurably reduces shipped defects.

**2. State scrubbing.** Periodic maintenance reboots clear accumulated corruption (memory leaks, pointer corruption, counter rollover). Schedule them at low-activity times. In Zephyr, use the `sys_reboot()` API with a boot counter to track frequency.

**3. Task supervision.** Monitor each RTOS task for deadlock or starvation. If a task misses its watchdog kick, restart it. This is the pattern Zephyr's `WDT` driver supports when combined with a software supervisor task.

## Graceful Degradation

The goal is not "never fail." The goal is "fail gracefully." Define how each component's failure affects system behavior:

| Component Failure | System Behavior |
|---|---|
| Cloud connectivity lost | Buffer locally, continue operating, retry on reconnect |
| GPS signal lost | Dead-reckoning for up to 60 seconds, alert operator |
| Sensor timeout | Use last known value for 5 seconds, then safe state |
| MCU reset | Resume from last persisted state in flash within 3 seconds |

Document these in your requirements. Test them by injecting failures in your CI hardware-in-the-loop setup.

---

## Real-World Case Studies: Dependability

### The Cost of Architectural Hubris (Negative Example)
In the mid-1980s, the **Therac-25** radiation therapy machine caused catastrophic injuries and deaths. The system relied entirely on software for safety, removing the physical interlocks used in previous models.

*   **The Error:** A race condition existed in the user interface code. If an operator corrected a mistake in under 8 seconds, the system could enter a lethal state where it delivered a raw electron beam without the necessary target shield in place. 
*   **The Data:** Patients received **100x the lethal dose** of radiation. The software was reused from the Therac-20, where physical hardware interlocks had previously masked the race condition bug.
*   **The Lesson:** Software must never be the sole gatekeeper for high-energy physical safety. Dependability requires hardware-software defense in depth.

### The Success of Formal Mathematical Verity (Positive Example)
High-assurance systems like the **seL4 microkernel** use formal verification to prove that the software matches its specification.

*   **The Resilience:** Proof assistants (like Isabelle/HOL) are used to mathematically verify that the implementation contains zero buffer overflows, zero null pointer dereferences, and zero data races.
*   **The Data:** In the verified code paths, the bug density is **0 bugs per thousand lines of code (kLOC)**. This is a 100x improvement over the industry-standard average of 1-10 bugs per kLOC for production code.
*   **The Lesson:** For critical kernels, mathematical proof is the ultimate dependability goal. It moves the system from "likely correct" to "provably secure."

---

## The Dependability Plan

Every product needs one. It does not need to be long. It needs to answer:

1. What does "dependable" mean for this product? (Define MTBSF, availability target)
2. What are the most likely failure modes? (List them; use FMEA if safety-relevant)
3. How does each failure mode affect the user?
4. What is the mitigation for each failure mode?
5. How will you measure whether you are meeting the target in the field?

> [!TIP]
> Field data from deployed units is the only real measure of dependability. Add an error log that records every reset with a reason code. Aggregate this data. Compare it to your written target.
