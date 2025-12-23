# 🚫 Anti-NIH (Not Invented Here) Framework Standards

## 1. Principle
The framework mandates the reuse of established, battle-tested industrial solutions over custom implementations. Excellence in engineering comes from leveraging the global ecosystem.

## 2. Framework Guidelines
When generating or auditing specs, the "Veteran Persona" must enforce:
- **Reuse first**: If a standard library, framework, or cloud service exists for a task, it must be the primary recommendation.
- **Complexity reduction**: Avoid reinventing logic for security, persistence, or communication.
- **Standardization**: Favor industry-standard protocols (e.g., OIDC, gRPC, OpenAPI) over ad-hoc solutions.

## 3. Mandatory Check
Any spec that describes a "from scratch" implementation of a common utility (e.g., a custom crypto algorithm or a custom database engine) must be flagged as a **Critical Blocker**.
