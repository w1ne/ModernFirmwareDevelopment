# Measuring Performance (DORA)

> "If you cannot measure it, you cannot improve it." , Lord Kelvin

## The End of Maturity Models

Traditional firmware measured performance by recording bugs found in QA or counting lines of code written. These are vanity metrics. These metrics record activity instead of measuring outcome.

The DORA (DevOps Research and Assessment) research team proved that software delivery performance is defined by four metrics that balance speed and stability. According to the *State of the Connected World* reports, the average firmware update delay is **2.5 years**. In a world of zero-day exploits, that isn't just slow. It is catastrophic.

## Elite vs. Novice Benchmarks
The DORA metrics categorize teams into performance levels. For firmware, the gap between an "Elite" team and a "Novice" team is the difference between market leadership and obsolescence.

| Metric | Elite Performers | Novice Performers |
| :--- | :--- | :--- |
| **Deployment Frequency** | On-demand (Multiple/day) | Less than once per month |
| **Lead Time for Changes** | Less than 24 hours | More than one month |
| **Change Failure Rate** | 0% - 15% | 46% - 60% |
| **Time to Restore (MTTR)** | Less than 1 hour | One week to one month |

These four metrics define your team performance:
- __Deployment Frequency__ Measures how often you release firmware to users. *Firmware Example:* Pushing automated builds to a test fleet nightly versus flashing boards manually once a month.
- __Lead Time for Changes__ Measures the hours a code commit needs to reach the device. *Example:* The time between merging a sensor driver fix and deploying the fix to a production device.
- __Change Failure Rate__ Measures the percentage of releases causing a failure. *Firmware Example:* The percentage of over-the-air (OTA) updates resulting in a bricked device or requiring a physical factory reset.
- __Time to Restore__ Measures the hours you take to recover from a field failure. *Firmware Example:* The time required to diagnose a hard fault from field logs, build a patch, and deploy the fix via OTA.

> [!TIP]
> **Expert Perspective: Memfault on Observability**
> "The first step to making reliable IoT devices is understanding their inherent unreliability... measuring the data points that matter most: crash-free hours, average battery life, and Wi-Fi/BLE connected time."
> *Memfault Leadership*

---

## Case Study: The HP LaserJet Transformation

In 2006, Gary Gruver led a technical transformation at the HP LaserJet firmware division.

- __The Problem__ The team of 400 developers spent 5% of their time on new features. Manual testing and slow integration consumed all other time.
- __The Tactical Shift__ The team removed separate product branches. The team moved to a single main branch for all printers. Developers built a simulation environment to provide instant feedback.
- __The Result__ Time dedicated to writing new features jumped from 5% to 40%. The team moved from two-month manual test cycles to continuous verification. The division reduced development costs by 40%.
- __The DORA Connection__ The team shortened Lead Time and increased Deployment Frequency to simulation.

### The Success of Containerized Pipelines (Positive Example)

In 2017, the global payroll company **ADP** transformed its deployment engineering using Docker and automated CI/CD pipelines.

- __The Shift__ ADP moved from legacy manual infrastructure to a standardized containerized pipeline. This shift provided a Golden Path for every developer to build, test, and ship code through a unified interface.
- __The Data__ Forrester Research reports a 43% productivity gain for enterprises adopting Docker Business. Teams shift focus from manual server configuration to core product logic.
- __The Lesson__ High-performance teams treat their development environment as an automated product. Consistency across the pipeline drives the Change Failure Rate and Lead Time metrics.

## The Formality Foundation: Process as a Safety Net

While DORA focuses on outcomes (Speed and Stability), sustainable firmware development focuses on the **formality** required to reach them. "Elite" performance isn't just about moving fast; it's about having a process that survives the "Beer Truck" test. If a key developer is hit by a beer truck (or simply leaves the project), does the code survive?

### The Paper Threshold

