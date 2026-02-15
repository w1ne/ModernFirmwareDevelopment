# Decision: Documentation Tool Selection

**Date:** 2026-02-15
**Status:** Proposed

## Context
We need to select a tool to author and publish a comprehensive guide/book on "Modern Firmware Development". The tool must support:
- A "book-like" structure (chapters, sections).
- Easy writing experience (Markdown preferred).
- Modern aesthetics.
- Maintainability.

## Context
We need to select a tool to author and publish a comprehensive guide/book on "Modern Firmware Development". 

## Research
Detailed comparison of options (mdBook, Sphinx, Docusaurus, MkDocs) can be found in `docs/research/documentation_tools.md`.

## Decision
**Proposed Tool: mdBook**

### Rationale
- **Fit for Purpose:** It is explicitly designed to build *books*.
- **Modern Firmware Relevance:** It is the standard tool for documentation in the modern embedded/Rust community.
- **Simplicity:** It allows focusing on content (Markdown) with minimal configuration.


## Consequences
- We need to install `mdbook` (available via cargo or binary download).
- Content will be written in Markdown.
- Structure is defined in `SUMMARY.md`.
