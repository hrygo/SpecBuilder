# Spec Builder 深度覆盖分析报告

## 1. 背景与目标
本项目旨在构建 **Spec Builder**，一个企业级研发资产治理平台。其核心价值是将非结构化的需求意图（聊天记录、会议纪要、工单）转化为结构化、可执行的规格资产（OpenAPI + Gherkin）。本分析旨在将 PRD `PRD_Spec_Builder.md` 拆解为可供 AI 编码的原子级规格 (Specs)。

## 2. 功能拆解 (Functional Decomposition)

### 2.1 核心领域：意图处理与生成
*   **意图采集 (Ingestion)**: 必须处理多模态输入 (Webhook, IDE 流)。
    *   *隐形状态*: 数据持久化**前**必须进行 PII 脱敏。
    *   *逻辑流*: 接收 -> PII 脱敏 -> 持久化原始数据 -> 触发异步处理。
*   **抄书员智能体 (Scribe Agent)**:
    *   *逻辑*: 提示词管理、上下文窗口控制、提取 "核心实体"。
*   **智能自愈循环 (Self-Healing Loop)**:
    *   *复杂逻辑*: 一个包含 Validator (校验器) -> Reflector (反思器) -> Corrector (修正器) 的状态机 (LangGraph)。需要严格的预算控制（最大循环 3 次）。

### 2.2 资产治理
*   **歧义检查器 (Ambiguity Linter)**:
    *   *逻辑*: 基于 "禁用词" 字典和启发式规则的静态文本分析。
*   **版本控制与同步 (Versioning & Sync)**:
    *   *逻辑*: 语义化版本控制、Git 提交生成、SHA 计算。
    *   *状态*: "草稿 (Draft)" vs "冻结 (Frozen)"。冻结的规格在 DB 中不可变。

### 2.3 消费接口
*   **MCP 服务器**: 向其他 AI 智能体暴露规格上下文。
*   **Web 控制台**: 人机协同评审界面。

## 3. 架构与非功能需求分析 (NFRs)

### 3.1 基础设施 (§5.2)
*   **技术栈**: Python FastAPI, LangGraph, PostgreSQL (JSONB + Vector), Redis Streams (异步削峰), APISIX (网关).
*   **数据存储**: 使用 PGVector 存储向量索引，JSONB 存储结构化意图。

### 3.2 非功能需求提取
*   **性能**:
    *   采集 API 延迟 < 200ms (必须异步处理)。
    *   Linter 检查 < 2s。
*   **安全性**:
    *   **PII 脱敏**: 关键。必须在网关边缘或入口处立即执行。
    *   **数据主权**: 本地向量存储，严禁将未清洗的上下文泄露给公有模型。
*   **可靠性**:
    *   使用 Redis Streams 缓冲突发 Webhook 流量。
    *   LLM API 调用的重试机制。

### 3.3 边缘情况与风险
*   **LLM 幻觉**: 生成的规格引用了不存在的 API。 -> *缓解*: Validator Agent 校验。
*   **冲突解决**: 用户在 Web 端编辑的同时 Agent 正在修正。 -> *策略*: PRD 建议采用交互式确认流，但在 MVP 中可能需要简单的乐观锁或“最后写入胜出”。
*   **超大负载**: 会议纪要超过 10k token。 -> *策略*: 分块 (Chunking) 或先摘要后处理。

## 4. 拟定原子规格列表 (Proposed Atomic Specs)

### 分类：基础设施 (0xx)
*   **Spec-010-Infrastructure**: 基础设施搭建。Docker Compose (Postgres, Redis, MinIO), Python 环境。
*   **Spec-011-Gateway_Security**: 网关与安全。APISIX 配置, PII 脱敏工具类。

### 分类：核心后端 (1xx)
*   **Spec-101-Intent_Ingestion**: 意图采集引擎。Webhook (`/ingest`) 和 Session (`/interact`) 接口, `IntentIR` 数据模型。
*   **Spec-102-Scribe_Agent**: 抄书员智能体。基于 LangGraph 的草稿生成逻辑。
*   **Spec-103-Self_Correction_Loop**: 智能自愈循环。Validator/Corrector 逻辑, 预算控制。
*   **Spec-104-Ambiguity_Linter**: 歧义检查器。基于规则的静态分析引擎。
*   **Spec-105-Spec_Versioning**: 资产治理。Git 集成, Spec ID 生成, 版本冻结逻辑。
*   **Spec-106-Reverse_Engineer**: 逆向工程处理器。Tree-sitter 源码扫描, SQL DDL 解析, 意图映射与差分对比。

