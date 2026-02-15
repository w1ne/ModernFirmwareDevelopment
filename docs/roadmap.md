# Project Roadmap: Modern Firmware Development

This roadmap defines the detailed execution path for the book.

## Phase 1: Foundation (The "Inner Loop")
*Goal: Establish the developer environment and feedback loops.*

### 1. Culture & Mindset
- [ ] Write `src/culture/mindset.md`: Shift from "Hero" to "System".
- [ ] Write `src/culture/dora-metrics.md`: Defining the 4 metrics for FW.

### 2. Reproducible Environments (Docker)
- [ ] Write `src/foundation/reproducible-environments.md`: The "Why" and "How" of containers.
- [ ] Create `examples/docker/Dockerfile`: Multi-stage build for ARM GCC + Rust.
- [ ] Create `examples/docker/devcontainer.json`: VS Code integration.

### 3. Unit Testing on Host
- [ ] Write `src/inner-loop/unit-testing.md`: Interaction-based testing vs state-based.
- [ ] Create `examples/firmware/unit-test-cpp`: C++ GoogleTest example.
- [ ] Create `examples/firmware/unit-test-rust`: Rust `cargo test` example.

## Phase 2: Modern Tech (The "Zero to One" Shift)
*Goal: Introduce game-changing technologies.*

### 4. Modern Toolchains
- [ ] Write `src/inner-loop/toolchains.md`: CMake vs Cargo vs Zephyr (West).
- [ ] Write `src/advanced/rust.md`: Safety, borrow checker, and embedded HAL.
- [ ] Create `examples/firmware/rust-blinky`: Basic `no_std` Rust app for STM32 (or generic).

### 5. Simulation (Renode & LabWired)
- [ ] Write `src/inner-loop/simulation.md`: Renode introduction.
- [ ] Section: "The Future: LabWired" (Visual topology, collaboration).
- [ ] Create `examples/tools/renode/`: Basic Renode script (`.resc`) and robot test.

## Phase 3: Delivery (The "Outer Loop")
*Goal: Automate delivery and ensure security.*

### 6. CI/CD Pipelines
- [ ] Write `src/outer-loop/ci.md`: Pipeline architecture.
- [ ] Create `examples/ci-cd/github-actions.yml`: Pipeline using the Docker image from Phase 1.

### 7. Security & Compliance (CRA)
- [ ] Write `src/advanced/security.md`: Secure Boot, SBOMs.
- [ ] Create `examples/tools/sbom-generator.sh`: Script to generate SBOM from Zephyr/Rust build.

## Phase 4: Verification & Publish
- [ ] Review all code examples.
- [ ] Proofread all chapters.
- [ ] Publish to GitHub Pages.
