# AI Coding Spec: 意图采集引擎 (Spec-101)

> **角色**: 后端工程师
> **任务**: 实现意图采集 API 与数据模型

## 1. 背景与目标
"意图采集" 模块是所有数据（聊天记录、工单、会议纪要）的入口。它**必须**捕获原始数据，执行初步清洗（PII 脱敏），将其标准化为 "Intent IR" (意图中间表示)，并异步持久化以供处理。

## 2. 核心规则
*   **异步优先**: 采集 API 必须极快（Fire-and-Forget）。繁重的处理逻辑通过 Redis Streams 卸载。
*   **数据完整性**: 绝不能丢失任何 payload。
*   **安全性**: 必须在入口处进行最小化的 PII 清洗。
*   **模型校验**: 对输入数据进行严格的 Pydantic 校验。

## 3. 实现步骤

1.  **数据模型 (领域层)**:
    *   定义 `SourceMetadata` (Discriminated Union，区分 IDE/Jira/Meeting)。
    *   定义 `IntentIR` (中间表示)，参考 PRD §5.4。
    *   **澄清状态池 (PendingClarification Pool)**: 针对需要人类确认的实体（US-005），在 Redis 中维护一个 `pending_clarifications:{session_id}` 的 Hash 结构，存储待确认项及其快照。
    *   **状态机逻辑 (US-005/006)**: 意图状态流转：`INGESTED` -> `PENDING_CLARIFICATION` (若有低置信度实体) -> `ACCEPTED/REJECTED` (用户操作后) -> `PROCESSING`。
2.  **安全性与脱敏 (网关与服务层)**:
    *   **网关清洗**: 优先在 APISIX 配置 `data-mask` 插件进行脱敏。
    *   **应用层降级**: 若网关未配置，实现 `PIIScrubber` 类，集成 **Microsoft Presidio** 库进行实体识别与脱敏，严禁仅依靠简单正则。
    *   **限流**: 在网关层配置 `limit-req` (Leaky Bucket)，防止恶意刷量导致的 LLM 成本爆炸。
3.  **API 路由 (接口层)**:
    *   `POST /api/v1/intent/ingest`: 通用 Webhook 端点。
    *   `POST /api/v1/session/interact`: 交互式会话端点 (同步+异步)。
4.  **异步生产者 (基础设施层)**:
    *   实现 `RedisStreamProducer`。
    *   将 `IntentIR` JSON 对象推送到流键 `events:intent:created`。

## 4. 技术规格

### 4.1 Intent IR Schema
```python
class IntentIR(BaseModel):
    # 幂等性：基于 source_metadata 生成确定性哈希
    intent_id: UUID 
    raw_content: str
    source_metadata: SourceMetadata
    created_at: datetime
    # ... 其他字段

    @classmethod
    def generate_id(cls, metadata: SourceMetadata) -> UUID:
        # 使用 UUID v5 确保同一来源的意图 ID 相同
        namespace = UUID("...") # 定义系统命名空间
        return uuid5(namespace, f"{metadata.source_type}:{metadata.source_id}")
```

### 4.2 处理流程
1.  **Request**: 客户端发送 JSON。
2.  **Validate**: Pydantic 确认 Schema。
3.  **Mask**: 调用 `PIIScrubber.clean(request.raw_content)`。
4.  **Enrich**: 添加 `ingested_at` 时间戳。
5.  **Queue**: 执行 `redis.xadd("events:intent:created", intent_ir.model_dump_json())`。
6.  **Respond**: 返回 202 Accepted `{"task_id": "..."}`。

### 4.3 Redis Stream Keys
*   `events:intent:created`

## 5. 验证步骤
1.  **单元测试**:
    *   测试 `PIIScrubber` 对包含敏感信息的样本进行脱敏。
    *   测试 `IntentIR` 对缺失字段的校验报错。
2.  **集成测试**:
    *   调用 `POST /ingest`。验证返回 202 状态码。
    *   检查 Redis: 执行 `XREAD COUNT 1 STREAMS events:intent:created 0` 确认消息存在。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                                           | 修订人          |
| :--------- | :--- | :----------------------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.2 | 根据审计报告优化：增加 PendingClarification 状态池及意图确认状态机 | Tech Veteran AI |
| 2025-12-23 | v1.1 | 根据审计报告优化：引入 UUID v5 幂等性，集成 Presidio 脱敏及限流    | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                                           | AI Scribe       |

