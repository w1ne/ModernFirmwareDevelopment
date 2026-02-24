# The Test-Driven Asset (Unit Testing)

> "A physician is obligated to provide the best possible care for their patient. An engineer is obligated to provide the best possible code for their device."

## The Hook: Hardware is Your Bottleneck
The traditional firmware cycle is a "Hardware Handshake":
1. Write code.
2. Flash to the board (2 minutes).
3. Push buttons and watch LEDs (5 minutes).
4. Discover it doesn't work.
5. Repeat.

This cycle is 100x slower than modern software development. To move at "Zero to One" speed, you must decouple your logic from your silicon. 

## Dual-Targeting: The "No-Block" Strategy

The primary goal of modern firmware testing is to ensure that development is **never blocked** by the availability of hardware. This is achieved through **Dual-Targeting**: writing code that runs both on your production MCU (the Target) and on your development machine (the Host).

### The Hardware Bottleneck
If you can only run code on the target, you suffer from:
*   **Hardware Scarcity:** Developers wait for limited prototypes, leading to "mounds of unverified work."
*   **Slow Toolchains:** Target compilers and flash loaders are significantly slower than native host tools.
*   **Co-Debugging Hell:** When a bug occurs, is it the new code, the hardware version, or the compiler optimization?

### The Host-Native Advantage
Running logic on the host (Linux, macOS, or Windows) allows you to:
1.  **Iterate in Seconds:** Native compilers (GCC/Clang) are highly optimized for build speed.
2.  **Use Deep Analysis Tools:** Tools like *Valgrind* or *AddressSanitizer* (ASan) provide a level of memory visibility that is impossible on small MCUs.
3.  **Prevent Integration Hell:** If the logic works on the host, you only have to debug the hardware interaction (HAL) when you move to the board.

---

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

### The Success of Test-Driven Development (Positive Example)
In 2008, **Microsoft** and **IBM** conducted a landmark study on the industrial impact of Test-Driven Development (TDD).

*   **The Problem:** Traditional "Code-then-Test" workflows often lead to high defect density and expensive late-stage rework.
*   **The Shift:** Teams across Microsoft and IBM adopted TDD (Red-Green-Refactor) for production systems. 
*   **The Data:** The study showed a **40% to 90% reduction in pre-release defect density** compared to non-TDD projects. While initial development time increased by **15% to 35%**, the cost was offset by the drastic reduction in bug-fixing effort later in the lifecycle.
*   **The Lesson:** TDD is not a speed penalty; it is a quality insurance policy. For firmware, where field failures can trigger expensive recalls, a 2x to 10x reduction in bugs is the difference between a profitable product and a financial disaster.

---

---

## The Implementation: Picking Your Weapons
For modern C++, you have three primary choices:

1.  **Google Test (GTest):** The industry standard for logic-heavy projects.
2.  **Ceedling (Unity/CMock):** Built specifically for Embedded C. It automates the "Mocking" of hardware registers.
3.  **Mull / Dextool:** Modern mutation testing plugins for the LLVM/Clang toolchain.

### The "Mock" Strategy:
Don't use the real driver. Create a `MockFlashMemory` that returns a "Simulated Error" to see how your file system handles a corrupt block. That is a test you can't easily perform on real hardware without a hammer.

---

## Practical: The Dual-Target CMake Blueprint

CMake allows you to manage two entirely different build worlds (Host and Target) in one project.

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(FirmwareProject)

# 1. Component Logic (Shared)
add_library(temp_logic STATIC src/temp_logic.c)

# 2. Target Build (ARM)
if(FIRMWARE_TARGET STREQUAL "stm32")
    add_executable(firmware.bin main.c)
    target_link_libraries(firmware.bin PRIVATE temp_logic hal_stm32)
endif()

# 3. Host Build (Unit Tests)
if(CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
    add_executable(unit_tests tests/test_temp_logic.c)
    target_link_libraries(unit_tests PRIVATE temp_logic mock_hal)
endif()
```

This structure ensures that your business logic is bit-identical on both the x86 host and the ARM target.

---

## Practical: The Mock Register (The Function Pointer Strategy)

If your code touches registers, it is "welded" to the hardware. Use function pointers to break the connection for testing.

```c
// temp_sensor.h
typedef struct {
    uint16_t (*read_reg)(uint8_t addr);
} i2c_ops_t;

// The logic under test
uint16_t get_temperature(i2c_ops_t *ops) {
    return ops->read_reg(0x05); 
}

// In your Test
uint16_t mock_read(uint8_t addr) {
    return 0x00FF; // Controlled test value
}

void test_temp(void) {
    i2c_ops_t mock_ops = { .read_reg = mock_read };
    assert(get_temperature(&mock_ops) == 255);
}
```

By passing a "dictionary" of operations, you can verify your algorithm's response to any hardware state without physical silicon.

---

---

## Testing Strategy: Beyond the Unit
A robust verification plan requires multiple perspectives on the same code.

*   **Black-Box Testing:** Testing against requirements without looking at the internal code. This ensures the system does what the user expects.
*   **White-Box Testing:** Testing based on the internal structure of the code (e.g., ensuring every branch of an `if-else` is hit). This finds technical edge cases that black-box testing misses.
*   **Regressions:** Every bug fix must be accompanied by a new test case that would have caught it. This ensures that old bugs never return.

---

## Issue Tracking: Learning from Failure
Finding a bug is only half the battle. Analyzing **why** it happened prevents future ones.

### The "Bug Farm" Pattern
Defects are rarely distributed evenly. They cluster in "Bug Farms"—complex modules that are responsible for a disproportionate number of failures. 
*   **The Action:** If a module has been "fixed" three times and still fails, do not fix it a fourth time. **Discard it.** Rewrite the module with simpler logic and better tests.
*   **Root Cause Analysis:** For every major defect, ask: *Was this a requirement error, a design flaw, or a coding slip?* Tracking this data reveals where your development process is weakest.

## References & Further Reading
*   [Distal Motion Case Study: Surgical Robotics Verification](https://www.vector.com)
*   *Test-Driven Development for Embedded C* by James Grenning.
*   [Mull: Practical Mutation Testing for C++](https://mull.readthedocs.io)
*   [TESSY: Automated Mutation Testing for Embedded C](https://www.razorcat.com)
