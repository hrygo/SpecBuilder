# AI Coding Spec: 基础设施搭建 (Spec-010)

> **角色**: 系统架构师 / DevOps 工程师
> **任务**: 初始化项目基础设施与基础服务

## 1. 背景与目标
我们正在构建 **Spec Builder**，一个意图治理平台。本任务的目标是搭建 **基础层 (Foundation Layer)**：包括 Docker 容器化环境、数据库模式管理以及核心的 FastAPI 应用结构。这是支撑所有后续服务的基础。

## 2. 核心规则
*   **技术栈**: Python 3.11, FastAPI, PostgreSQL (pgvector), Redis (Streams), APISIX (网关).
*   **容器化**: 所有外部依赖使用 Docker Compose 管理。
*   **包管理**: 推荐使用 `poetry` 进行 Python 依赖管理。
*   **架构**: 清洁架构 (Clean Architecture) 或标准的 FastAPI 分层结构。
*   **配置**: 遵循 12-factor app 原则 (基于环境变量)。

## 3. 实现步骤

1.  **项目骨架**:
    *   初始化 `poetry` 项目。
    *   创建目录结构: `src/`, `tests/`, `scripts/`, `deploy/`。
2.  **Docker Compose (`docker-compose.yml`)**:
    *   **Postgres**: 使用 `pgvector/pgvector:pg16` 镜像。暴露端口 5432。
    *   **Redise**: 使用 `redis:alpine`。启用 AOF 持久化。
    *   **APISIX**: 
        *   运行于 **Standalone Mode**（无 ETCD，降低复杂度）。
        *   使用 `deploy/apisix.yaml` 进行声明式路由定义。
    *   **MinIO** (可选/后续): 用于存储文件制品。
3.  **数据库配置**:
    *   设置 `sqlalchemy` (Async 引擎) 和 `alembic` 用于迁移。
    *   编写初始迁移脚本，确保 `Vector` 扩展已启用。
4.  **应用入口**:
    *   创建 `src/main.py`。
    *   设置 `Lifespan` 事件以初始化 DB 连接池和 Redis 客户端。
5.  **可观测性**:
    *   配置 `structlog` 进行结构化的 JSON 日志记录。
    *   设置基础的 `/health` 端点，检查 DB 和 Redis 的连通性。

## 4. 技术规格

### 4.1 依赖定义 (pyproject.toml)
*   `fastapi`, `uvicorn[standard]`
*   `sqlalchemy`, `asyncpg`, `alembic`
*   `redis`
*   `pydantic-settings`
*   `langgraph` (占位符)
*   `structlog`

### 4.2 配置模型 (`src/config.py`)
```python
class Settings(BaseSettings):
    DATABASE_URL: PostgresDsn
    REDIS_URL: RedisDsn
    LOG_LEVEL: str = "INFO"
    ENV: str = "dev"
```

### 4.5 多租户隔离 (RLS)
*   所有表（如 `specs`, `intents`）必须启用 `ROW LEVEL SECURITY`。
*   创建 `tenant_id` 字段并建立 Policy：`FOR ALL TO authenticated USING (tenant_id = current_setting('app.current_tenant_id'));`。

## 5. 验证步骤
1.  运行 `docker-compose up -d`。检查所有容器状态为 `healthy`。
2.  运行 `poetry run uvicorn src.main:app --reload`。
3.  访问 `GET http://localhost:8000/health`。
    *   预期响应: `{"status": "ok", "db": "connected", "redis": "connected"}`。
4.  运行 `alembic upgrade head` 并验证无错误。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                                                | 修订人          |
| :--------- | :--- | :---------------------------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.1 | 根据审计报告优化：明确 APISIX Standalone 模式及 Postgres RLS 多租户隔离 | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                                                | AI Scribe       |

