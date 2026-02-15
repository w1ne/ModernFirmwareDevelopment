# Contributing to Modern Firmware Development

Thank you for your interest in contributing! We want to create the definitive guide for modern firmware practices.

## How to Contribute

1.  **Open an Issue:** If you want to add a new chapter or significantly change existing content, please open an issue first to discuss the scope.
2.  **Fork & Branch:** Create a branch for your changes.
3.  **writing Style:**
    *   Be concise.
    *   Use "we" to refer to the engineering community/authors.
    *   Focus on *application* of concepts, not just theory.
    *   Provide code examples where possible.
4.  **Submit a PR:** meaningful commit messages are appreciated.

## Content Guidelines

We follow the "Accelerate" model. If you are adding a new practice (e.g., "Fuzz Testing"), try to relate it back to how it improves one of the four DORA metrics (Deployment Frequency, Lead Time, Mean Time to Recovery, Change Failure Rate).

## structure

*   Chapters are located in `src/`.
*   Update `src/SUMMARY.md` if you add new files.
*   Place lengthy code examples in `examples/` and reference them in the text.
