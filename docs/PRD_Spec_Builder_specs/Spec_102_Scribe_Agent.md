# AI Coding Spec: 抄书员智能体 - 草稿生成引擎 (Spec-102)

> **角色**: AI 工程师
> **任务**: 使用 LangGraph 实现抄书员智能体 (Scribe Agent)

## 1. 背景与目标
**抄书员智能体 (Scribe Agent)** 是生成流水线的第一步。它接收 `IntentIR` (原始意图)，利用 LLM 生成包含 Frontmatter、OpenAPI 和 Gherkin 的结构化 **规格草稿 (Spec Draft)**。

## 2. 核心规则
*   **框架**: LangGraph。
*   **模型**: 使用 Provider 抽象层 (支持 OpenAI/Anthropic/DeepSeek)。
*   **提示词策略**: 使用思维链 (Chain-of-Thought, CoT) 确保质量。
*   **输出控制**: 必须输出结构化内容（或带有严格分隔符的 Markdown）。

## 3. 实现步骤

1.  **Agent 状态定义**:
    *   定义 `AgentState` TypedDict:
        *   `intent`: IntentIR
        *   `draft`: Optional[str]
        *   `critique`: Optional[str]
        *   `revision_count`: int
2.  **LLM 服务与上下文管理**:
    *   创建 `LLMService` 封装 LangChain/LiteLLM。
    *   **Summary Service**: 实现 `summarize_long_content(content: str) -> str`。
        *   算法：如果 content > 10,000 tokens，使用 Map-Reduce 策略进行摘要。
    *   实现方法 `generate_draft(intent: IntentIR) -> str`。
3.  **提示工程**:
    *   创建 `prompts/scribe_system.toml`。
    *   将 "可执行规格 Profile" (Executable Spec Profile) 的规则包含在 System Prompt 中。
4.  **LangGraph 节点: Scribe**:
    *   **预处理步骤**: 在 `scribe_node` 中检查 `intent.raw_content` 长度。
    *   如果超长，先分块并利用 `Summary Service` 提取核心逻辑点，再生成草稿。
    *   **解析增强**: 使用 `StreamingOutputParser` 监控 LLM 输出。如果检测到输出被截断（如 JSON 不完整），抛出重试异常或标记 `draft_truncated=True` 触发自愈循环。
    *   调用 LLM 生成草稿。
    *   更新状态中的 `draft` 字段。

## 4. 技术规格

### 4.1 状态 Schema
```python
class SpecGenerationState(TypedDict):
    intent_id: str
    raw_intent: str
    current_draft: str  # 完整的 Markdown 内容
    messages: List[BaseMessage]
    iteration: int
```

### 4.2 提示词策略
*   **角色**: 高级系统架构师。
*   **上下文**: "你正在将一段对话/工单转化为技术规格文档。"
*   **约束**:
    *   必须包含 Frontmatter (spec_id, version)。
    *   必须包含 OpenAPI 3.1 块。
    *   必须包含 Gherkin 块。
    *   必须显式包含 NFRs (非功能需求)。

### 4.3 图结构 (初始)
*   **Nodes**: `scribe`.
*   **Edges**: `START` -> `scribe` -> `END` (MVP V1 阶段，V2 将引入自愈循环)。

## 5. 验证步骤
1.  **Mock 测试**: 创建脚本 `tests/test_scribe.py`，构造一个虚拟的 `IntentIR`。
2.  **运行图**: 执行 Graph。
3.  **检查输出**:
    *   验证输出是字符串且以 `---` 开头。
    *   验证包含 `openapi:` 和 `Feature:` 关键字。
    *   验证无幻觉占位符 (如 "[在此处插入代码]")。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                                           | 修订人          |
| :--------- | :--- | :----------------------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.2 | 根据审计报告优化：增加 StreamingOutputParser 处理 LLM 响应截断场景 | Tech Veteran AI |
| 2025-12-23 | v1.1 | 根据审计报告优化：增加 Summary Service 处理长文本上下文爆炸问题    | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                                           | AI Scribe       |


