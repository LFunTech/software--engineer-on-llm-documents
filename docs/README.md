# 仓库导航与阅读顺序

本仓库采用 OpenSpec 驱动和 Vibe Coding 助手（Codex CLI、Claude Code、Cursor 等）协作开发。此文档为入口：简要介绍仓库内容，并索引其他指南，方便按角色快速上手。

## 仓库概览
- `openspec/AGENTS.md`：AI 助手的项目约束与操作规则（必读）
- `openspec/project.md`：项目风格、架构与测试策略（需补充项目细节）
- `openspec/changes/`：在研变更（proposal、tasks、delta 规格）
- `openspec/specs/`：已落地的规格（当前可为空）
- `docs/`：教程与角色手册（见下方导航）

## 快速开始（通用）
1) 进入仓库根目录，运行：
   - `openspec list`，`openspec list --specs`（查看变更/规格现状）
   - `rg --files | head`（确认可读文件）
2) 阅读规则：`openspec/AGENTS.md`、`openspec/project.md`
3) 根据角色选文档（见下文导航），按步骤练习或执行
4) 任何变更前先跑 `openspec validate <change-id> --strict`（如在改 specs/changes）

## 阅读顺序与导航
1) 必读规则：`openspec/AGENTS.md`、`openspec/project.md`
2) 角色/任务选择（至少选其一）：
   - 新人/通用上手：`docs/vc-foundations.md`
   - Vibe Coding 总览 + 案例：`docs/vibe-coding-tutorial.md`
   - 规划/规格作者：`docs/vc-openspec-planning.md`
   - 业务分析/BA：`docs/vc-ba-guide.md`
   - 数据分析师：`docs/vc-data-analyst-guide.md`
   - 大数据工程师：`docs/vc-bigdata-guide.md`
   - 前端/后端实现：`docs/vc-implementation-cookbook.md`
   - 测试工程师：`docs/vc-testing-guide.md`
   - 运维/发布工程师：`docs/vc-ops-guide.md`
3) 工具深入（按需）：
   - Codex 终端：`docs/vc-codex-guide.md`
   - Claude Code：`docs/vc-claude-guide.md`
   - Cursor 行内助手：`docs/vc-cursor-guide.md`
   - OpenSpec CLI：`docs/vc-openspec-tool-guide.md`
   - Playwright UI 自动化：`docs/vc-playwright-guide.md`

## 常用命令速查
- 查看变更/规格：`openspec list`，`openspec list --specs`
- 严格校验：`openspec validate <change-id> --strict`
- 搜索规格文本：`rg -n "Requirement:|Scenario:" openspec`
- 测试/构建命令：按项目实际（如 `npm test`、`npm run lint`）补充到各角色文档
