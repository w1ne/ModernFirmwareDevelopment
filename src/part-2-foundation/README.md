# Part 2: The Foundation (Inner Loop)

> "It works on my machine" is an unprovable statement.

The "Inner Loop" is the cycle of **Code -> Build -> Test** that a developer repeats hundreds of times a day.

In traditional firmware, this loop is slow (flash time) and fragile (dependency hell). In this section, we build the technical foundation to make this loop **fast**, **reproducible**, and **hardware-independent**.

## Chapters
- [The Hardware Handshake](hardware-handshake.md)
- [Requirements Engineering](requirements.md)
- [The Boot Sequence](boot-sequence.md)
- [The Abstraction Shift](abstraction-shift.md)
- [Flow of Activity](flow-of-activity.md)
- [The Peripheral Paradox](peripherals.md)
- [The Memory Safety Shift](memory-safety.md)
- [Debugging as Thinking Patterns](debugging-patterns.md)
- [Reproducible Environments (Docker)](reproducible-environments.md)
- [Dependency Management](dependency-management.md)
- [Unit Testing on Host](unit-testing.md)
- [Static Analysis](static-analysis.md)
