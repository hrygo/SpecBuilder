# AI Coding Spec: Web 协同控制台 MVP (Spec-201)

> **角色**: 前端工程师
> **任务**: 实现主评审界面 (会话与规格)

## 1. 背景与目标
Web 控制台是 "人机协同 (Human-in-the-Loop)" 发生的场所。MVP 聚焦于 "会话视图 (Session View)"，该视图显示聊天日志（左）、规格草稿（中）和待办事项（右）。

## 2. 核心规则
*   **技术栈**: Next.js 14 (App Router), Tailwind CSS。
*   **实时性**: 强制使用 **SSE (Server-Sent Events)** 接收实时更新。
    *   **重连策略**: 实现指数退避 (Exponential Backoff) 的重连逻辑，确保连接稳定性。
    *   **状态管理**: 使用 Zustand 处理客户端状态 (Session 数据)。

## 3. 实现步骤
1.  **布局**: 创建包含侧边栏 (Nav) 和内容区的 `MainLayout`。
2.  **会话视图 (`/session/[id]`)**:
    *   **分栏布局**: 使用 `react-resizable-panels`。
    *   **ChatPanel**: 渲染消息列表 (支持 Markdown)。
    *   **SpecPanel**:
        *   Tab 1: Markdown 预览 (`react-markdown`)。
        *   Tab 2: 编辑表单 (未来功能/占位)。
    *   **ActionPanel**: "澄清问题" 列表。
3.  **交互**:
    *   点击 Action Item -> SpecPanel 滚动到相关章节。

## 4. 技术规格
### 4.1 路由
*   `/dashboard`: 最近会话列表。
*   `/session/[id]`: 工作区。

### 4.2 组件树
*   `SessionWorkshop`
    *   `ChatStream`
    *   `SpecViewer` (Monaco Editor 只读或 Markdown)
    *   `CopilotSidebar`

## 6. 修订记录 (Revision History)

| 日期       | 版本 | 修订内容                                          | 修订人          |
| :--------- | :--- | :------------------------------------------------ | :-------------- |
| 2025-12-23 | v1.1 | 根据审计报告优化：强制使用 SSE 及指数退避重连策略 | Tech Veteran AI |
| 2025-12-23 | v1.0 | 初次生成                                          | AI Scribe       |

