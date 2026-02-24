# The Peripheral Paradox

The "Great Leap" in firmware development is the transition from manually toggling bits to managing **Peripheral Abstraction Layers**. 

In the old model, you spent days with a 1,000-page datasheet, calculating baud rate divisors and shift-register layouts. In the modern model, you treat peripherals as services that fulfill a contract.

## The Big Three: UART, SPI, and I2C

Every modern system relies on these three protocols. Understanding their physical nature is essential, even if you never bit-bang them.

1.  **UART (Universal Asynchronous Receiver-Transmitter):** The "Hello World" of silicon. It is point-to-point, asynchronous, and requires no clock signal. It is the fallback for logging and simple CLI interfaces.
2.  **SPI (Serial Peripheral Interface):** The high-speed backbone. It is a synchronous, full-duplex protocol with a Master/Slave (or Main/Subordinate) architecture. Use SPI for high-bandwidth components like Flash memory, LCDs, and Ethernet controllers.
3.  **I2C (Inter-Integrated Circuit):** The sensor bus. I2C uses only two wires (SDA/SCL) to address dozens of devices on a single bus through unique 7-bit addresses. It handles arbitration and clock stretching, making it ideal for low-speed sensors and EEPROMs.

## Modern Abstraction: PAC vs. HAL

Manual register manipulation is a liability. It creates brittle code that breaks when you switch from an STM32F4 to an STM32H7. Modern development uses layers of indirection.

### Peripheral Access Crate (PAC)
In the Rust ecosystem, a **PAC** is a low-level, zero-cost abstraction generated directly from a System View Description (SVD) file. It provides type-safe access to registers but doesn't manage state.

### Hardware Abstraction Layer (HAL)
A **HAL** provides a higher-level API (e.g., `spi.write()`). This is where the **modern approach** shines. By targeting a HAL trait (like `embedded-hal` in Rust or the Zephyr Driver API), your application code remains hardware-agnostic.

## The Zephyr Way: Devicetree and Kconfig

Zephyr RTOS handles peripherals through **Devicetree (dts)**. This separates the *hardware description* from the *software logic*.

*   **Devicetree:** Defines which pins are assigned to I2C1 and what sensors are attached to that bus.
*   **Driver API:** Your application code simply calls `device_get_binding("I2C_1")`. You don't care about register offsets or clock gating; the RTOS infrastructure handles the plumbing.

## Verification: Beyond Printf

When communication fails, `printf` is useless. Modern verification requires "visibility into the wire."

1.  **Logic Analyzers:** Use tools like Saleae or Sigrok. Seeing the actual waveform allows you to catch clock polarity mismatches or I2C NACKs that software tools might hide.
2.  **Simulation (Renode/QEMU):** Before touching hardware, simulate your peripheral interactions. Renode allows you to model entire multi-node systems and verify your SPI/I2C drivers in a deterministic environment.

## The Peripheral Rule
> "Never write a driver for a device that already has a HAL implementation, unless you are the one writing the HAL."

Modern firmware engineering is about **Integration**, not **Reinhabitation**. Your value lies in the unique application logic, not in re-implementing an I2C transaction engine for the 50th time.
