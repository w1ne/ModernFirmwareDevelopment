# Continuous Delivery and OTA

Shipping firmware used to mean physical access to every device. OTA (Over-The-Air) updates change that. But an OTA system that ships broken firmware to 10,000 devices is worse than no OTA at all.

## The Failure Modes of Naive OTA

1.  **No signature verification:** An attacker intercepts the update and replaces the firmware with malicious code. The device accepts it.
2.  **No rollback:** A power loss mid-update leaves the device with a half-written image. It is now a brick.
3.  **Big-bang rollout:** A bad update goes to every device simultaneously. Your entire fleet is broken.

Modern OTA engineering eliminates all three by using a robust [Security](file:///home/andrii/Projects/Proemion/ModernFirmwareDevelopment/src/advanced/security.md) architecture.

## The A/B Partition Model

The safest OTA architecture requires two firmware partitions: **Slot A** (active) and **Slot B** (pending).

1.  The device downloads the new image into **Slot B** while running from **Slot A**.
2.  The bootloader verifies the signature of **Slot B**.
3.  The bootloader sets a "try next" flag and reboots into **Slot B**.
4.  If the application in **Slot B** boots successfully and marks itself as "confirmed," **Slot B** becomes the new active slot.
5.  If Slot B fails to boot within a timeout, the bootloader automatically rolls back to **Slot A**.

This model guarantees the device always has a runnable firmware image, regardless of what happens to the update.

## The Bootloader: MCUboot

**MCUboot** is the standard open-source bootloader for embedded OTA. It supports:
*   **Image signing** (RSA, ECDSA) to verify authenticity.
*   **A/B swap** with automatic rollback.
*   **Encrypted images** for confidentiality in transit and at rest.

Zephyr and Mynewt both support MCUboot natively.

## Staged Rollout

Never send an update to your entire fleet at once.

1.  **Canary (1%):** Deploy to 1% of devices first. Watch crash rates and telemetry for 24-48 hours.
2.  **Ring 2 (10%):** Expand if metrics are healthy.
3.  **Full Rollout (100%):** Only when confidence is high.

A good OTA system evaluates device health automatically and pauses rollout if error rates exceed a threshold.

---

## Real-World Case Studies: Continuous Delivery

### The Cost of Manual Deployment (Negative Example)
In 2012, **Knight Capital Group** lost **$440 million** in just 45 minutes. The disaster was caused by a manual deployment error that reactivated retired code.

*   **The Error:** Technicians manually copied new software to eight routing servers. They missed the eighth server. This single server continued running legacy code that had been repurposed for a new function. When the markets opened, the legacy "Power Peg" code triggered an uncontrolled loop of erroneous trades.
*   **The Data:** The system executed 4 million trades in 45 minutes. The firm nearly went bankrupt before the server was identified and shut down.
*   **The Lesson:** Manual deployment is a high-risk failure point. Continuous delivery must be automated, audited, and strictly consistent across every node in the fleet.

### The Success of Fleet-Scale OTA (Positive Example)
In late 2023, **Tesla** performed a software recall for over **2 million vehicles** to update Autosteer safety features.

*   **The Resilience:** Instead of requiring physical service center visits for 2,000,000 cars, the company deployed the fix via encrypted OTA updates. 
*   **The Data:** The recall was completed with near-zero logistics cost. A physical recall of this scale would cost approximately **$2 billion** in dealer labor and administrative overhead.
*   **The Lesson:** OTA transforms hardware maintenance into a software-defined asset. It allows for rapid, fleet-wide security and safety patches that would be physically impossible in traditional manufacturing.

---

## The Delta Update

Full binary images are large. A **delta update** ships only the difference between the current version and the new version. Tools like `bsdiff` or SUIT (IETF Software Update for Internet of Things) generate minimal patches, reducing update payload by 80-90%.

This matters on constrained devices with cellular connections where data costs money and update time equals power consumption.
