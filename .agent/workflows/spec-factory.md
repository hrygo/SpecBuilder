---
description: "[Factory] 🏭 Automated Pipeline: PRD Audit -> Spec Gen -> Coverage Audit -> Iterative Optimization."
---

# 🏭 Spec Factory: Automated Production Loop

This workflow implements an end-to-end pipeline to transform a PRD into high-quality technical specs, ensuring a confidence score of >90/100 through iterative self-healing.

## 0. Initialization
- **Input**: `prd_path` (Required).
- **Configuration**: `max_iterations = 5`, `target_score = 90 / 100`.
- **Environment**: 
    - Extract `{prd_filename}` from `prd_path`.
    - Define `target_dir = docs/{prd_filename}_specs`.

## 1. Phase 1: PRD Quality Audit (Source Governance)
- **Check for Skip**: 
    - If `{target_dir}/PRD_Quality_Report.md` exists and is consistent with the current PRD, **SKIP** this phase.
- **Trigger**: Run `audit-prd.md`.
- **Logic**: Inspect PRD for logical loopholes and ambiguities.
- **Decision**: 
    - If score < 85, **ABORT** and notify the user for PRD improvements.
    - Else, proceed to Phase 2.

## 2. Phase 2: Specs Initial Generation (Asset Crystallization)
- **Check for Skip**: 
    - If `{target_dir}/_Spec_Index_Matrix.md` exists, **SKIP** this phase. 
- **Trigger**: Run `gen-specs.md` in `INITIAL_GENERATION` mode.
- **Output**: Generates initial Spec set, Deep Analysis, and Development Plan.

## 3. Phase 3: The Refinement Loop (Self-Healing)
- **Variable**: `loop_count = 1`.

### 3.1 Independent Audit
- **Trigger**: Run `audit-specs.md`.
- **Input**: `target_dir` as `specs_dir`.
- **Read Score**: Extract the `Confidence Score` from the generated `_Audit_Report_Coverage.md`.

### 3.2 Exit Condition
- **Condition A (Pass)**: `Confidence Score >= target_score`.
- **Condition B (Exhausted)**: `loop_count >= max_iterations`.
- **Action**: If A or B is met, exit the loop and move to Phase 4.

### 3.3 Iterative Optimization
- **Trigger**: Run `gen-specs.md` in `OPTIMIZATION` mode.
- **Read Input**: Adhere to the latest `_Audit_Report_Coverage.md` and address all gaps.
- **Update**: Increment `loop_count` and jump back to **3.1**.

## 4. Phase 4: Final Delivery & Closing Report
- **Summary**:
    - Display final audit score.
    - List the number of iterations performed.
    - Provide the root path of the generated assets.
- **Cleanup**: Mark the final audit report as `[Status: Processed]`.

## ⚠️ Constraints
- Strictly follow standards in `./prompt/`.
- All updates MUST be recorded in the `## Revision History` of each Spec.
- **Cross-Check**: For any Spec creation or update, you MUST ensure `_Spec_Index_Matrix.md` and `Development_Plan.md` are synchronized.
- **Output Language**: **Simplified Chinese (简体中文)**.

// turbo-all
