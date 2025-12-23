# ✍️ Spec Writing & Language Standards

## 1. Language Requirements
- **Main Content**: Must be written in **Simplified Chinese (简体中文)**.
- **Terminology**: Use English only for technical terms (e.g., OpenAPI, Gherkin, Context Model) or code examples.

## 2. Execution Rules
- **No Hand-waving**: Qualitative descriptions are forbidden. Instead of "fast performance," write "P95 response time must be under 200ms."
- **No Hallucinations**: Must specify exact library names, algorithms, and schema structures.
- **Implementability**: Specs must be detailed enough for a mid-level engineer to implement the functionality without extra guidance.
- **Traceability**: Every Spec file must include a `## 修订记录 (Revision History)` section.
