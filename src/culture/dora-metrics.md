# Measuring Performance (DORA)

> "If you cannot measure it, you cannot improve it." , Lord Kelvin

## The Hook: The End of "Maturity Models"
In traditional firmware, we measured "Performance" by how many bugs we found in QA or how many lines of code we wrote. These are vanity metrics. They measure activity, not outcome.

The DORA (DevOps Research and Assessment) research team proved that software delivery performance is defined by four metrics that balance speed and stability. According to the *State of the Connected World* reports, the average firmware update delay is **2.5 years**. In a world of zero-day exploits, that isn't just slow. It is catastrophic.

## The Theory: Elite vs. Novice Benchmarks
The DORA metrics categorize teams into performance levels. For firmware, the gap between an "Elite" team and a "Novice" team is the difference between market leadership and obsolescence.

| Metric | Elite Performers | Novice Performers |
| :--- | :--- | :--- |
| **Deployment Frequency** | On-demand (Multiple/day) | Less than once per month |
| **Lead Time for Changes** | Less than 24 hours | More than one month |
| **Change Failure Rate** | 0% - 15% | 46% - 60% |
| **Time to Restore (MTTR)** | Less than 1 hour | One week to one month |

> [!TIP]
> **Expert Perspective: Memfault on Observability**
> "The first step to making reliable IoT devices is understanding their inherent unreliability... measuring the data points that matter most: crash-free hours, average battery life, and Wi-Fi/BLE connected time."
> *Memfault Leadership*

---

## Case Study: The HP LaserJet Revolution (Success)
In 2006, Gary Gruver led one of the most successful technical transformations in history at HP’s LaserJet firmware division.

*   **The Problem:** The team of 400+ developers was spending only **5% of its time on new features**. The rest was lost to manual testing and slow integration.
*   **The Tactical Shift:**
    *   **Trunk-Based Development:** They killed separate product branches and moved to a single "main" branch for all printers.
    *   **Total Automation:** They built a massive simulation environment to provide instant feedback to every dev.
*   **The Result:** A **200% increase in productivity**. They moved from 6-week manual test cycles to continuous verification.
*   **The DORA Connection:** By shortening **Lead Time** and increasing **Deployment Frequency** (to simulation), they freed up 40% of their budget for innovation.

### The Success of Containerized Pipelines (Positive Example)
In 2017, the global payroll company **ADP** transformed its deployment engineering using Docker and automated CI/CD pipelines.

*   **The Shift:** ADP moved from legacy manual infrastructure to a standardized, containerized pipeline. This provided a "Golden Path" for every developer to build, test, and ship code through a unified interface.
*   **The Data:** The transition resulted in a **40% increase in developer productivity**. Lead times dropped from weeks to minutes. Teams shifted their focus from manual server configuration to core product logic.
*   **The Lesson:** High-performance teams treat their development environment as an automated product. Consistency across the pipeline is the primary driver of the **Change Failure Rate** and **Lead Time** metrics.

---

---

## The Formality Foundation: Process as a Safety Net

While DORA focuses on outcomes (Speed and Stability), sustainable firmware development focuses on the **formality** required to reach them. "Elite" performance isn't just about moving fast; it's about having a process that survives the "Beer Truck" test. If a key developer is hit by a beer truck (or simply leaves the project), does the code survive?

### The "Paper" Threshold
Every distinct development step should produce a verifiable record. This does not mean 500-page PDFs. It means:
*   **Requirements as Code:** Gherkin files or tests that define "done."
*   **Architecture Decision Records (ADRs):** Short, version-controlled notes on *why* a decision was made.
*   **Peer Reviews:** Not just for quality, but for **knowledge transfer** (reducing "Key Person Risk").

Similarly, skipping process formality creates **technical debt** that kills DORA's Lead Time and Deployment Frequency metrics within 18-24 months.

---

## Designing for the Life of the System

Elite firmware engineering recognizes that a product’s life doesn't end at the "ship" date. It starts there. 

### The Lifecycle Map
1.  **Prototyping:** Hack it together to prove the concept.
2.  **Board Bring-Up:** Verify the hardware-software handshake.
3.  **Manufacturability:** How do we flash 10,000 units? Elite software includes **Factory Test Modes** and serial number provisioning.
4.  **Field Life:** The most expensive phase. How does it handle a corrupt flash sector or a "broken" hardware input?
5.  **Graceful Degradation:** A robust system should fail in a way that provides information or safe-state defaults, rather than silent catastrophic failure.

### Software as a Product Lifecycle Tool
Consider which constraints will become problems *later*. If you don't save 20% of your Flash/RAM/CPU cycles during development, you will have zero room to patch security vulnerabilities in the field. 

---

---

## The Implementation: Instrumenting Your Pipeline
To move from "Novice" to "Elite," you must stop guessing and start measuring:

4.  **Visual Dashboards:** Use tools like GitHub Insights or custom GitLab exporters to make these four metrics visible to the entire engineering org.

---

## Elite Benchmarks: The GitHub-Driven Pipeline

To achieve "Elite" DORA metrics in firmware, your CI/CD pipeline must move beyond "just compiling code." It must provide **Binary Provenance** and **Release Stability**.

### GitHub Actions Core Patterns
Elite firmware teams utilize GitHub Actions to automate the 0 to 1 flow:
*   **The Matrix Strategy:** Compile for every supported hardware target (Zephyr overlays, Rust features) in parallel to catch platform-specific regressions instantly.
*   **Immutable Artifacts:** Upload every successful build as a versioned artifact. Use SHA-256 checksums to ensure the binary flashed in the factory is identical to the one tested in CI.
*   **Self-Hosted Runners:** Use self-hosted GitHub runners (ARM or RISC-V machines) to run integration tests on real hardware as part of the PR check.

### Trunk-Based Development in Firmware
The "Trunk-Based" pattern is the single most important driver of **Deployment Frequency** and **Lead Time**.
1.  **Short-Lived Branches:** Feature branches should live for hours, not weeks.
2.  **Continuous Integration:** Merge to `main` multiple times a day. If the build breaks, the *entire team* stops to fix it.
3.  **Feature Flags:** Use compile-time flags to disable incomplete features in the `main` branch. This allows you to ship code without exposing unfinished logic to the device.

---

## Mental Model: The Inverted Pyramid of Testing

Total system reliability is a function of where you spend your testing effort. 

*   **The Traditional Trap:** Many teams spend 80% of their time on manual hardware testing and 20% on automated logic. This is the **Testing Iceberg**. Most of the effort is underwater (manual), slow, and non-reproducible.
*   **The Inverted Pyramid:** Modern firmware flips the script:
    1.  **Bottom Layer (Base):** 70% Unit Tests (Host-based). Fast, exhaustive, and runs in seconds.
    2.  **Middle Layer:** 20% Integration/Simulation Tests (Renode/QEMU). Verifies the interaction between modules and virtual peripherals.
    3.  **Top Layer (Peak):** 10% System/HIL Tests (Real Hardware). Verifies final physical behavior and electrical compliance.
*   **The Benefit:** By pushing 90% of your verification to the "Off-Target" base, you ensure that the expensive hardware phase is for **validation** (did we build the right thing?), not **debugging** (did we build it correctly?).

---
*   *Accelerate: The Science of Lean Software and DevOps* by Nicole Forsgren, Jez Humble, and Gene Kim.
*   [DORA.dev - The Four Keys](https://dora.dev/guides/four-keys/)
*   [World Economic Forum: State of the Connected World Report](https://www.weforum.org)
*   *A Practical Approach to Large-Scale Agile Development* by Gary Gruver.
