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

## References & Further Reading
*   [West: Zephyr's Meta-Tool Documentation](https://docs.zephyrproject.org)
*   [Vendoring vs. Remote Dependencies - ActivaState Analysis](https://www.activestate.com)
*   [Introduction to Software Bill of Materials (SBOM)](https://www.cisa.gov/sbom)
*   [Cargo: Dependency Management for Rust](https://doc.rust-lang.org/cargo/)
