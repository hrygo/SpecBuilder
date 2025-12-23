# 开发计划 (Development Plan): Spec Builder MVP

> **状态**: Generated
> **日期**: 2025-12-23

## 1. 概览
本计划优先构建 "采集 -> 生成 -> 评审" 的核心流水线。

## 2. Sprint 待办列表

| Spec ID | Category     | Feature Name                | Priority | Status    | Dependencies | PRD Refs     |
| :------ | :----------- | :-------------------------- | :------- | :-------- | :----------- | :----------- |
| **010** | **Infra**    | **基础设施搭建 (APISIX)**   | **P0**   | **Ready** | None         | §5.1, §5.2   |
| **011** | Infra        | 网关安全 (PII 插件)         | P1       | Planned   | Spec-010     | §4.3, §4.4   |
| **012** | Infra        | **可观测性 (Otel/Metrics)** | **P1**   | **Ready** | Spec-010     | NFR Audit    |
| **013** | Infra        | **成本控制与熔断**          | **P1**   | **Ready** | Spec-102     | NFR Audit    |
| **101** | **Core**     | **意图采集 (幂等/PII)**     | **P0**   | **Ready** | Spec-010     | §5.3.1, §5.4 |
| **102** | **Core**     | **抄书员Agent (分片摘要)**  | **P0**   | **Ready** | Spec-101     | §3.2.1       |
| **103** | Core         | 智能自愈 (PG 持久化)        | P0       | Ready     | Spec-102     | §3.2.1.3     |
| **104** | Quality      | 歧义检查 (Aho-Corasick)     | P1       | Ready     | None         | §3.2.2       |
| **105** | Core         | 资产治理 (事务控制)         | P1       | Ready     | Spec-010     | §3.2.3       |
| **201** | **Frontend** | **Web控制台 (SSE 实时)**    | **P0**   | **Ready** | Spec-101     | §3.2.6.2     |
| **106** | Core         | 逆向工程 (Tree-sitter)      | P1       | Ready     | Spec-105     | §3.2.4       |


## 3. 实施顺序
1.  **第一阶段: 奠基 (Sprint 1)**
    *   实现 **Spec-010** (Docker/FastAPI/DB)。
    *   实现 **Spec-101** (采集 API & 模型)。
2.  **第二阶段: 大脑 (Sprint 2)**
    *   实现 **Spec-102** (基础抄书员)。
    *   实现 **Spec-201** (基础 UI 以触发展示结果)。
3.  **第三阶段: 质量与治理 (Sprint 3)**
    *   实现 **Spec-103** (自愈)。
    *   实现 **Spec-104** (Linter)。
    *   实现 **Spec-105** (版本冻结)。
    *   实现 **Spec-106** (逆向工程与差异分析)。

## 4. 资源分配
*   **后端/AI**: 聚焦 101, 102, 103。
*   **前端**: 聚焦 201。
