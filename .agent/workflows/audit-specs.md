---
description: "[3.Audit] Strictly audit Spec coverage and technical robustness against PRD requirements."
input:
  - name: prd_path
    description: Absolute path to the Product Requirement Document (PRD).
  - name: specs_dir
    description: Absolute path to the directory containing Specs documents.
---

1. **Preparation**
    - Read `prd_path` and all `*.md` files in `specs_dir`.
    - **Archiving**: Move existing `_Audit_Report_Coverage.md` to `_backups/` with a timestamp.
    - **Context**: Adhere to `prompt/veteran-persona.md`, `prompt/anti-nih.md`, `prompt/nfr-standards.md`, and `prompt/execution-standards.md`.

2. **Phase 1: Deep PRD & Context Comprehension**
    - Understand Business Intent and Implicit Technical Challenges.
    - Create a mental `Context Model` of Must-Haves vs Nice-to-Haves.

3. **Phase 2: Forensic Analysis & Gap Detection**
    - **Term Verification**: Use `search_web` for unfamiliar technical terms or algorithms.
    - **Veteran Criteria**:
        - ❌ Missing requirements.
        - 🤡 Hand-waving (qualitative only).
        - ⚠️ Logical loopholes or race conditions.
        - 🚫 NIH Syndrome (re-inventing wheels).
        - ✅ Verified, implementable designs.
    - **Edge Cases**: Check for network failures, partial success, retry logic.

4. **Phase 3: Quantitative Audit Scoring**
    - Evaluate 5 dimensions (0-20 each):
        - C1: Completeness (Coverage).
        - C2: Detail & Technicality (Quantitative designs).
        - C3: Feasibility & Risk (Concurrency, consistency).
        - C4: Security & NFR (PII, Observability).
        - C5: Standard & Reuse (Anti-NIH).

5. **Phase 4: Generate Audit Report**
    - Generate `_Audit_Report_Coverage.md` **in Simplified Chinese**.
    - Structure includes: Scoring details, Critical Blockers, Hand-waving warnings, and a Coverage Expansion Guide.

6. **Conclusion**
    - Display report path and backup path.
