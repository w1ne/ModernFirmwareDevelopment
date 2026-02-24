# Certification Mapping (ISO 26262 / IEC 62304)

Modern DevOps practices don't hinder certification; they **enable** it. This annex maps our technical recommendations to the formal requirements of Safety Standards.

## ISO 26262 (Automotive)

| Standard Requirement | Modern DevOps Counterpart | Location in Book |
| :--- | :--- | :--- |
| **Traceability** | Git Commit to Issue Linking | [Introduction](../README.md) |
| **Reproducibility** | Containerized Toolchains (Docker) | [Reproducible Envs](../part-2-foundation/reproducible-environments.md) |
| **Verification & Validation** | Automated Unit Testing / HIL | [Testing](../part-2-foundation/unit-testing.md) / [HIL](../part-4-delivery/hil.md) |
| **Safety Lifecycle** | CI/CD Pipelines as Guardrails | [Continuous Integration](../part-4-delivery/ci.md) |
| **Coding Standards** | Linters (Clang-Tidy) / Rust Compiler | [The Safety Shift](../part-3-transformation/rust.md) |

## IEC 62304 (Medical Devices)

| Standard Requirement | Modern DevOps Counterpart | Location in Book |
| :--- | :--- | :--- |
| **Software Maintenance** | Secure OTA Updates / Gitflow | [Security](../part-4-delivery/security.md) |
| **SOUP Management** | Automated SBOM Generation | [Security](../part-4-delivery/security.md) |
| **Risk Management** | Automated Regression Suites | [CI/CD](../part-4-delivery/ci.md) |
| **Unit Testing** | Off-Target Unit Testing | [Unit Testing](../part-2-foundation/unit-testing.md) |

> [!TIP]
> **Pro Tip:** When auditing for ISO 26262, your CI/CD pipeline logs serve as your "Process Compliance" proof. Automating the generation of these reports saves months of manual work.
