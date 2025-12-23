# AI Coding Spec: 成本控制与熔断 (Spec-013)

> **角色**: 后端工程师 / AI 架构师
> **任务**: 实现租户级的 Token 预算管理与自愈循环熔断

## 1. 背景与目标
LLM 资源昂贵且受限。为防止恶意刷量或因为 Agent 系统死循环（如自愈逻辑失效）导致的账单爆炸，必须建立严苛的预算管理。

## 2. 核心规则
*   **预算检查**: 每次 LLM 调用前必须检查余额。
*   **熔断**: 单词生成 Session 消耗超过阈值立即终止。
*   **存储**: 使用 Redis 存储实时配额 (`rate_limit:tenant_id:token_budget`)。

## 3. 实现步骤

1.  **Token 计数服务**:
    *   在 `LLMService` 中封装回调，记录每次调用的 `prompt_tokens` 和 `completion_tokens`。
2.  **配额中间件/装饰器**:
    *   在 LangGraph 节点启动前执行 `check_budget(tenant_id)`。
    *   如果余额不足，抛出 `InsufficientBudgetError`，中断流程。
3.  **循环计数熔断**:
    *   在 `Spec-103` 循环逻辑中，除了 `iteration >= 3` 外，增加 `total_tokens > Max_Per_Session` 的判断，防止单次请求成本溢出。

## 4. 技术规格
### 4.1 Redis 键设计
*   `budget:daily:{tenant_id}`: 当日累计消耗。
*   `budget:session:{intent_id}`: 单次会话最高预算。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                             | 修订人          |
| :--------- | :--- | :----------------------------------- | :-------------- |
| 2025-12-23 | v1.0 | 初次生成 (基于审计报告 NFR 建议补全) | Tech Veteran AI |

