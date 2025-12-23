# AI Coding Spec: 可观测性 (Spec-012)

> **角色**: DevOps / 后端工程师
> **任务**: 实现指标监控与全链路追踪

## 1. 背景与目标
为了确保 Spec Builder 的稳定性并量化 LLM 消耗，我们需要建立全方位的可观测性方案。重点在于监控 Token 消耗、Agent 循环延迟以及异步队列深度。

## 2. 核心规则
*   **指标 (Metrics)**: 使用 `prometheus-client`。
*   **追踪 (Tracing)**: 使用 OpenTelemetry (OTel)。
*   **日志**: 结构化 JSON 日志 (StructLog)。

## 3. 实现步骤

1.  **Prometheus 埋点**:
    *   在网关/应用层统计：
        *   `http_request_duration_seconds`: API 响应延迟。
        *   `llm_token_usage_total`: 累计 Token 消耗（按模型、租户拆分）。
        *   `scribe_agent_loop_count`: 自愈循环次数分布。
2.  **OpenTelemetry 集成**:
    *   在 Ingestion API 入口创建 `trace_id`。
    *   将 `trace_id` 注入 Redis Streams 消息头。
    *   在 Scribe Worker 中继承 context，确保从 Webhook 到规格生成的全链路可追溯。
3.  **仪表盘**:
    *   提供基础的 Grafana 配置 JSON 文件。

## 4. 技术规格
### 4.1 关键指标字典
| 指标名                          | 类型      | 描述                       |
| :------------------------------ | :-------- | :------------------------- |
| `agent_self_healing_iterations` | Histogram | 每个规格生成的循环次数     |
| `intent_queue_backlog`          | Gauge     | Redis Streams 待处理消息数 |

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                             | 修订人          |
| :--------- | :--- | :----------------------------------- | :-------------- |
| 2025-12-23 | v1.0 | 初次生成 (基于审计报告 NFR 建议补全) | Tech Veteran AI |

