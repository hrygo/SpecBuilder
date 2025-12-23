# AI Coding Spec: 歧义检查器 (Spec-104)

> **角色**: 后端工程师
> **任务**: 实现用于规格质量控制的静态分析规则

## 1. 背景与目标
**歧义检查器 (Ambiguity Linter)** 是一个确定性的质量门控。它扫描规格的 Markdown/文本内容，查找导致下游幻觉或理解偏差的“禁用词”（如“快速”、“用户友好”、“大约”）。

## 2. 核心规则
*   **高性能**: 必须在 < 2s 内完成。强制使用 **Aho-Corasick** 算法进行全文本扫描（推荐库：`pyahocorasick`）。
*   **可扩展性**: 规则应定义在 JSON/YAML 配置文件中，而非硬编码。

## 3. 实现步骤
1.  **规则字典**: 创建 `config/linter_rules.yaml`。
    *   分类: `Ambiguity` (歧义), `Subjectivity` (主观), `Untestable` (不可测)。
2.  **Linter 引擎与持久化**:
    *   类 `AmbiguityLinter`。
    *   **自动机持久化**: 
        *   初始化时检查 `cache/linter_automaton.pkl`。
        *   如果存在且 `linter_rules.yaml` 未更新，直接 `pickle.load`。
        *   否则，从 `config/linter_rules.yaml` 加载并从构建、序列化。
    *   方法 `lint(text: str) -> List[LintIssue]`。
    *   `LintIssue`包含: `line_number`, `matched_text`, `suggestion`。
3.  **集成**:
    *   将此功能作为函数依赖暴露给 `Spec-103` (Validator Agent)。

## 4. 技术规格
### 4.1 规则格式
```yaml
rules:
  - id: "AMB-001"
    pattern: "(?i)user-friendly"
    suggestion: "请定义具体的点击次数限制或 UI 标准"
    severity: "warning"
```

## 5. 验证
*   单元测试: 输入文本 "This API is user-friendly"。断言返回 AMB-001。

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                                       | 修订人          |
| :--------- | :--- | :------------------------------------------------------------- | :-------------- |
| 2025-12-23 | v1.1 | 根据审计报告优化：强制 Aho-Corasick 算法并增加自动机序列化缓存 | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                                       | AI Scribe       |

