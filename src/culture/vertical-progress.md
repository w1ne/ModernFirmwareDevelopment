# Vertical Progress (Strategy)

> "Horizontal progress is easy to imagine because we already know what it looks like. Vertical progress is harder to imagine because it requires doing something nobody else has done." — Peter Thiel

## The Hook: Escaping the Vendor Trap
Most firmware teams are stuck in **Horizontal Progress (1 to N)**. They take a vendor's reference design, copy a vendor's driver, and modify it slightly. If the silicon goes out of stock, their business stops. They are competing on price and chip availability. In Thiel's terms, they are building 100 typewriters when the world needs a word processor.

**Vertical Progress (0 to 1)** in firmware means building a proprietary software ecosystem that makes the underlying hardware irrelevant. It is about creating a "Secret" that your competitors haven't discovered yet.

## The Theory: The 10x Rule of Verticality
In *Zero to One*, Thiel argues that technology is the primary driver of vertical progress. To achieve this in the embedded world, your solution must be at least **10x better** than the status quo in some dimension.

1.  **The Monopoly of Experience:** Apple's success isn't just "design"; it's **Vertical Integration**. By designing the M-series chips, the macOS kernel, and the application layer simultaneously, they achieve performance and battery life that modular, horizontal competitors (Windows/Intel) cannot match.
2.  **Proprietary Strength:** If your firmware can do in 100ms what others do in 1s, or if your system has **0% memory safety bugs** while others have 70%, you have a vertical lead.
3.  **Network Effects (Internal):** When your firmware stack is modular and reusable (like a internal "Stripe for Firmware"), adding a new sensor takes 5 minutes instead of 5 weeks.

---

## Case Study: SpaceX - The Vertical Integration Monopoly
SpaceX did to the aerospace industry what modern firmware engineers must do to the MCU world.

*   **The Status Quo:** Incumbents (Boeing, Lockheed) built rockets by outsourcing components to thousands of vendors. This created a fragile supply chain and massive "Horizontal" bloat.
*   **The Vertical Leap:** SpaceX brought almost everything in-house, especially the **Software**. By writing their own flight control systems and using commodity hardware where possible, they achieved a **10x cost reduction**.
*   **The Lesson:** When you own the software stack, you own the economics. SpaceX doesn't wait for "Vendor HALs"; they write their own, optimized for their specific mission.

## Analogy: The Stripe of Firmware
Stripe didn't "improve" payments; they turned payments into **7 lines of code**. They moved from a "banking" problem to a "developer" problem.

Vertical progress in firmware means moving from a "circuit" problem to a "logic" problem. This involves using tools like **Digital Twins (Simulation)** and **Modern Languages (Rust)** to abstract away the "wiring" complexity. 

By 2027, the ultimate vertical leap is **RISC-V Co-Design**. By moving from proprietary ISAs (Arm/Intel) to open hardware, companies are now designing custom instructions for their specific AI or safety algorithms—achieving efficiency that horizontal, off-the-shelf silicon can never match.

---

## The Implementation: Finding Your Secrets
To achieve vertical progress, ask your team the "Thiel Question": *What important truth about firmware development do very few people agree with you on?*

*   **Elite Secret:** *We don't need the physical hardware to write 90% of the firmware. Simulation is the primary development environment.*
*   **Elite Secret:** *The vendor's SDK is a trap. We will wrap it in a proprietary, safety-critical abstraction layer.*

## References & Further Reading
*   *Zero to One: Notes on Startups, or How to Build the Future* by Peter Thiel.
*   [Apple’s Vertical Integration Strategy](https://www.strategy-business.com)
*   [How SpaceX Software Works](https://www.spacex.com)
*   [Thiel: Vertical vs Horizontal Progress (Video)](https://www.youtube.com)
