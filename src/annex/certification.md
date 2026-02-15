# Certification Mapping (ISO 26262 / IEC 62304)

Modern DevOps practices don't hinder certification; they **enable** it. This annex maps our technical recommendations to the formal requirements of Safety Standards.

## ISO 26262 (Automotive)

| Standard Requirement | Modern DevOps Counterpart | Location in Book |
| :--- | :--- | :--- |
| **Traceability** | Git Commit to Issue Linking | [Introduction](../README.md) |
| **Reproducibility** | Containerized Toolchains (Docker) | [Reproducible Envs](../foundation/reproducible-environments.md) |
| **Verification & Validation** | Automated Unit Testing / HIL | [Testing](../inner-loop/unit-testing.md) / [HIL](../outer-loop/hil.md) |
| **Safety Lifecycle** | CI/CD Pipelines as Guardrails | [Continuous Integration](../outer-loop/ci.md) |
| **Coding Standards** | Linters (Clang-Tidy) / Rust Compiler | [The Safety Shift](../advanced/rust.md) |

## IEC 62304 (Medical Devices)

| Standard Requirement | Modern DevOps Counterpart | Location in Book |
| :--- | :--- | :--- |
| **Software Maintenance** | Secure OTA Updates / Gitflow | [Security](../advanced/security.md) |
| **SOUP Management** | Automated SBOM Generation | [Security](../advanced/security.md) |
| **Risk Management** | Automated Regression Suites | [CI/CD](../outer-loop/ci.md) |
| **Unit Testing** | Off-Target Unit Testing | [Unit Testing](../inner-loop/unit-testing.md) |

> [!TIP]
> **Pro Tip:** When auditing for ISO 26262, your CI/CD pipeline logs serve as your "Process Compliance" proof. Automating the generation of these reports saves months of manual work.
