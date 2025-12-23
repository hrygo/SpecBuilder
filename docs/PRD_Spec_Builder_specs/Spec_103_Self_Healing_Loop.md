# AI Coding Spec: 智能自愈引擎 (Spec-103)

> **角色**: AI 工程师
> **任务**: 实现 Validator, Reflector, Corrector 智能体及循环逻辑

## 1. 背景与目标
抄书员智能体 (Scribe Agent) 的输出很难一次完美。**智能自愈引擎** 实现了一个特定的反馈循环 (Agentic Pattern)，在将草稿提交给人类之前，迭代地提升草稿质量。

## 2. 核心规则
*   **预算意识**: 最大循环次数 = 3。如果质量没有提升则停止。
*   **确定性校验优先**: 优先于 LLM 的主观判断 (例如：先使用 YAML linter 检查 OpenAPI 有效性)。
*   **结构化批评**: `Validator` 必须输出 JSON 格式的问题列表，而非自由文本。

## 3. 实现步骤

1.  **Validator (校验者) 节点**:
    *   **工具**: `AmbiguityLinter` (基于正则) - 关联 Spec-104。
    *   **工具**: `OpenAPIValidator` (Python `openapi-spec-validator`)。
    *   **LLM 检查**: 用于逻辑一致性 (例如："Gherkin 步骤是否与 API 参数匹配？")。
    *   **输出**: `ValidationReport` (passed: bool, errors: List[str])。
2.  **Reflector (反思者) 节点** (可选但推荐):
    *   分析 *为什么* 会出错。制定修正计划。
3.  **Corrector (修正者) 节点**:
    *   输入 `current_draft` + `ValidationReport`。
    *   生成 `new_draft`。
    *   增加 `iteration` 计数器。
4.  **图逻辑与持久化 (Graph Logic & Persistence)**:
    *   在 `Validator` 处添加条件边 (Conditional Edge):
        *   如果 `passed` 为真 或 `iteration >= 3` -> END。
        *   否则 -> `Corrector`。
    *   边 `Corrector` -> `Validator`。
    *   **持久化层**: 使用 `langgraph-checkpoint-postgres`。
    *   实现 `PostgresSaver` 并将其作为 `checkpointer` 传入 `compile()` 过程，确保会话状态持久化在 `checkpoints` 表中。

## 4. 技术规格

### 4.1 状态更新
向 `AgentState` 添加 `validation_errors: List[str]`。

### 4.2 Validator 逻辑
```python
def validator_node(state):
    errors = []
    # 1. 结构检查
    if "openapi:" not in state['draft']: errors.append("Missing OpenAPI block")
    # 2. Linter 检查 (调用 Spec-104 逻辑)
    # 3. LLM 逻辑检查
    return {"validation_errors": errors, "iteration": state["iteration"]}
```

### 4.3 路由逻辑
```python
def route_after_validation(state):
    if not state["validation_errors"]:
        return END
    if state["iteration"] >= 3:
        return END  # 放弃修正，返回尽力而为的结果
    return "corrector"
```

## 5. 验证步骤
1.  **测试内部循环**:
    *   向 State 注入一个 *损坏* 的草稿 (例如：无效的 YAML)。
    *   从 `validator` 节点开始运行 Graph。
    *   验证 `corrector` 被调用。
    *   验证 `validator` 再次被调用。
    *   验证循环最终终止。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                            | 修订人          |
| :--------- | :--- | :-------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.1 | 根据审计报告优化：集成 PostgresSaver 进行状态持久化 | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                            | AI Scribe       |

