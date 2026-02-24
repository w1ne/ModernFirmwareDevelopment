# Requirements Engineering

> "Requirements that aren't written down aren't requirements. They're wishes."

Most embedded projects have requirements. Few have them written well. The difference between a written requirement and a good written requirement is measurability.

## Why Written Requirements Matter

Industry research into firmware design failures shows that the single most common source of project failure is requirements that exist only in someone's head. When they leave the team, the requirement leaves with them.

A written requirement does three things:
1. Forces a concrete definition of "done."
2. Gives the team something to push back with against scope creep ("that's not in the requirements").
3. Creates a testable checkpoint.

## Functional vs. Non-Functional Requirements

Every system has both types. Leaving out non-functional requirements is the norm. It is also a disaster waiting to happen.

**Functional:** What the system shall do. Easy to write. "When the button is pressed, the LED shall turn on."

**Non-Functional:** How the system shall behave. Hard to write, impossible to ignore.

| Category | Example Requirement |
|---|---|
| Performance | System shall respond to a button press within 100 ms |
| Resource usage | Flash shall not exceed 75% capacity at initial release |
| Reliability | MTBSF (software) shall exceed 1,000 operating hours |
| Energy | System shall operate on 2xAA cells for at least 18 months |
| Boot time | System shall be ready for use within 3 seconds of power-on |
| Security | System shall authenticate firmware updates using ECDSA-P256 |
| Safety | System shall not command motor movement when door sensor is open |

## The Good Requirement Rule

A requirement must have a **pass/fail criterion** attached. If you cannot write a test for it, it is not a requirement. It is a goal.

Bad: "The system shall be fast."
Good: "The system shall process sensor data with a maximum latency of 50 ms at the 99th percentile under rated load."

Bad: "The software shall not crash."
Good: "Mean time between software resets shall exceed 5,000 operating hours. 95% of resets shall be followed by automatic recovery within 30 seconds."

Avoid requiring perfection. "Shall never crash" is not testable. "Mean time between software resets shall exceed..." is.

## Requirement Traceability

Every requirement must trace to at least one test. Every test must trace back to a requirement.

Use a simple matrix: requirements as columns, tests as rows. Fill in a mark where a test covers a requirement. Empty columns reveal untested requirements. Empty rows reveal tests with no justification.

```
        R1   R2   R3   R4   R5
T1       X         X
T2            X         X
T3       X    X
T4                       X   X
T5                  X        X
```

In this matrix, R4 has a gap: T4 only partially covers it. R5 has no coverage at all. Both must be fixed before ship.

## Requirement Churn: The Silent Killer

Requirements that change constantly after development has started do not just cost time. They invalidate tests, break designs, and erode team confidence.

Track requirement changes explicitly. Every change needs:
- A justification
- An impact assessment (which tests, which files?)
- Approval from the person who owns the requirement

If requirements churn is high, the project is not yet ready to code. Stop. Stabilize requirements first.

---

## Real-World Case Studies: The Data of Discipline

### The Cost of Vague Requirements (Negative Example)
In 1996, the **Ariane 5 Flight 501** rocket disintegrated 40 seconds after launch. The failure originated from unverified requirements during software reuse.

*   **The Error:** An Ada software module from the Ariane 4 was reused without updating its environmental requirements. The Ariane 5 had a significantly higher horizontal velocity. This caused a 64-bit floating-point value to overflow a 16-bit signed integer. 
*   **The Code:** 
```ada
-- The vulnerable conversion (Conceptual Ada)
variable_16bit := integer(variable_64bit_float); 
-- Overflow occurred when variable_64bit_float > 32,767
```
*   **The Data:** The unhandled exception triggered a self-destruct. The total loss exceeded **$370 million**. 
*   **The Lesson:** Requirements must include strict operating ranges for every input. Reused code is a liability if the new environment requirements are not verified.

### The Success of Priority Requirements (Positive Example)
During the **Apollo 11** lunar landing in 1969, the guidance computer encountered multiple **1201 and 1202 alarms** (Executive Overflow). The mission succeeded because the requirements prioritized life-critical tasks over informational ones.

*   **The Resilience:** The software was built on a "Fail-Gracefully" requirement. When the processor became overloaded by spurious radar data, it discarded lower-priority tasks (like the landing site display) to maintain ascent-engine control.
*   **The Data:** The system performed a soft restart and resumed only the most critical jobs. The crew landed safely within 30 seconds of fuel exhaustion.
*   **The Lesson:** Good requirements define a hierarchy of importance. When resources fail, the system must know exactly which requirements to sacrifice to preserve the primary mission.

---

## The Modern Approach

**Acceptance tests as requirements.** Write acceptance tests in Gherkin (Given/When/Then) format. These are executable requirements. The CI pipeline runs them. Requirements that cannot be expressed as acceptance tests are too vague to build.

**Architecture Decision Records (ADRs).** Non-functional requirements that drive architectural choices (e.g., "must use TLS 1.3") should be captured as ADRs alongside the requirement. This links the why to the what.

**Living requirements.** Use a tool like GitHub Issues with requirement labels. Link issues to commits. The CI pipeline enforces that no code ships without a closed requirement backing it.

---

## Practical: Requirements as Code (Gherkin)

A requirement that you can execute is a requirement that stays true. Gherkin (Given/When/Then) format turns a wish into a verifiable test case.

```gherkin
Feature: Motor Safety Interlock
  Scenario: Prevent motor movement when safety door is open
    Given the safety door is "OPEN"
    And the motor is "IDLE"
    When the user commands "START"
    Then the motor shall remain "IDLE"
    And the system shall log a "SAFETY_INTERLOCK_FAULT"
```

### The Power of Executable Specs
1.  **Automation:** This file is read by a test runner (e.g., Cucumber or C-based equivalents). It generates the test harness automatically.
2.  **Shared Language:** Mechanical engineers, software engineers, and safety auditors can all read and approve this file.
3.  **Traceability:** The ID of this requirement is linked to the test result in every CI run.

---

> [!IMPORTANT]
> If your requirements document is 100% functional requirements, you are missing the hardest part. Add a non-functional requirements section before you write a single line of code.