### 分类：接口与前端 (2xx/3xx)
*   **Spec-201-Web_Console_MVP**: Web 控制台 MVP。Next.js 三栏式布局实现。
*   **Spec-202-MCP_Server**: MCP 协议实现。暴露 `spec://` 资源。

## 5. 优先级排序
1.  **Infra (010)**: 基础。
2.  **Ingestion (101)**: 数据入口。
3.  **Scribe (102)**: 核心价值。
4.  **Correction (103)**: 质量保证。
5.  **Web (201)**: 用户交互。
6.  **Governance (104, 105)**: 资产化。

## 6. 审计缺陷修复计划 (Gap Resolution Plan)

基于 `_Audit_Report_Coverage.md` 的审计反馈，现有的 Specs 存在多处“耍花枪”和“架构性缺失”。以下是针对性的修正方案：

### 6.1 阻断性漏洞修正
*   **[Spec-102] 解决 Context Explosion**: 增加 `SummaryChain` 前置处理，针对 >10k tokens 的输入自动执行 Map-Reduce 摘要，避免 LLM 崩溃。
*   **[Spec-101] 采集幂等性**: 将 `intent_id` 从随机 UUID 改为基于 `source_type + source_id` 的 **Deterministic Hash (UUID v5)**，防止 Webhook 重跑产生脏数据。
*   **[Spec-105] 资产冻结原子性**: 强制在 `freeze_spec` 逻辑中使用 **SQLAlchemy 异步事务** (`BEGIN...COMMIT`)，确保版本切换的原子性。

### 6.2 拒绝“耍花枪”与 NIH
*   **[Spec-101] PII 脱敏降级**: 放弃应用层手写正则。优先使用 **APISIX `data-mask` 插件**（网关层拦截）；若在应用层处理，必须使用 **`presidio`** 工业级库。
*   **[Spec-101] 引入 Rate Limiting**: 在网关层配置 APISIX **`limit-req`** 插件（漏桶算法），或在应用层使用 **`redis-cell`**，防止 LLM 被恶意刷爆。
*   **[Spec-201] 实时性确定**: 废弃“轮询”描述，明确使用 **SSE (Server-Sent Events)**，并定义前端的 **Exponential Backoff 重连策略**。
*   **[Spec-104] 算法锁定**: 明确要求使用 **`pyahocorasick`** (Aho-Corasick) 库处理关键词匹配，而非原生 `re`。
*   **[Spec-010] APISIX 配置规范**: 明确采用 **Declarative Configuration (apisix.yaml)**，通过 Git 管理路由规则。

### 6.3 缺失的硬核 NFR 补全
*   **[New Spec-012] 可观测性 (Observability)**:
    *   **Metrics**: 监控 `token_usage` (按租户), `scribe_latency`, `queue_depth`。
    *   **Tracing**: 引入 OpenTelemetry，全链路追踪 `trace_id` (从 Webhook 到异步 Worker)。
*   **[New Spec-013] 成本熔断 (Cost Control)**:
    *   实现 `TokenBucketLimiter`，每轮 AI 循环检查租户剩余预算，超标即熔断。
*   **[Spec-105] Git Sync 容灾**:
    *   设计 **Dead Letter Queue (DLQ)** 处理 Git 操作超时，支持异步重试。

### 6.4 智能体持久化
### 6.5 极致性能与多租户隔离 (V3 审计补充)
*   **[Spec-104] 自动机序列化**: 针对 Aho-Corasick 算法，增加预编译与序列化机制，避免在高并发、多规则场景下反复构建自动机。
*   **[Spec-010] 数据库多租户策略**: 明确采用 **Postgres Row Level Security (RLS)**，在数据库层面强制隔离租户数据，确保数据主权。
*   **[Spec-102] 鲁棒性增强**: 增加 **StreamingOutputParser**，处理 LLM 因 Token 限制或异常中断导致的截断输出，确保解析逻辑不崩溃。
*   **[New Spec-106] 逆向反馈闭环**: 补齐 PRD §3.2.4 定义的逆向工程功能。锁定 **Tree-sitter** 为跨语言解析引擎，解决从存量代码/数据库库表到标准规格的“回流”问题，确保 SSOT 闭环。

