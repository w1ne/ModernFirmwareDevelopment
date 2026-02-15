# The Test-Driven Asset (Unit Testing)

> "A physician is obligated to provide the best possible care for their patient. An engineer is obligated to provide the best possible code for their device."

## The Hook: Hardware is Your Bottleneck
The traditional firmware cycle is a "Hardware Handshake":
1. Write code.
2. Flash to the board (2 minutes).
3. Push buttons and watch LEDs (5 minutes).
4. Discover it doesn't work.
5. Repeat.

This cycle is 100x slower than modern software development. To move at "Zero to One" speed, you must decouple your logic from your silicon. You must test **Off-Target**.

## The Theory: Coverage vs. Mutation Score
Elite teams know that **Code Coverage** (did the line run?) is a vanity metric. You can have 100% coverage with zero assertions. 

The "Elite" metric is the **Mutation Score**.
*   **Mutation Testing:** A tool (like *Mull* or *TESSY*) automatically injects bugs ("mutants") into your code—changing a `+` to a `-` or a `>` to a `>=`.
*   **The Goal:** If your tests still pass after the code is mangled, your tests are weak. 
*   **The Insight:** Mutation testing identifies the "silent failures" that coverage misses.

---

## Case Study: Distal Motion - Surgical Robots
Distal Motion, the manufacturer of the *Dexter* surgical robot, operates in a world where software failure isn't an option (Safety Class C, IEC 62304).

*   **The Problem:** Manually verifying safety requirements on physical robot prototypes was slow and expensive.
*   **The Solution:** They implemented a rigorous off-target unit testing pipeline. They treated software verification as a continuous process powered by simulation.
*   **The Result:** A **43% increase in code coverage** and a significant reduction in post-integration defects. More importantly, they achieved medical certification faster by providing automated proof of correctness.
*   **The ROI:** While unit testing adds roughly 30% to initial development time, it reduces the cost of "late-stage debugging"—which is typically **10x to 100x more expensive** in embedded systems.

---

## The Implementation: Picking Your Weapons
For modern C++, you have three primary choices:

1.  **Google Test (GTest):** The industry standard for logic-heavy projects.
2.  **Ceedling (Unity/CMock):** Built specifically for Embedded C. It automates the "Mocking" of hardware registers.
3.  **Mull / Dextool:** Modern mutation testing plugins for the LLVM/Clang toolchain.

### The "Mock" Strategy:
Don't use the real driver. Create a `MockFlashMemory` that returns a "Simulated Error" to see how your file system handles a corrupt block. That is a test you can't easily perform on real hardware without a hammer.

## References & Further Reading
*   [Distal Motion Case Study: Surgical Robotics Verification](https://www.vector.com)
*   *Test-Driven Development for Embedded C* by James Grenning.
*   [Mull: Practical Mutation Testing for C++](https://mull.readthedocs.io)
*   [TESSY: Automated Mutation Testing for Embedded C](https://www.razorcat.com)
