# Security & Compliance (CRA)

> "Security is not a feature you add at the end; it is a foundational pillar that determines whether your product is a success or a liability."

## The Hook: The Legal Hammer is Falling
For decades, firmware security was an afterthought. That era is over. The **EU Cyber Resilience Act (CRA)** has turned cybersecurity from a "Best Practice" into a "Legal Requirement." 

If your connected device is not **Secure by Design**, you can no longer sell it in Europe. Non-compliance results in fines up to **€15 million** or 2.5% of global revenue.

## The Theory: The Compliance Pillar
The CRA mandates three critical "Vertical" shifts for firmware teams:
1.  **Security by Default:** No hardcoded passwords, encrypted communication by default, and a minimized attack surface.
2.  **Mandatory SBOM & VEX:** You must provide an SBOM accompanied by a **Vulnerability Exploitability eXchange (VEX)** document. VEX allows you to state that even if a library has a known vulnerability, your firmware is not exploitable because the vulnerable function is never called.
3.  **The 5-Year Window:** You are legally obligated to provide security updates for the expected lifetime of the product (minimum 5 years).

---

## Case Study: The Jeep Takeover (Lateral Movement)
In 2015, researchers remotely took over a Jeep Cherokee, controlling steering and brakes while it was on a highway.

*   **The Flaw:** They didn't hack the brakes directly. They hacked the **Infotainment system** (connected to the web) and used a "Lateral Movement" to send commands to the vehicle's CAN bus.
*   **The Lesson:** Security is only as strong as your weakest node. If your "unimportant" UI processor is connected to the web, your safety-critical flight controller is at risk.

---

## The Implementation: Hardware Root of Trust
Elite security doesn't rely on software alone. It moves the "Secret" into the hardware using a **TPM 2.0 (Trusted Platform Module)**.

### 1. Measured Boot (PCRs)
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
