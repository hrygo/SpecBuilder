# AI Coding Spec: 资产治理与版本控制 (Spec-105)

> **角色**: 后端工程师
> **任务**: 实现规格版本控制、冻结与 Git 同步

## 1. 背景与目标
规格是资产。它们必须拥有唯一的 ID，不可变的版本 ("Frozen" 冻结状态)，并能够同步到 Git 仓库。

## 2. 核心规则
*   **不可变性**: 一旦 `status='frozen'`，DB 中的行就不能再更新。必须插入新版本。
*   **Git 同步**: 操作应当异步执行，避免阻塞 HTTP 请求。

## 3. 实现步骤
1.  **Spec 实体**: 更新 DB Schema。
    *   `spec_id` (UUID), `human_id` (如 SPEC-AUTH-1.0), `version`, `content`, `status`。
2.  **ID 生成器 (Human ID Generator)**:
    *   **算法**: `SPEC-{module}-{version}-{hash}`。
    *   **Hash 逻辑**: 取 `SHA256(intent_id + content)` 的前 8 位 16 进制字符作为后缀，确保在同一意图和内容下 ID 的确定性。
3.  **冻结服务 (Freeze Service)**:
    *   `freeze_spec(draft_id) -> frozen_spec`。
    *   **并发与原子性保证**:
        1.  启动事务。
        2.  执行 `SELECT * FROM specs WHERE module=:m AND is_current=true FOR UPDATE;` (加排他锁，防止并发冻结冲突)。
        3.  `UPDATE specs SET is_current=false WHERE module=:m AND is_current=true;`
        4.  `INSERT INTO specs (...) VALUES (...);`
        5.  提交事务。
    *   检查 "质量门控" (参考 Spec-103/104)。
4.  **Git 同步与容灾**:
    *   使用 `GitPython` 将文件异步推送到 Git 仓库。
    *   **DLQ 处理**: 如果 Git Push 失败（由网络或认证导致），将任务推入 **Redis DLQ (Dead Letter Queue)**，由后台 Worker 进行指数退避重试。界面应展示“正在同步到云端”状态。

## 4. 技术规格
### 4.1 Spec 表结构
```sql
CREATE TABLE specs (
    id UUID PRIMARY KEY,
    module VARCHAR(50),
    version VARCHAR(20),
    human_id VARCHAR(100) UNIQUE,
    is_current BOOLEAN DEFAULT TRUE,
    is_archived BOOLEAN DEFAULT FALSE, -- 冷热数据分离：老旧版本标记为已存档
    content TEXT,
    lock_hash VARCHAR(64) -- 内容的 SHA256
);
CREATE INDEX idx_specs_module_current ON specs(module) WHERE is_current = TRUE;
```

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                                             | 修订人          |
| :--------- | :--- | :------------------------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.2 | 根据审计报告优化：明确 Human ID 哈希算法，增加并发锁控制及冷数据标记 | Tech Veteran AI |
| 2025-12-23 | v1.1 | 根据审计报告优化：增加数据库事务保证及 Git 同步 DLQ 容灾机制         | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                                             | AI Scribe       |

