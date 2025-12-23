---
description: "[1.Audit] Conduct deep evaluation of PRD quality, identifying logical loopholes and ambiguous requirements."
---

1. **Target Document Acquisition**
   - **Priority 1**: Check if `prd_path` is provided. If not, check if the user explicitly provided a document path or content in the request (e.g., via `@[filename]` or direct text). Use this if available.
   - **Priority 2**: If no specific document is provided, use the currently active/open file in the editor.
   - **Priority 3**: If neither is detected, ask the user to specify the target PRD document.
   - Read the full content of the selected target document.

2. **Run Execution**
   - **Context**: Adhere to the following prompt components:
     - `prompt/veteran-persona.md` (Persona)
     - `prompt/execution-standards.md` (Language & Quality)
   - **Task**: Perform a comprehensive quantitative evaluation of the input PRD document.

   **Goals**:
   1. Deeply analyze the content of the PRD document.
   2. Score based on 5 core dimensions (0-20 scale each, totaling 100 points).
   3. Identify logical loopholes, ambiguous descriptions, and missing modules.
   4. Output a structured quantitative evaluation report **in Simplified Chinese**.

   **Evaluation Dimensions (Weightage-based)**:
   1. **Completeness - 30%**: Background, objectives, business processes, feature details, NFRs, and change logs.
   2. **Clarity & Precision - 25%**: Absence of vague terms; boolean closed-loop logic.
   3. **Feasibility & Compliance - 15%**: System boundaries, security, audit logs, etc.
   4. **Testability - 20%**: Quantifiability of requirements; expected results.
   5. **Interaction Consistency - 10%**: Logic consistency and uniform feedback.

   **Constraints**:
   - The evaluation must be **objective and strict**.
   - If a core module is missing, score strictly **0** for that item.
   - Use reverse thinking to find missing "What if..." scenarios (Edge Cases).
   - **IMPORTANT: The final output MUST be in Simplified Chinese.**

3. **Report Generation**
   - Output the evaluation report following the Markdown format (keep headers in Chinese).

   ```markdown
   ## [PRD Name] 质量评估报告
   ### 1. 综合得分: [Score] / 100

   ### 2. 维度评分表
   | 维度               | 分数 | 权重 | 加权分 | 评价结论 |
   | :----------------- | :--- | :--- | :----- | :------- |
   | 完整性             |      | 30%  |        |          |
   | 清晰度与准确性     |      | 25%  |        |          |
   | 技术可行性与合规性 |      | 15%  |        |          |
   | 可测试性           |      | 20%  |        |          |
   | 交互一致性         |      | 10%  |        |          |

   ### 3. 详细审计详情
   - **逻辑漏洞**: (List format, indicating document location)
   - **模糊表述**: (List original text and suggested modification)
   - **缺失场景**: (List undefined exception branches)

   ### 4. 改进行动建议 (Action Items)
   - [ ] Priority-High: ...
   - [ ] Priority-Medium: ...
   ```
