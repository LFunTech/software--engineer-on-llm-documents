# 实现菜谱（前端 + 后端）

本模块提供按规格落地的分步流程与提示语，适合在阅读基础与规划模块后使用。

## 通用原则
- 先要计划：修改前先让助手给 3–5 步计划。
- 小步提交：一轮只处理一个组件或一个端点。
- 遵循现有模式：样式、日志、错误处理、测试保持一致。
- 每次实质改动都要跑测试。

## 前端流程（React/SPA 类）
### 1) 发现文件与模式
- 提示： “列出与 <feature> 相关的组件、hooks、样式文件及路径。”
- 命令辅助：`rg <keyword> src`，`ls src/components`，`rg -n "StatusBadge" src`。

### 2) 创建或扩展组件
- 提示： “在 `src/components/StatusBadge.tsx` 添加组件，props `{ status, message }`，颜色 green=ok、yellow=degraded、red=failed，保持现有样式 token。”
- 要求补丁预览；用 Codex CLI `apply_patch`、Cursor 行内、或 Claude diff 逐块应用。

### 3) 连接数据/状态
- 提示： “演示如何请求 `/status` 并传入 `StatusBadge`，包含加载/错误状态，复用已有 fetch 工具。”
- 先查是否已有 hook/工具，能复用就不造轮子。

### 4) 测试与故事
- 提示： “为 `StatusBadge` 写 Jest/RTL 测试：渲染文本、按状态上色、错误兜底。”
- 提示： “添加 Storybook 控件 story，支持 `status` 与 `message`，文件放 `StatusBadge.stories.tsx`。”
- UI 自动化：如需端到端覆盖，参考 [docs/vc-playwright-guide.md](vc-playwright-guide.md) 生成 Playwright 用例（加载/成功/失败三态）。

### 5) 验收清单
- props 有校验/类型
- 覆盖加载与错误态
- 可访问性：必要时加 aria-label/role
- 测试/Story 已补；命令示例：`npm test StatusBadge`

## 后端流程（API/服务）
### 1) 路由与处理器地图
- 提示： “列出与 <feature> 相关的路由文件和模式，建议将 `GET /status` 放在哪。”
- 命令辅助：`rg -n "router" src`，`ls src/api`。

### 2) 实现处理器
- 提示： “实现 `GET /status` 返回 `{ service, build, gitSha, time }`，复用 logger，500 用现有错误助手。”
- 确认校验/错误处理遵循当前工具（搜索 `validate`、`errorResponse`）。

### 3) 数据与配置
- 提示： “说明构建信息目前从哪里读取（env/config/文件），给出注入 `gitSha` 的最小方式。”
- 机密与配置集中管理，避免硬编码。

### 4) 测试
- 提示： “写 `GET /status` 的 Jest 测试：配置正常返回 200+数据，配置缺失走 500；复用现有测试助手。”
- 命令示例：`npm test status.test.ts`（按项目调整）。

### 5) 日志/指标清单
- 仅在需要的地方加请求/响应日志
- 错误包含关联 ID/请求 ID（如全局已使用）
- 避免在正常流大量输出

## 提示语模板（可改）
- “规划在 <file> 实现 <feature> 的 4 步，只列要改的文件。”
- “生成 <file> 的最小补丁，限制 50 行内，不引入新依赖。”
- “列出该组件/处理器的边界情况，并为前三个写测试。”
- “只给 diff，不要解释。”
> 工具配合：终端补丁与搜索见 [docs/vc-codex-guide.md](vc-codex-guide.md)；多文件 diff 见 [docs/vc-claude-guide.md](vc-claude-guide.md)；行内改动见 [docs/vc-cursor-guide.md](vc-cursor-guide.md)；UI 自动化见 [docs/vc-playwright-guide.md](vc-playwright-guide.md)；OpenSpec CLI 速查 [docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)。

## 多工具协作
- Codex CLI：快速搜索、小补丁、跑测试。
- Claude Code：预览多文件 diff，适合重构函数。
- Cursor：IDE 行内改动，交互式选择 hunk。

## 端到端案例：实现状态面板（含角色视角）
沿用规划模块的 `add-project-status-surface`，这里给具体实施步骤。

### 后端（后端开发）
1) 路由定位：`rg -n "router|route" src/api`，确认挂载点。
2) 起草处理器
   - 提示： “在 `src/api/status.ts` 添加 `GET /status`，返回 `{ service, build, gitSha, time }`；用现有 logger；500 用错误助手。”
3) 校验与日志
   - 提示： “添加对缺失配置的校验，失败时返回 500 并记录 requestId。”
4) 单测
   - 提示： “写 Jest 测试：配置正常→200，配置缺失→500；用现有 mock/logger 辅助。”
   - 命令：`npm test api/status.test.ts`

### 前端（前端开发）
1) 组件骨架
   - 提示： “在 `src/components/StatusBadge.tsx` 实现组件，props `{ status, message }`，颜色 green/yellow/red，保持样式 token。”
2) 数据接入
   - 提示： “在 `src/pages/StatusPage.tsx`（或等价文件）请求 `/status`，加载/错误态要显示；向 `StatusBadge` 传递数据。”
3) Story 与测试
   - 提示： “为 StatusBadge 写 Storybook 控件；写 RTL 测试覆盖三种状态与错误文案。”
   - 命令：`npm test StatusBadge.test.tsx`

### QA/测试
1) 场景检查：确认后端/前端都覆盖成功+失败（超时、配置缺失）。
2) 回归套件：运行 `npm test`（或分组）并记录结果。

### 运维/发布
1) CI 片段：在 CI 配置加入 `openspec validate add-project-status-surface --strict`、lint、tests。
2) 上线核查：部署到预发后访问 `/status`，验证 payload；检查 UI 显示与日志异常。

### 评审要点
- 规格与实现一致，未新增未定义行为。
- 日志/错误处理符合现有模式。
- 测试覆盖成功+主要失败路径，命令可复现。
- 验收前再跑：`openspec validate <id> --strict`，并记录测试命令/结果（OpenSpec CLI 详见 [docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)）。

## 自检（提审前）
- diff 小而聚焦
- 测试已更新并通过
- 行为符合规格场景
- 无多余调试日志或 TODO
