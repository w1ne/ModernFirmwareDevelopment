# Modern Firmware Development

> A comprehensive guide to applying modern DevOps and software delivery practices to firmware engineering.

## Vision

This project aims to bridge the gap between traditional embedded development and modern software engineering practices. We apply the **DORA (DevOps Research and Assessment)** metrics to firmware:

*   **Deployment Frequency:** How often can we flash/release a reliable firmware build?
*   **Lead Time for Changes:** How long does it take for a commit to reach a device?
*   **Time to Restore Service:** How quickly can we recover from a bad OTA update?
*   **Change Failure Rate:** How often do updates brick devices or cause regressions?

## Getting Started

This book is built with [mdBook](https://rust-lang.github.io/mdBook/).

### Prerequisites

*   Rust toolchain (to install mdbook)
*   `cargo install mdbook`

### Building the Book

```bash
mdbook serve --open
```

## Repository Structure

*   `src/`: The book content (Markdown source).
*   `examples/`: Reference implementations for CI/CD, Testing, and Build Systems.
*   `tools/`: Helper scripts.
*   `docs/`: Meta-documentation about this project.

## Contributing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to propose changes.
