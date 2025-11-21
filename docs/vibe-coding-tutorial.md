# 使用 OpenSpec 的 Vibe Coding：端到端教程

本教程说明如何用 Codex CLI、Claude Code、Cursor 等 vibe coding 工具配合 OpenSpec 完成长规格、前端、后端、测试和运维的工作流。可用于培训或自学的实操脚本。

## 适用对象与产出
- 适用：想用 AI 辅助并遵循 OpenSpec 的工程师和技术负责人。
- 产出：能写规格、能按规格落地代码、能自检验证，并遵守安全与格式要求。

## 使用方式
- 顺序：基础 → 规划 → 实现 →（按角色）测试 或 运维。
- 每个模块都是单独的详细指南：
  - 基础：`docs/vc-foundations.md`（环境准备、安全、上手练习）
  - OpenSpec 规划：`docs/vc-openspec-planning.md`（proposal、delta、校验）
  - 业务分析/BA：`docs/vc-ba-guide.md`（需求梳理、场景补全、验收清单）
  - 数据分析师：`docs/vc-data-analyst-guide.md`（数据理解、指标设计、查询/可视化、质量检查）
  - 大数据工程师：`docs/vc-bigdata-guide.md`（管道设计、数据契约、质量/回填、性能/监控）
  - 实现菜谱：`docs/vc-implementation-cookbook.md`（前端/后端落地流程）
  - 测试工程师：`docs/vc-testing-guide.md`（测试矩阵、夹具、生成/运行测试、案例）
  - 运维/发布工程师：`docs/vc-ops-guide.md`（CI/CD、上线核查、回滚、runbook、案例）
  - 工具详解：Codex 终端助手 `docs/vc-codex-guide.md`、Claude Code `docs/vc-claude-guide.md`、Cursor 行内助手 `docs/vc-cursor-guide.md`、OpenSpec CLI `docs/vc-openspec-tool-guide.md`、Playwright UI 自动化 `docs/vc-playwright-guide.md`
- 本页提供总览与速查，深度步骤请进入对应模块。

## 工具概览
- Codex CLI：终端优先，能跑命令和打补丁（当前环境）。
- Claude Code：IDE/桌面 AI，可查看文件、预览多文件 diff，并在沙箱里运行命令。
- Cursor：VS Code 分支，支持聊天、行内编辑、模型驱动的重构。
- OpenSpec：规格优先；proposal 在 `openspec/changes`，已落地规格在 `openspec/specs`。

## 快速准备（按需选择）
1) 仓库准备
   - Clone/更新仓库；运行 `openspec list`、`openspec list --specs` 了解现状。
   - 阅读 `openspec/AGENTS.md` 和 `openspec/project.md` 了解规约。
2) Codex CLI
   - 本环境已就绪，可直接跑命令并用 `apply_patch` 编辑。
3) Claude Code
   - 安装应用/插件，打开仓库根目录，授予读写权限。
4) Cursor
   - 安装 Cursor，打开仓库，开启 “Ask Cursor” 和行内编辑。
5) 账户与配置
   - 登录各自 AI 账号，确认模型权限（若有企业管控）。

## 安全与卫生
- 先读后写：优先使用只读命令（`ls`、`rg`、`cat`），审清文件再改。
- 避免破坏性命令：不要随意 `git reset --hard`、批量 `rm`。
- 小步快跑：先让 AI 给计划，再申请具体补丁；逐段检查 diff。
- 结果自证：改完跑测试/linters，不盲信生成代码。
- 真相来源：正式规格在 `openspec/specs`，变更在 `openspec/changes/<id>/`。

## 新手路线（0-1 天）
1) 安装与验证
   - 按 `docs/vc-foundations.md` 完成工具安装、登录、仓库权限检查。
   - 在仓库根目录运行：`openspec list`、`openspec list --specs`、`rg --files | head`。
2) 读规则
   - 打开 `openspec/AGENTS.md`、`openspec/project.md`，弄清命名、格式与禁忌。
3) 选择练习案例
   - 建议用文档里“项目状态面板”案例，覆盖前后端、测试与运维。
4) 先规划再动手
   - 按 `docs/vc-openspec-planning.md` 脚手架 change 目录，写 proposal、tasks、delta。
5) 分阶段实现
   - 依次完成后端 → 前端 → 测试 → 运维（见下方完整案例），每阶段都要跑验证。
6) 复盘
   - 记录遇到的边界、修改的文件与测试结果，为后续 PR/评审准备素材。

## OpenSpec 基础流程
1) 看上下文
   - `openspec list`，`openspec list --specs`，`rg -n "Requirement:|Scenario:" openspec`。
2) 需要创建变更时
   - 取动词开头的 `change-id`，如 `add-vibe-coding-tutorial`。
   - 脚手架：`mkdir -p openspec/changes/<id>/specs/<capability>/`。
   - 写 `proposal.md`、`tasks.md`、delta `spec.md`（用 `## ADDED|MODIFIED|REMOVED Requirements`，每条至少一个 `#### Scenario:`）。
3) 早验证
   - `openspec validate <id> --strict`。
4) 按任务实现，完成后勾选 `tasks.md`。
5) 提交前再次验证。

## 工具使用套路
### Codex CLI
- 优势：全仓搜索、精准补丁、跑测试，命令记录清晰。
- 提问套路：先要计划，再 `rg`/`cat` 查文件，最后请求小补丁。
- 示例：
  - “给出用现有模式添加 `GET /status` 的 3 步计划。”
  - “列出 `routes/` 中状态相关的路由和测试文件。”
  - “在 `api/status.ts` 起草返回构建信息的处理器，并附 Jest 测试。”

