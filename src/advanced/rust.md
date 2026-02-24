# The Safety Shift (Rust)

> "In C, you shoot yourself in the foot. In C++, you recycle the bullet. In Rust, you can't even pick up the gun until you've proven you won't shoot anyone."

## The Hook: The $10 Million Typo
Memory safety is the "Ghost in the Silicon." In traditional C/C++ firmware, a single off-by-one error doesn't just crash the app. It creates a security vulnerability that can brick an entire fleet of IoT devices.

For 50 years, the industry accepted these bugs as "the cost of doing business." **Rust** is the "Zero to One" technology that makes these bugs mathematically impossible at compile time, turning "Security" from a process into a property of the language.

## The Theory: Ownership and The Borrow Checker
The "Secret" of Rust is its **Ownership model**. It enforces rules that eliminate 70% of high-severity security vulnerabilities (buffer overflows, use-after-free, race conditions):
1.  **Ownership:** Every resource has a single owner.
2.  **Borrowing:** You can lend the resource, but the "Borrow Checker" ensures no two threads can write to it at the same time.
3.  **Compile-Time Safety:** These checks happen at compile time, meaning there is **zero runtime overhead** for this safety.

---

## Case Study: Android OS - The 1000x Reduction
Google made the "Vertical" decision to move to Rust for all new system-level code in Android.

*   **The Result:** Google reported a **1000x lower density of memory safety vulnerabilities** in Rust code compared to legacy C/C++.
*   **The Velocity Win:** Rust code showed a 4x lower rollback rate and required 25% less time in code reviews. Safety didn't just provide security. It provided **speed**.

## Case Study: Volvo Cars - SPA2 Production (Success)
Volvo Cars (and Polestar) integrated Rust into the production firmware of the low-power management node for the EX90 and Polestar 3.

*   **The Implementation:** This node handles the critical "Wake-up" sequence for the entire electric vehicle platform.
*   **The Outcome:** Volvo reported **zero production issues** directly traced to Rust code in vehicles already shipped.
*   **The Lesson:** By escaping the "Vendor SDK" trap and using a modern language, Volvo turned a commodity ECU into a high-security asset.

---

## The Implementation: The Embassy Advantage
To write Rust for microcontrollers, we use the `no_std` attribute. For modern, low-power applications, the **Embassy** framework is the game-changer.

*   **Async/Await:** Embassy allows you to write asynchronous firmware that looks like synchronous code. The compiler automatically handles the state machines.
*   **Ultra-Low Power:** Embassy automatically puts the CPU into sleep mode when no tasks are running, without requiring manual power management code.
*   **Ferrocene (Certification):** For safety-critical systems, Rust is now fortified by **Ferrocene**. The first ISO 26262 and IEC 61508 certified Rust toolchain. This eliminates the "But is it certified?" blocker for Automotive and Medical industries.

```rust
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    let mut led = Output::new(p.PB7, Level::High, Speed::Low);

    loop {
        led.set_high(); // No more RTOS context switches
        Timer::after(Duration::from_millis(500)).await;
        led.set_low();
        Timer::after(Duration::from_millis(500)).await;
    }
}
```

## References & Further Reading
*   [Google Security Blog: 1000x Safer with Rust](https://security.googleblog.com)
*   [Volvo Cars: Rust in the EX90 Firmware Architecture](https://www.corrode.dev)
*   [Embassy Framework: Async Firmware for Microcontrollers](https://embassy.dev)
*   [Ferrocene: The ISO 26262 Certified Rust Toolchain](https://ferrous-systems.com/ferrocene/)
