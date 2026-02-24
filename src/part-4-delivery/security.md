# Security & Compliance (CRA)

> "Security is not a feature you add at the end; it is a foundational pillar that determines whether your product is a success or a liability."

## The Hook: The Legal Hammer is Falling
For decades, firmware security was an afterthought. That era is over. The **EU Cyber Resilience Act (CRA)** has turned cybersecurity from a "Best Practice" into a "Legal Requirement." 

If your connected device is not **Secure by Design**, you can no longer sell it in Europe. Non-compliance results in fines up to **€15 million** or 2.5% of global revenue.

## The Theory: The Compliance Pillar

The Cyber Resilience Act (CRA) mandates three critical "Vertical" shifts for firmware teams. Compliance is not just a checkbox; it is a mechanism for building high-trust products.

1.  **Security by Default:** No hardcoded passwords. Encrypted communication by default. Minimized attack surface.
2.  **The SBOM and VEX Ecosystem:** You must provide a **Software Bill of Materials (SBOM)** accompanied by a **Vulnerability Exploitability eXchange (VEX)** document. VEX allows you to declare that a known library vulnerability is not exploitable because the vulnerable code path is never reachable in your specific firmware configuration.
3.  **The 5-Year Lifecycle:** You are legally obligated to provide security updates for a minimum of 5 years. Elite teams turn this into a business model by using software-defined architectures that are easy to patch remotely without hardware revisions.

---

---

## Case Study: The Jeep Takeover (Lateral Movement)
In 2015, researchers remotely took over a Jeep Cherokee, controlling steering and brakes while it was on a highway.

*   **The Flaw:** They didn't hack the brakes directly. They hacked the **Infotainment system** (connected to the web) and used a "Lateral Movement" to send commands to the vehicle's CAN bus.
*   **The Lesson:** Security is only as strong as your weakest node. If your "unimportant" UI processor is connected to the web, your safety-critical flight controller is at risk.

---

## The Implementation: Hardware Root of Trust

Elite security doesn't rely on software alone. It anchors secrets in the hardware using dedicated security subsystems.

### Secure Boot Chain

A secure boot chain validates every stage of the boot process before execution.

1.  **ROM Bootloader (Immutable):** The first-stage bootloader lives in read-only ROM. It cannot be tampered with. It verifies the signature of the second-stage bootloader.
2.  **Second-Stage Bootloader:** Verifies the application image's signature using an asymmetric key pair. The private key never leaves the factory. The public key lives in OTP (One-Time Programmable) flash.
3.  **Application:** Only executes if the entire chain of signatures is valid.

If any stage fails verification, the device refuses to boot.

### ARM TrustZone (Cortex-M33+)

TrustZone divides the processor into two worlds: **Secure** and **Non-Secure**.

*   **Secure World:** Holds cryptographic keys, Secure Boot logic, and trusted services. Non-Secure code can never read this memory region.
*   **Non-Secure World:** Runs your application firmware. It requests sensitive operations via the PSA Crypto API.

This hardware boundary ensures that even if the application firmware is fully compromised, the attacker cannot extract the device's private key.

### ARM Platform Security Architecture (PSA)

**PSA Certified** (Level 1, 2, 3) gives customers a verifiable guarantee your device meets a baseline security standard. In regulated industries, this certification is increasingly a procurement requirement.

### Hardware Security Modules (HSMs)

For industrial and automotive use cases, a dedicated **HSM** (e.g., Infineon SLx) provides a physically separate, tamper-resistant environment for all cryptographic operations. The main CPU never sees the private key; it only sees the output.

### TPM 2.0: Measured Boot (PCRs)
Instead of just checking a signature, a TPM creates an incorruptible record of the boot process.
*   **The Identity:** Every stage of the bootloader "measures" the next stage (hashing the code).
*   **The Gate:** If the code has been tampered with, the TPM will refuse to release the encryption keys needed to boot the application. This is a "Zero to One" shift from *hoping* it's safe to *proving* it's safe.

### 2. Automated Scanning
Your CI pipeline must act as your security guard. 
*   **Step 1:** Generate a `CycloneDX` SBOM using `cdxgen`.
*   **Step 2:** Scan it against the **Global CVE Database** using `Trivy` or `Grype`.
*   **Step 3:** If a high-severity vulnerability is found in a library, the build fails.

## References & Further Reading
*   [EU Cyber Resilience Act (CRA) - Official Text](https://commission.europa.eu)
*   [CycloneDX: The Security-Focused SBOM Standard](https://cyclonedx.org)
*   [The Jeep Hack: A Milestone in Automotive Security](https://www.wired.com)
*   [Trusted Computing Group: TPM 2.0 Library Specification](https://trustedcomputinggroup.org)