### Claude Code
- 优势：大文件推理、多文件 diff 预览、对话式调试。
- 流程：打开相关文件 → 请求修改 → 在 diff 预览中逐块应用。
- 示例： “参考 `openspec/changes/add-status-endpoint/specs/api/spec.md`，在 `src/api/status.ts` 实现 GET 端点并补一个最小单测。”

### Cursor
- 优势：IDE 内行内修改、快速重构、上下文可视化。
- 流程：选中代码 → “Ask Cursor” 指定改动 → 挑选需要的 hunk。
- 示例： “把这个 React 表单改用 `react-hook-form`，保持校验不变并补一个单测桩。”

## 生命周期脚本
### 1) 技术方案
- 输入：问题陈述、现有规格、约束。
- 步骤：
  1. 让 AI 只读总结相关规格与代码。
  2. 在 `openspec/changes/<id>/specs/<capability>/spec.md` 起草 delta。
  3. 用 AI 列举边界： “列出该 API 的失败模式并补场景。”
- 提示语： “按 `specs/api` 风格给出 3 种 API 契约方案并写利弊。”

### 2) 前端实现
- 输入：已批准的规格 + UI 验收口径。
- 步骤：
  1. 要文件/组件地图。
  2. 每组件一小补丁，保持样式与类型一致。
  3. 同步要求测试/Story 更新。
- 提示语：
  - “根据规格创建 `StatusBadge` 组件与 Story，允许 Tailwind，颜色由 props 控制。”
  - “写一个 Playwright 测试，断言 `status=failed` 时颜色正确。”

### 3) 后端实现
- 输入：API 规格 + 数据契约。
- 步骤：
  1. 要路由/处理器骨架。
  2. 加校验、日志、错误处理。
  3. 生成单测/集成测，连接夹具。
- 提示语：
  - “添加 `GET /status` 返回 `{ service, build, gitSha, time }`，复用 logger 和错误助手。”
  - “写覆盖成功/失败路径的 Jest 测试。”

### 4) 自动化测试
- 让 AI 枚举规格对应的单元/集成/e2e。
- 先要夹具/Mock，再要用例，再要断言。
- 提示语： “列出 `StatusBadge` 的前 5 个测试；输出一个 Jest 测试文件。”

### 5) 运维/DevOps
- 用 AI 起草 runbook、CI 步骤、部署校验。
- 提示语：
  - “增加运行 `openspec validate <id> --strict` 与 Jest 的 CI 步骤，给出 GitHub Actions 片段。”
  - “为新端点写回滚清单。”

## 完整案例：项目状态面板（含角色视角）
目标：添加后端状态端点与前端状态徽章，并建立测试与运维检查。

### 步骤总览
1) 新建变更（新人/开发）
   - 取 `change-id`: `add-project-status-surface`，capability: `status-surface`。
   - 命令：`mkdir -p openspec/changes/add-project-status-surface/specs/status-surface/`
   - 写 `proposal.md`（Why/What/Impact），`tasks.md`，并在 delta `spec.md` 中添加 ADDED 需求（成功/失败场景）。
   - 验证：`openspec validate add-project-status-surface --strict`
2) 后端实现（后端开发）
   - 提示示例： “在 `src/api/status.ts` 实现 `GET /status`，返回 `{ service, build, gitSha, time }`；复用 logger；500 走错误助手；补 Jest 单测。”
   - 命令（示例）：`npm test api/status.test.ts`
   - 产出：路由/处理器、校验、错误处理、单测。
3) 前端实现（前端开发）
   - 提示示例： “在 `src/components/StatusBadge.tsx` 实现组件，props `{ status, message }`，颜色 green/yellow/red；补 Storybook；RTL 测试。”
   - 命令（示例）：`npm test StatusBadge.test.tsx`
   - 产出：组件、Story、前端测试。
4) 测试补全（测试/QA）
   - 提示： “列出状态端点与徽章的未覆盖用例，再补 2 条测试并合并现有套件。”
   - 命令：`npm test`（或分组执行），确认无失败。
5) 运维与发布（运维/发布工程师）
   - 提示： “为 change `add-project-status-surface` 添加 CI 步骤：openspec 校验 + lint + tests；写部署前检查与回滚步骤。”
   - 产出：CI 片段（如 GitHub Actions）、上线核查清单、回滚步骤。
6) 汇总与交付（评审/负责人）
   - 再跑验证：`openspec validate add-project-status-surface --strict`
   - 汇总：变更影响、文件列表、测试结果、已知风险。

### 角色穿插要点
- 新人：严格按任务清单，小范围提交；遇到不确定性先补场景或提问，不要凭空猜。
- 后端：优先复用日志/错误/校验工具，避免新模式；测试覆盖成功+主要失败路径。
- 前端：遵循现有设计系统/样式 tokens；UI 测试至少覆盖文案与状态色。
- 测试/QA：从规格提取场景，补充高风险边界（配置缺失、网络失败）。
- 运维：CI 先跑 openspec，再跑 lint/tests；上线前跑健康检查，预备回滚命令与验证步骤。
- 评审：检查规格与实现一致性，确认任务清单与验证命令执行过。

## Prompt 库（可改）
- “总结 <feature> 相关文件并提示风险（数据、性能、安全）。”
- “写满足此要求的最小补丁，限单文件。”
- “为此能力给出单元/集成/e2e 的测试矩阵。”
- “基于这些日志，给出 3 个排查步骤和命令。”
- “解释规格和现有代码的差异并给出修复建议。”

## 快速检查清单
- 先读规格和未合并变更。
- 小而可审的补丁；每次改完跑测试。
- 提问要准：先要计划，再要补丁，不要大而泛。
- 评审前重跑 `openspec validate <id> --strict`。
- 在变更目录记录决策与边界情况。
