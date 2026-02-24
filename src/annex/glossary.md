# Technical Glossary

A dictionary for the modern firmware engineer.

| Term | Definition |
| :--- | :--- |
| **ADAS** | Advanced Driver Assistance Systems. Systems developed to automate, adapt, and enhance vehicle systems for safety and better driving. |
| **ADR** | Architecture Decision Record. A short, version-controlled document that captures an architectural decision and its rationale. |
| **Block Diagram** | A high-level representation of a system's physical and logical components, used to identify hardware dependencies and shared resource red flags. |
| **Brownout Simulation** | A verification technique where a programmable power supply drops voltage to marginal thresholds to test system resilience and memory protection. |
| **CAN Bus** | Controller Area Network. A robust vehicle bus standard designed to allow microcontrollers and devices to communicate with each other's applications without a host computer. |
| **CI/CD** | Continuous Integration and Continuous Deployment. A set of operating principles and practice that enable software development teams to deliver code changes more frequently and reliably. |
| **Cognitive Load** | The total amount of mental effort being used in the working memory. In firmware, engineering organizations aim to reduce load via platform teams and clear abstractions. |
| **Context Diagram** | An architectural overview showing how a system fits into the world, focusing on interactions with users, servers, and other external entities. |
| **Conway's Law** | The observation that organizations design systems which mirror their own communication structures. |
| **CRA** | Cyber Resilience Act. EU regulation that sets mandatory cybersecurity requirements for products with digital elements. |
| **DI** | Dependency Injection. A design pattern where a module's dependencies are provided ("injected") at runtime, typically via function pointers, to improve testability. |
| **DORA** | DevOps Research and Assessment. A set of metrics (Deployment Frequency, Lead Time, MTTR, Change Failure Rate) used to measure software delivery performance. |
| **DTS** | Devicetree. A data structure used to describe hardware components to an operating system (common in Zephyr and Linux). |
| **DVFS** | Dynamic Voltage and Frequency Scaling. A technique to reduce CPU power consumption by lowering clock speed and voltage during low-activity periods. |
| **Fleet Health** | A set of high-level metrics (e.g., Crash-Free Hours, Battery Degradation) used to monitor the wellness of a deployed population of devices. |
| **FMEA** | Failure Mode and Effects Analysis. A systematic process for identifying potential failure modes in a system and assessing their impact. |
| **HAL** | Hardware Abstraction Layer. A software layer that provides a hardware-agnostic interface to device peripherals. |
| **HIL** | Hardware-in-the-Loop. A technique where real hardware is tested against a real-time simulation of the physical system it controls. |
| **HSM** | Hardware Security Module. A physical device that provides tamper-resistant storage and processing of cryptographic keys. |
| **ISO 26262** | An international standard for functional safety of electrical and/or electronic systems that are installed in serial production road vehicles. |
| **ISR** | Interrupt Service Routine. A function called by the hardware in response to an interrupt event, outside of the normal program flow. |
| **IVT** | Interrupt Vector Table. A fixed array of function pointers at the start of flash that the CPU reads on power-up to find the reset handler and all ISRs. |
| **LPWAN** | Low-Power Wide Area Network. A family of wireless technologies (LoRaWAN, NB-IoT) designed for long-range, low-bandwidth IoT communication. |
| **LTO** | Link-Time Optimization. A compiler technique that performs cross-module optimizations at the final link stage, reducing code size and improving speed. |
| **lwIP** | Lightweight IP. An open-source, memory-efficient TCP/IP stack commonly used in embedded systems for IPv4/IPv6 networking. |
| **MCU** | Microcontroller Unit. A small computer on a single integrated circuit. |
| **MCUboot** | An open-source secure bootloader for IoT devices that supports image signing, A/B partition swap, and automatic rollback. |
| **Mock** | A type of test double that records interactions and enforces expectations about how it is called (e.g., number of calls, specific arguments). |
| **MPU** | Memory Protection Unit. A hardware component that enforces memory access rules (e.g., read-only, no-execute) between regions of memory. |
| **MTBSF** | Mean Time Between Software Failures. A metric used to measure the reliability of a software system. |
| **Mutual Exclusion (Mutex)** | A software mechanism that ensures only one task can access a shared resource (memory or hardware) at a time, preventing race conditions. |
| **no_std** | A Rust attribute that indicates a crate does not link to the standard library, essential for bare-metal firmware. |
| **Organigram** | A hierarchical diagram of software modules, used to visualize control flow and shared resource dependencies. |
| **OTA** | Over-The-Air. The process of updating firmware on a deployed device wirelessly, without physical access. |
| **PAC** | Peripheral Access Crate. A Rust crate generated from an SVD file that provides low-level access to a microcontroller's peripherals. |
| **PendSV** | Pendable Service Request. A special ARM Cortex-M interrupt used exclusively by RTOS kernels to trigger context switches. |
| **Platform Team** | A team that provides internal tools and services (BSP, HAL, CI) to reduce the cognitive load of stream-aligned teams. |
| **Priority Inversion** | A scheduling anomaly where a high-priority task is blocked by a low-priority task holding a shared mutex. Resolved via priority inheritance. |
| **PSA** | Platform Security Architecture. An ARM framework for security evaluation and certification of IoT devices (PSA Certified). |
| **Race Condition** | A concurrency bug where the system's correctness depends on the non-deterministic timing or order of events. |
| **RMS** | Rate Monotonic Scheduling. A scheduling policy where tasks with shorter periods are assigned higher priorities. |
| **SAST** | Static Application Security Testing. Automated analysis of source code to find security vulnerabilities and bugs before the code is run. |
| **SBOM** | Software Bill of Materials. A nested inventory, a list of ingredients that make up software components. |
| **SPI** | Serial Peripheral Interface. A synchronous, full-duplex serial communication bus used for short-distance communication with peripherals. |
| **Spy** | A type of test double that records indirect outputs (state changes or function calls) so the test can verify them after execution. |
| **Stream-aligned Team** | A team organized around a continuous flow of work from a specific functional area or user outcome. |
| **Stub** | A type of test double that provides "canned" indirect inputs to the code under test (e.g., a fixed sensor reading). |
| **SVD** | System View Description. An XML-based format that describes the peripherals and registers of a microcontroller. |
| **System Latency** | The time it takes for a system to respond to an asynchronous event (e.g., an interrupt). High latency can break real-time guarantees. |
| **TCB** | Task Control Block. A data structure the OS uses to save and restore the CPU state (registers, stack pointer) of a task during context switches. |
| **TDD** | Test-Driven Development. A development microcycle (Red-Green-Refactor) where tests are written before the production code. |
| **Test Double** | A generic term for any object that replaces a real production component for testing purposes (e.g., Stubs, Mocks, Spies). |
| **TrustZone** | An ARM hardware security feature (available on Cortex-M33+) that divides the processor into Secure and Non-Secure worlds. |
| **VEX** | Vulnerability Exploitability eXchange. A machine-readable document used to state whether a specific vulnerability in an SBOM is actually exploitable in a product. |
| **WCET** | Worst-Case Execution Time. The maximum amount of time a task takes to execute on a specific hardware platform. |
| **XIP** | Execute in Place. The ability of an MCU to execute code directly from flash memory without copying it to RAM first. |
