---
description: "[2.Generate] Generate atomic AI coding specs and development plans based on PRD."
input:
  - name: prd_path
    description: (Optional) Absolute path to the Product Requirement Document (PRD).
  - name: audit_report_path
    description: (Optional) Absolute path to the audit report (_Audit_Report_Coverage.md).
---

1. **Environment Awareness & Preparation**
    - **Detect PRD**:
        - If `prd_path` is provided, use it.
        - If NOT provided, search for files matching `docs/*PRD*.md`.
    - **Detect Audit Report**:
        - Search for `_Audit_Report_Coverage.md` in `docs/*_specs/`.
        - **Check Status**: Ignore reports containing `[Status: Processed]`.
        - If a non-processed report exists, set `MODE = "OPTIMIZATION"`.
        - Else, set `MODE = "INITIAL_GENERATION"`.
    - **Setup**:
        - Read PRD content.
        - Extract `{filename}` from `prd_path`.
        - Define `target_dir = docs/{filename}_specs`.

2. **Analysis Phase (Veteran Mode)**
    - **Goal**: Create/Update `Deep_Coverage_Analysis.md`.
    - **Context**: Adhere to `prompt/veteran-persona.md`, `prompt/anti-nih.md`, `prompt/nfr-standards.md`, and `prompt/execution-standards.md`.
    - **If MODE == "INITIAL_GENERATION"**:
        - Perform deep analysis: Context Model, Term Verification, Implicit Challenges, Anti-NIH, Functional Decomposition, Hardcore NFRs.
    - **If MODE == "OPTIMIZATION"**:
        - Read `audit_report_path`.
        - **Forensic Review**: Analyze "Critical Blockers", "Hand-Waving Detection", and "Coverage Expansion Guide".
        - **Action Plan**: Create a `Gap_Resolution_Plan` section mapping audit findings to Spec updates.

3. **Spec Generation / Refinement & Version Control**
    - **Goal**: Create or Update `Spec_[ID]_[Name].md` files.
    - **Standard Rules**:
        - **NO Hand-Waving**: Quantitative designs only.
        - **NO NIH Syndrome**: Reuse industry standards (APISIX, Redis, etc.).
        - **Implement Edge Cases**: DLQs, Idempotency, etc.
    - **Revision History (Mandatory)**:
        - Include a `## 修订记录 (Revision History)` section.
        - Initial entry for `v1.0` or append optimization entry for `v[NewVersion]`.
    - **Output Language**: **Simplified Chinese (简体中文)**.

4. **Governance & Traceability (Mandatory Consistency Check)**
    - **Asset Synchronization**: For ANY creation or modification of Spec files, you MUST check and update the following two files in `{target_dir}`:
        1. **`_Spec_Index_Matrix.md`**: 
            - Ensure the new/updated Spec is listed in the Index.
            - Update the Traceability Matrix (PRD mapping) if the scope of the Spec has changed.
        2. **`Development_Plan.md`**:
            - Update the task status, priorities, or dependencies if applicable.
            - Add new tasks if new Specs were created.
    - **Language Constraint**: **Simplified Chinese (简体中文)**.

5. **Completion & Marking**
    - **Mark Audit Report**: If `MODE == "OPTIMIZATION"`, append `[Status: Processed]` to the audit report.
    - **Final Report**: List all created/updated files.
