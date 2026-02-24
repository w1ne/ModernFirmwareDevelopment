# The Boot Sequence

Understanding how a microcontroller boots is not optional. It is the foundation on which every line of your application code runs.

## The Interrupt Vector Table (IVT)

Before `main()`, the processor runs the **reset handler**, which it finds via the Interrupt Vector Table. This table is a fixed array of function pointers stored at the very beginning of flash (address `0x00000000` on ARM).

The two most critical entries:
*   **Word 0 (0x00):** The initial value of the Main Stack Pointer (MSP). The CPU loads this into the stack pointer register on power-up.
*   **Word 1 (0x04):** The address of the reset handler. The CPU jumps here immediately.

The linker script places `.isr_vector` at the front of the `.text` section to guarantee these appear at address zero.

## The Reset Handler

`isr_reset` runs before `main()`. It performs three essential steps:

1.  **Copy `.data` from flash to RAM.** Initialized global variables live in flash at link time. The reset handler copies them to their RAM addresses at boot time.
2.  **Zero `.bss` in RAM.** Uninitialized globals must start at zero. The reset handler writes zeros to the entire `.bss` region.
3.  **Call `main()`.** Control transfers to the application.

Any global variable you access in `main()` works correctly because of these two memory operations.

## The Linker Script: Who Builds the Map

The linker script defines the memory geometry. It tells the linker where flash and RAM start, how large each section is, and what symbols to export so the reset handler can find the section boundaries.

```
MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 256K
    RAM  (rwx) : ORIGIN = 0x20000000, LENGTH = 64K
}
```

Symbols like `_start_data`, `_end_data`, and `_start_bss` are computed at link time. The reset handler iterates between them. If your linker script is wrong, your global variables will be garbage.

## Multi-Stage Boot: Bootloaders

A first-stage bootloader reserves a small partition at the start of flash (e.g., the first 4 KB). On every power-up, it runs first. Its job:

1.  Verify the integrity of the application image (signature check or CRC).
2.  Update the Vector Table Offset Register (VTOR) to point at the application's IVT.
3.  Set the new Stack Pointer to the application's MSP.
4.  Jump to the application's reset handler at `APP_OFFSET + 4`.

This two-stage design means you can update the application remotely without ever touching the bootloader.

## Fault Handlers: Your Debugger in Production

The Cortex-M3/M4 provides specific registers (e.g. CFSR at `0xE000ED28`) that capture exactly why a fault occurred: invalid instruction fetch, bus error, stack overflow, etc. A good fault handler reads these registers and logs them before resetting. This transforms a silent crash into a traceable event.

---

## Real-World Case Studies: Boot Integrity

### The Vulnerability of a Fragile Boot (Negative Example)
Secure boot mechanisms often rely on simple "Readout Protection" (RDP) bits. On many microcontrollers, these bits are vulnerable to **Voltage Glitching** attacks.

*   **The Error:** Researchers bypassed the readout protection on **STM32F0 and NXP LPC** processors. By introducing a precisely timed dip in the supply voltage during the bootloader's security check, they forced the processor to skip the "Protection Enabled" check.
*   **The Data:** These attacks allow 100% extraction of proprietary firmware from a "secured" device using hardware costing less than $100. Naive bootloaders that trust a single memory bit are fundamentally insecure.
*   **The Lesson:** A secure boot sequence must be rooted in immutable hardware. Never trust a soft check that a physical glitch can skip.

### The Success of Verified Chains (Positive Example)
Modular bootloaders like **MCUboot** use cryptographically signed image headers to ensure the firmware has not been tampered with.

*   **The Resilience:** When implemented with a **Hardware Root of Trust** (like STM32 TrustZone or NXP High Assurance Boot), every stage of the bootloader verifies the signature of the next stage before execution. 
*   **The Data:** Verified boot chains effectively eliminate the "Malicious Update" attack vector. Zero bypasses have been reported for systems where keys are stored in One-Time Programmable (OTP) memory and verified by immutable ROM code.
*   **The Lesson:** Security is a chain of trust. Start with immutable code in ROM and verify every subsequent layer with asymmetric cryptography.

---

## Mental Model: The Chain of Trust

Security is not a single gateway. It is a sequence of handshakes.

*   **Layer 0: Immutable Root.** This is the ROM code baked into the silicon. It cannot be changed. It holds the "Public Key" used to verify the next layer.
*   **Layer 1: Secure Bootloader.** The ROM verifies the bootloader's signature. If valid, the bootloader runs.
*   **Layer 2: Application.** The bootloader verifies the application image. Only a valid, signed image is allowed to execute.
*   **The Broken Link:** If any layer skips verification or a key is compromised, the entire chain collapses. The strength of your firmware security is equal to the strength of its weakest link.

---

## The Modern Take: MCUboot

Writing a bootloader from scratch is a solved problem. **MCUboot** handles:
*   A/B partition swap with automatic rollback.
*   ECDSA signature verification.
*   SHA-256 image integrity checking.

For new projects, use MCUboot. Write a custom bootloader only when a specific constraint (flash size, crypto hardware) requires it.
