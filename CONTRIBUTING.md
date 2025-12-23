# Contributing to Spec Builder Framework

First off, thanks for taking the time to contribute! 🎉

## 🏛️ Framework-First Contribution

Spec Builder is a **Spec Engineering Framework**. Contributions should primarily target the framework's logic and standards, rather than the sample data.

### 1. Workflows (`.agent/workflows/`)
If you want to improve the automation logic:
- Focus on the feedback loop mechanisms.
- Improve error handling or audit dimension scoring logic.
- Ensure workflows remain tech-stack agnostic.

### 2. Prompts (`prompt/`)
If you want to refine the technical standards:
- All prompts in `prompt/` must be abstract and applicable to multiple tech stacks.
- Ensure the "Veteran Persona" maintains a professional, direct, and rigorous tone.
- Add or improve non-functional requirement categories (NFRs).

### 3. Documentation & Samples (`docs/`)
- Content in `docs/` is considered **User Workspace** or **Example Data**.
- Only modify `docs/PRD_Spec_Builder.md` if it is needed to demonstrate new framework capabilities or for regression testing.

## How to Submit a Change

1. Fork the repo and create your branch from `main`.
2. Update the logic in `.agent/workflows/` or standard prompts in `prompt/`.
3. Test your changes by running the workflow against a sample PRD (like `docs/PRD_Spec_Builder.md`).
4. Ensure the output quality is maintained or improved.
5. Create a Pull Request with a clear description of the *logic improvement*.

---
© 2025 SparkForge Team.
