# Starter Blueprints: From Zero to One

> "The hardest part of any journey is the first step. These blueprints ensure your first step is on solid ground."

To move from theory to practice, you need a project structure that is **Reproducible**, **Secure**, and **Testable** from day one. Below are two "Elite" blueprints for the most common modern firmware stacks.

---

## 1. The Zephyr "Zero-to-One" Blueprint
This structure is designed for scalability. It keeps your application code separate from the RTOS kernel, making it easy to up-copy or port to new hardware.

### Project Structure
```text
my-awesome-project/
├── .github/workflows/ci.yml   # Quality Gates
├── app/                       # Your safe, portable application code
│   ├── CMakeLists.txt
│   ├── prj.conf               # Kconfig features
│   └── src/main.c
├── boards/                    # Custom board definitions (DTS)
├── Dockerfile                 # Reproducible build environment
├── west.yml                   # Dependency manifest
└── README.md
```

### The "Elite" `west.yml`
```yaml
manifest:
  remotes:
    - name: zephyrproject-rtos
      url-base: https://github.com/zephyrproject-rtos
  projects:
    - name: zephyr
      remote: zephyrproject-rtos
      revision: v3.5.0  # Pin your version!
      import: true
  self:
    path: my-awesome-project
```

### The CI Gate (`.github/workflows/ci.yml`)
```yaml
name: Build and Test
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    container: zephyrprojectrtos/zephyr-build:latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: West Init
        run: west init -l app && west update
      - name: Build
        run: west build -b native_sim app
```

---

## 2. The Rust "Safety-First" Blueprint
For teams prioritizing memory safety and modern concurrency using the **Embassy** framework.

### Project Structure
```text
rust-firmware/
├── .cargo/config.toml         # Target & Runner config
├── .github/workflows/ci.yml
├── src/
│   ├── main.rs                # Async entry point
│   └── lib.rs                 # Business logic (unit-testable)
├── Cargo.toml                 # Dependency & Profile config
├── rust-toolchain.toml        # Pin the compiler version
└── README.md
```

### The "Elite" `Cargo.toml`
```toml
[package]
name = "rust-firmware"
version = "0.1.0"
edition = "2021"

[dependencies]
embassy-executor = { version = "0.5", features = ["task-arena-size-32768"] }
embassy-time = { version = "0.3", features = ["defmt"] }
embassy-stm32 = { version = "0.1", features = ["defmt", "stm32f411ce", "time-driver-tim2"] }
defmt = "0.3"
panic-probe = { version = "0.3", features = ["print-defmt"] }

[profile.release]
lto = true
opt-level = 'z' # Optimize for size
```

### The Async Entry Point (`src/main.rs`)
```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    defmt::info!("Zero to One: Rust Edition Started!");

    loop {
        Timer::after(Duration::from_millis(1000)).await;
        defmt::info!("System Heartbeat: OK");
    }
}
```

---

## Which one should I choose?
*   **Choose Zephyr** if you need massive driver support for BLE, WiFi, or complex networking stacks and need to support multiple distinct hardware chips (e.g., STM32 and nRF52).
*   **Choose Rust** if security is your #1 priority or if you want to eliminate 70% of common firmware bugs at compile-time while leveraging a modern async programming model.
