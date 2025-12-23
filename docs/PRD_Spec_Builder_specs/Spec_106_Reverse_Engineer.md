# AI Coding Spec: 逆向工程处理器 (Spec-106)

> **角色**: AI 架构师 / 后端工程师
> **任务**: 实现从现有代码和数据库 Schema 提取规格的自动化逻辑

## 1. 背景与目标
逆向工程器是实现 "代码即规格 (Spec as Code)" 闭环的关键。其目标是扫描现有系统的源代码和 DDL 文件，自动提取接口契约 (OpenAPI) 和数据模型，并将其转化为平台标准的 Intent IR，最终生成或更新规格文档 (Spec Profile)。

## 2. 核心规则
*   **多语言支持**: 采用 **Tree-sitter** 抽象语法树解析，首期支持 Python, Java, Go。
*   **非侵入性**: 全静态分析，不需要运行用户代码。
*   **语义映射**: 不仅仅是提取语法，由于代码中可能缺乏描述，需结合 LLM 对提取出的函数名和注释进行语义增强。
*   **差异检测 (Diffing)**: 必须对比提取出的规格与现有资产中的规格，识别 "意图漂移"。

## 3. 实现步骤

1.  **源码扫描层 (Source Scanner)**:
    - 集成 `GitPython` 从指定 Git 分支拉取或读取代码。
    - 使用 **Tree-sitter** 构建 AST，定位 Web 框架（如 FastAPI, Spring Boot, Gin）的路由装饰器与控制器类。
2.  **契约提取器 (Contract Extractor)**:
    - **Route Logic**: 提取 URL Path, HTTP Method, Request Body (Schema), Response Body.
    - **SQL Logic**: 使用 `sqlglot` 解析 DDL 文件，提取表名、字段类型、约束与外键关系。
3.  **意图增强器 (Intent En richer)**:
    - 将提取出的粗糙结构推送到 LLM。
    - **Prompt**: "根据以下代码片段和注释，生成符合 OpenAPI 规范的描述字段，并判断其核心业务意图。"
4.  **规格生成与对比 (Spec Reconciler)**:
    - 将提取结果转换为 `Intent IR`。
    - 若系统中已存在相同 `Spec ID` 的规格，执行深度对比，生成差异报告。

## 4. 技术规格

### 4.1 核心库选型
- **AST Parsing**: `py-tree-sitter` (支持多语言语法补丁)。
- **SQL Parsing**: `sqlglot` (高性能多方言 SQL 解析)。
- **IR Mapping**: 遵循 PRD §5.4 的 Intent IR 结构。

### 4.2 逆向映射逻辑流
```python
class ReverseEngineerService:
    def process_repo(self, repo_url: str):
        # 1. Clone/Fetch code
        # 2. Walk through files
        # 3. If file matches pattern (.java, .go, .py):
        #    - call ast_analyzer.extract_endpoints(file_content)
        # 4. If file matches .sql/ddl:
        #    - call sql_analyzer.extract_schema(file_content)
        # 5. Aggregate findings -> LLM Refinement -> Intent IR
        pass
```

### 4.3 差异分析 (Diff Engine) 标准
- **Breaking Changes**: 接口路径变更、必填字段增加、字段类型不匹配（标记为 🔴）。
- **Compatible Changes**: 描述更新、可选字段增加（标记为 🟡）。

## 5. 验证步骤
1.  **静态测试**: 针对一个标准的 FastAPI 项目代码，验证是否能成功提取出 OpenAPI JSON 结构。
2.  **DDL 测试**: 针对包含 RLS 策略的 Postgres SQL 文件，验证是否能提取出带有权限约束的实体定义。
3.  **对比测试**: 手动修改 Spec 资产，验证逆向工程器是否能准确报告 "意图漂移" (Intent Drift)。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                      | 修订人          |
| :--------- | :--- | :-------------------------------------------- | :-------------- |
| 2025-12-23 | v1.0 | 初次生成：涵盖 Tree-sitter 选型与逆向映射逻辑 | Tech Veteran AI |
