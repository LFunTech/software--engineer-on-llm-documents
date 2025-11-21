# Vibe Coding 基础（第一天上手）

本模块帮助新人在第一天内用 Codex CLI、Claude Code、Cursor 等 AI 助手完成安装、配置、访问验证和安全练习。

## 目标
- 完成工具安装与仓库访问确认
- 知道各工具的适用场景
- 养成安全提问与执行习惯
- 通过低风险练习建立信心

## 角色
- 新人（第 0–2 天）：需要端到端上手与安全的首次修改
- 现有开发：想用 AI 提速导航与小型改动
- 培训者：组织工作坊、准备安全默认设置

## 工具分工
- Codex CLI：命令行优先，擅长搜索、补丁、跑测试；适合精确修改和命令留痕。
- Claude Code：IDE/桌面 AI，擅长大文件理解和多文件 diff 预览。
- Cursor：VS Code 兼容，支持行内编辑与快速重构。
- AI 协同心态（新人必读）：先讲清意图/范围/禁区，再要答案；要求助手列假设/缺口、返回补丁或命令（带文件路径），小步应用并自检。

## 安装/配置清单
1) 账户
   - 登录各 AI 提供商账号。
   - 若有企业/组织限制，确认已获准使用。
2) 本地环境
   - 安装 Git 和项目所需运行时（如 Node）。
   - 终端可进入仓库目录。
3) 仓库访问
   - Clone/更新：`git clone <repo>` 或 `git pull`。
   - 在仓库根目录试运行：`ls`，`rg --files | head`（只读验证）。
4) Codex CLI
   - 运行 `openspec list` 验证 CLI 可用。
5) Claude Code
   - 安装应用/插件；打开仓库；如需授权，允许读取文件。
6) Cursor
   - 安装 Cursor；打开仓库；开启 “Ask Cursor” 与行内编辑。

## 90 分钟上手路线（按顺序做）
1) 10 分钟：仓库与规则
   - 读 `openspec/AGENTS.md`，记住命名/格式和禁止操作。
   - 读 `openspec/project.md` 了解代码风格/架构/测试策略。
   - AI 协同：提问时要求“只读总结，不要改文件”，并让助手列出未解之处。
2) 15 分钟：只读探索
   - 命令：`openspec list`、`openspec list --specs`、`rg -n "Requirement:|Scenario:" openspec`
   - 提问（Codex/Claude）： “总结本仓库的目录与入口文件，不要修改。”
   - AI 协同：要求引用路径，先列假设/疑问再给结论。
3) 20 分钟：小补丁演练
   - 选择一份 Markdown 文件。
   - 提问： “给出错别字/措辞改进的最小补丁，只返回 diff。”
   - 用 `apply_patch` 或 Cursor 行内应用；检查 diff。
   - AI 协同：限制“只改这个文件，这几行”；让助手解释意图和风险。
4) 20 分钟：简单代码改动
   - 选择一处日志或文案改动。
   - 提问： “在 <file> 增加启动时间日志，保持现有风格；给出补丁和测试建议。”
   - 运行相关测试或 `npm test <scope>`。
5) 25 分钟：总结与自检
   - 提问： “我们改了什么？风险点有哪些？应跑哪些测试？” 记录在笔记/PR 草稿中。

> 具体工具操作可参考：Codex 终端助手 [docs/vc-codex-guide.md](vc-codex-guide.md)，Claude Code [docs/vc-claude-guide.md](vc-claude-guide.md)，Cursor 行内助手 [docs/vc-cursor-guide.md](vc-cursor-guide.md)，UI 自动化 [docs/vc-playwright-guide.md](vc-playwright-guide.md)。

## 安全与卫生
- 先用只读命令（`ls`、`rg`、`cat`、`git status -sb`）。
- 非必要不做破坏性操作：避免 `git reset --hard`、`rm -rf`、批量删除。
- 提问要聚焦：先要计划，再要具体 diff。
- AI 协同：每次请求补丁前先限定范围/文件；要求助手提供验证或回滚提示（如相关测试命令、检查点）。
- 应用前审查 diff：留意范围蔓延、泄露机密、配置漂移。
- 有实质改动就跑测试/linters。
- 编码前先读 `openspec/AGENTS.md`、`openspec/project.md` 了解规则。

### 安全/不安全提示语示例
- 安全： “按现有模式增加 `GET /status` 的 3 步计划，不要编辑文件。”
- 安全： “指出 StatusBadge 定义和相关测试的位置。”
- 不安全： “为性能重构整个项目”（范围过大，风险高）。
- 不安全： “删除未使用的文件”（破坏性、高歧义）。

## 第一天练习
1) 浏览与搜索
   - 提问： “列出本仓库的关键目录与入口文件，不要改动。”
   - 运行 `rg -n "Requirement:|Scenario:" openspec` 查看规格与场景。
2) 小型阅读/编辑
   - 选一个 Markdown 文件；提问： “给出精简的错别字修复，只展示补丁。”
   - 用 CLI `apply_patch` 或 Cursor 行内编辑应用，并审查 diff。
3) 简单代码提示
   - 提问： “在 <file> 里增加打印启动时间的 console.log，保持现有风格。”
   - 如有测试/linters，运行 `npm test -- --watch=false` 等命令。
4) 复盘
   - 提问： “我们改了什么？风险在哪里？” 练习自检。

## 完整入门练习（贯穿案例）
使用“项目状态面板”作为练习（与主教程一致）：
- 只读：定位规格位置，确认是否已有状态相关条目。
- 规划：创建 change 脚手架（见规划模块），不要写代码。
- 实现：按实现模块指引，先做后端再做前端，每次小补丁。
- 测试/运维：按测试与运维模块执行最小测试与 CI 片段。

## 常见问题
- 工具无法读文件：重新打开仓库目录并授权。
- 网络受限：仅用本地上下文，避免需要联网的提示。
- 出现幻想内容：要求给出文件引用，让提示更具体。

## 进阶判断
- 能描述仓库结构。
- 能安全请求并应用小补丁。
- 能运行基础验证（测试或 `openspec validate <id> --strict`）。
