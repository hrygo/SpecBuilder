# 规格索引与追溯矩阵 (Spec Index & Traceability)

## 1. 规格索引 (Spec Index)

### 基础设施
- [Spec-010: 基础设施搭建 (APISIX Standalone)](./Spec_010_Infrastructure.md)
- [Spec-012: 可观测性 (Otel/Metrics)](./Spec_012_Observability.md)
- [Spec-013: 成本控制与熔断](./Spec_013_Cost_Control.md)

### 核心后端与 AI
- [Spec-101: 意图采集引擎 (幂等/PII/限流)](./Spec_101_Intent_Ingestion.md)
- [Spec-102: 抄书员智能体 (分片摘要)](./Spec_102_Scribe_Agent.md)
- [Spec-103: 智能自愈引擎 (PG 持久化)](./Spec_103_Self_Healing_Loop.md)
- [Spec-104: 歧义检查器 (Aho-Corasick)](./Spec_104_Ambiguity_Linter.md)
- [Spec-105: 资产治理 (事务控制)](./Spec_105_Asset_Governance.md)
- [Spec-106: 逆向工程处理器 (Tree-sitter/Diff)](./Spec_106_Reverse_Engineer.md)

### 前端
- [Spec-201: Web 控制台 MVP (SSE 实时)](./Spec_201_Web_Console_MVP.md)

---

## 2. PRD 覆盖率矩阵 (PRD Coverage Matrix)

### 2.1 功能需求 (User Stories)

| PRD Req ID | Description          | Covered By Spec(s) | Status    | Notes                      |
| :--------- | :------------------- | :----------------- | :-------- | :------------------------- |
| **US-001** | 抄书员草稿生成       | Spec-102           | ✅ Covered | 核心 Agent 逻辑            |
| **US-002** | 自动自愈修复         | Spec-103           | ✅ Covered | 循环与校验器               |
| **US-003** | 混合输出格式         | Spec-102           | ✅ Covered | Prompt 约束                |
| **US-004** | 置信度评分           | Spec-102           | ✅ Covered | 包含在 State Schema 中     |
| **US-005** | 实时高亮             | Spec-201, 101      | ✅ Covered | 通过 SSE 推送 (隐含在 101) |
| **US-006** | 接受/拒绝变更        | Spec-201           | ✅ Covered | UI 交互                    |
| **US-007** | 结构化表单 (OpenAPI) | Spec-202           | ⚠️ Planned | MVP Specs 中暂未详细设计   |
| **US-010** | 歧义统计检查         | Spec-104           | ✅ Covered | Linter 规则                |
| **US-013** | 冻结与版本化         | Spec-105           | ✅ Covered | DB 与 Git 逻辑             |
| **US-017** | 逆向工程 (Code)      | Spec-106           | ✅ Covered | Tree-sitter 静态扫描       |
| **US-018** | 逆向工程 (DDL)       | Spec-106           | ✅ Covered | SQLGlot 解析               |
| **US-019** | 差异对比 (Diff)      | Spec-106           | ✅ Covered | 意图漂移检测               |

### 2.2 非功能需求 (NFRs)

| PRD Section   | Requirement               | Covered By Spec(s)    | Status    |
| :------------ | :------------------------ | :-------------------- | :-------- |
| **§4.1**      | 延迟 < 200ms              | Spec-101 (异步)       | ✅ Covered |
| **§4.3**      | PII 脱敏                  | Spec-101 (网关+应用)  | ✅ Covered |
| **§4.4**      | 安全限流                  | Spec-101 (网关 limit) | ✅ Covered |
| **§5.2**      | 技术栈 (FastAPI/Postgres) | Spec-010              | ✅ Covered |
| **NFR Audit** | 可观测性 (Trace/Metrics)  | Spec-012              | ✅ Covered |
| **NFR Audit** | 成本控制与熔断            | Spec-013              | ✅ Covered |
| **Database**  | 多租户隔离 (RLS)          | Spec-010              | ✅ Covered |


## 3. 差距分析 (Gap Analysis)
*   **结构化表单编辑器 (US-007, 008)**: Spec-202 已识别为需求，但未包含在此次 MVP 详细设计中。优先用 Markdown 编辑。
*   **MCP 服务器 (NFR §4.3.1)**: Spec-106 已列入计划但尚未编写。对“消费”至关重要，但对“生成”不是阻塞性的。
