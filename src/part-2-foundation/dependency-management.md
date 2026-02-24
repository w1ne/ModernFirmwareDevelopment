# Dependency Management

> "Dependencies are like house guests: some are helpful, some are annoying, and all of them need to be managed or they'll take over your life."

## The Hook: The Hidden Cost of "Copy-Paste"
In the "Horizontal" world, dependencies are managed by copying a `.c` and `.h` file into a `drivers/` folder and forgetting about them. Three years later, you realize you have 16 different versions of a CRC library and zero way to patch a security vulnerability. 

**Elite Dependency Management** focuses on three pillars: **Reproducibility**, **Auditability**, and **Vulnerability Tracking**. 

By 2027, a **Software Bill of Materials (SBOM)** is no longer an "extra" provided for compliance; it is the fundamental ledger you use to manage your project. If it's not in the SBOM, it doesn't belong in the binary.

## The Theory: Vendoring vs. Remote Fetching
There is a fundamental tension in firmware between stability and security.

| Strategy | Pros | Cons |
| :--- | :--- | :--- |
| **Vendoring** (Check-in) | Guaranteed offline builds; stable. | "Security Debt"; repo bloat. |
| **Remote Fetching** (On-demand) | Easy updates; lean repo. | Build breaks if server is down; non-deterministic. |

### The "Elite" Compromise: Manifests + Lockfiles
Modern tools allow you to fetch remotely but freeze the version locally using a **Lockfile**.
*   **Rust:** Uses `Cargo.lock` to pin every crate to a specific hash.
*   **Zephyr:** Uses `west.yml` to define exactly which Revision (SHA) of a repo to pull.

---

## Breaking Internal Dependencies

While external dependencies (libraries) are managed by package managers, **internal dependencies** (how your modules talk to each other) must be managed by design. If your application logic directly includes `hardware_registers.h`, you cannot test it on a host.

### The Problem: The "Welded" Architecture
In many legacy systems, every file includes every other file. This creates a "Spaghetti and Meatball" structure where you cannot compile a single function without including the entire project.

### The Solution: Substitution
To break a dependency, you must be able to swap a real requirement for a test double.

1.  **Link-Time Substitution:**
    *   **The Strategy:** The application calls `UART_Send()`. In production, you link `uart_stm32.c`. In testing, you link `uart_mock.c`.
    *   **The Key:** Both files must implement the exact same function signature defined in `uart.h`.
    *   **Advantage:** Zero runtime overhead. Perfect for highly constrained systems.

2.  **Runtime Substitution (Function Pointers):**
    *   **The Strategy:** A module is initialized with a pointer to a function or a `struct` of pointers.
    *   **The Key:** The module never calls a global function; it only calls through its internal pointers.
    *   **Advantage:** Allows you to change behavior during a test (e.g., making a mock UART fail halfway through a transmission).

### Dependency Injection in C
Do not let your modules "go shopping" for their dependencies. Provide them during initialization.
```c
// Instead of this:
void Logger_Init(void) {
    I2C_Init(); // Hardcoded dependency
}

// Do this:
void Logger_Init(I2C_Interface *interface) {
    this->i2c = interface; // Injected dependency
}
```

---

---

## The Implementation: The West Manifest (Zephyr)
If you are moving to a multi-repo or OS-based architecture, you need a manifest-based tool. Here is a simplified `west.yml` example for an Elite project:

```yaml
manifest:
  remotes:
    - name: ncs
      url-base: https://github.com/nrfconnect
    - name: my-company
      url-base: https://github.com/my-firmware-org

  projects:
    - name: zephyr
      remote: ncs
      revision: v2.7.0  # Strict Pinning
    - name: custom-drivers
      remote: my-company
      revision: main
      path: drivers/proprietary

  self:
    path: application
```

### Pro Tip: The "Audit" Command
Use tools like `cargo audit` (Rust) or `west list` (Zephyr) to generate a **Software Bill of Materials (SBOM)**. This is a list of every third-party component in your firmware, which is now a requirement for many safety certifications (ISO 26262) and governmental regulations (EU Cyber Resilience Act).

---

## Practical: The 3-Line SBOM Command

Generating an SBOM should be an automated step in your CI pipeline.

### For Rust Projects
Use `cargo-cyclonedx` to generate a machine-readable SBOM in JSON or XML format.
```bash
cargo install cargo-cyclonedx
cargo cyclonedx --format json
# Result: target/cyclonedx/vulnerability-report.json
```

### For Zephyr Projects
Use the `west` tool to export the project manifest.
```bash
west manifest --freeze > sbom.yml
west list --format "{name} {revision} {path}" >> sbom.txt
# Result: Comprehensive list of all repositories and versions
```

---

---

## Real-World Case Studies: Dependency Integrity

### The Cost of Invisible Dependencies (Negative Example)
In 2021, the **Log4Shell** vulnerability (Log4j) exposed the critical danger of transition dependencies in the global software supply chain.

*   **The Error:** A vulnerability in a common Java logging library allowed remote code execution (RCE). Because Java is used in over 3 billion devices, including millions of IoT and industrial systems, the impact was nearly universal.
*   **The Data:** Research showed that over **60%** of technology suppliers used Log4j as an **indirect dependency**. The vulnerable code was often found **5 to 9 levels deep** in dependency graphs, making it invisible to manual audits.
*   **The Lesson:** You are responsible for every line of code in your binary, even the lines you didn't write. Automated dependency scanning and a complete **SBOM** are the only ways to defend against the technical debt of your supply chain.

---

## References & Further Reading
*   [West: Zephyr's Meta-Tool Documentation](https://docs.zephyrproject.org)
*   [Vendoring vs. Remote Dependencies - ActivaState Analysis](https://www.activestate.com)
*   [Introduction to Software Bill of Materials (SBOM)](https://www.cisa.gov/sbom)
*   [Cargo: Dependency Management for Rust](https://doc.rust-lang.org/cargo/)