Every distinct development step requires a verifiable record. This requirement means:
- __Requirements as Code__ Gherkin files or tests defining done. *Example:* A unit test case explicitly named `Motor_Stops_When_EStop_Pulled` that fails until the safety logic is implemented.
- __Architecture Decision Records__ Short version-controlled notes explaining reasoning. *Example:* A markdown file in the repository explaining why the team chose Zephyr RTOS over FreeRTOS for the new microcontroller.
- __Peer Reviews__ You need these reviews for knowledge transfer. Reviews reduce Key Person Risk. *Example:* A senior engineer reviewing a junior engineer's pull request for a new DMA configuration to ensure memory safety.

Similarly, skipping process formality creates **technical debt** that kills DORA's Lead Time and Deployment Frequency metrics within 18-24 months.

---

## Designing for the Life of the System

Elite firmware engineering recognizes that a product’s life doesn't end at the "ship" date. It starts there. 

### The Lifecycle Map

1. __Prototyping__ Build a quick prototype to prove the concept.
2. __Board Bring-Up__ Verify the hardware and software handshake.
3. __Manufacturability__ You must plan how to flash 10000 units. Elite software includes Factory Test Modes and serial number provisioning.
4. __Field Life__ The field represents the most expensive phase. You must decide how the system handles a corrupt flash sector or a broken hardware input.
5. __Graceful Degradation__ A robust system fails safely. The system provides information or safe-state defaults to avoid silent failures.

### Software as a Product Lifecycle Tool
Consider which constraints will become problems *later*. If you don't save 20% of your Flash/RAM/CPU cycles during development, you will have zero room to patch security vulnerabilities in the field. 

## The Implementation: Instrumenting Your Pipeline

To move from Novice to Elite, you must stop guessing and start measuring:

- __Visual Dashboards__ Implement tools like GitHub Insights or custom GitLab exporters to make these four metrics visible to the entire engineering organization.

## The GitHub-Driven Pipeline

To achieve "Elite" DORA metrics in firmware, your CI/CD pipeline must move beyond "just compiling code." It must provide **Binary Provenance** and **Release Stability**.

### GitHub Actions Core Patterns

Elite firmware teams automate the deployment flow:
- __The Matrix Strategy__ Compile for every supported hardware target in parallel. Catch platform-specific regressions instantly.
- __Immutable Artifacts__ Upload every successful build as a versioned artifact. Use SHA-256 checksums to guarantee the factory flashed binary matches the tested binary.
- __Self-Hosted Runners__ Run integration tests on real hardware on ARM or RISC-V machines during the pull request.

### Trunk-Based Development in Firmware
The "Trunk-Based" pattern is the single most important driver of **Deployment Frequency** and **Lead Time**.
1.  **Short-Lived Branches** Feature branches should live for hours, not weeks.
2.  **Continuous Integration** Merge to `main` multiple times a day. If the build breaks, the *entire team* stops to fix it.
3.  **Feature Flags** Use compile-time flags to disable incomplete features in the `main` branch. This allows you to ship code without exposing unfinished logic to the device.

## Mental Model: The Inverted Pyramid of Testing

Total system reliability is a function of where you spend your testing effort. 

- __The Traditional Trap__ Many teams spend 80% of their time on manual hardware testing and 20% on automated logic. This imbalance creates the Testing Iceberg. Most effort is manual, slow, and non-reproducible.
- __The Inverted Pyramid__ Modern firmware flips the script:
    1. __Bottom Layer__ 70% Unit Tests. Fast, exhaustive, and runs in seconds.
    2. __Middle Layer__ 20% Integration Tests. Verifies the interaction between modules and virtual peripherals.
    3. __Top Layer__ 10% System Tests. Verifies final physical behavior and electrical compliance on real hardware.
- __The Benefit__ Shifting 90% of your verification off-target ensures the expensive hardware phase validates correct requirements instead of debugging logic.

---
*   *Accelerate: The Science of Lean Software and DevOps* by Nicole Forsgren, Jez Humble, and Gene Kim.
*   [DORA.dev - The Four Keys](https://dora.dev/guides/four-keys/)
*   [World Economic Forum: State of the Connected World Report](https://www.weforum.org)
*   *A Practical Approach to Large-Scale Agile Development* by Gary Gruver.
