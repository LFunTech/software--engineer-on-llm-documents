# Codex CLI 使用指南（终端助手）

适用对象：在终端使用 Codex CLI（本环境即此）的开发/测试/运维。目标：快捷检索、写补丁、跑命令/测试，并保持安全。

## 环境与基础操作
- 你已经在 Codex CLI 环境中：可直接运行 shell 命令，使用 `apply_patch` 编辑。
- 只读探索：`ls`、`rg --files`、`rg -n "<keyword>" <path>`、`cat <file>`.
- 安全注意：禁止 `git reset --hard`、`rm -rf` 等破坏性命令；先读文件再改。

## 常用工作流
### 1) 先看计划
- 提问模式：让助手先给 3–5 步计划，再逐步执行。
- 示例：“为 <feature> 给出 3 步计划，列要看的文件，不要修改。”

### 2) 定位与阅读
- 搜索：`rg -n "Requirement:|Scenario:" openspec/specs`（规格）；`rg -n "<term>" src`.
- 查看文件：`sed -n '1,120p' <file>`；`cat <file>`。

### 3) 生成/应用补丁
- 提问：“在 <file> 中添加 <change>，保持现有风格，返回补丁。”
- 应用：将补丁内容传给 `apply_patch`。修改前确认上下文。
- 小步提交：一次改一件事，便于回溯。

### 4) 跑测试/校验
- OpenSpec 校验：`openspec validate <change-id> --strict`
- 项目测试：`npm test <scope>`、`npm run lint`（按项目替换）。
- 失败后：提问“根据以下日志给出三步修复建议”。

## 提示语模板
- “给出 3 步计划实现 <feature>，不修改文件。”
- “在 <file> 增加函数/组件 <name> 的补丁，限制 50 行内。”
- “列出和 <term> 相关的文件与行号，勿修改。”
- “根据这段测试失败日志，写最小修复补丁（单文件，<50 行）。”

## 与 OpenSpec 协同
- 查看变更/规格：`openspec list`，`openspec list --specs`
- 起草/验证变更：见 [docs/vc-openspec-planning.md](vc-openspec-planning.md)；校验用 `openspec validate --strict`

## 调试与故障
- 命令失败：检查路径/权限；必要时重跑并附错误日志。
- 补丁失败：确认上下文未变；先 `sed -n 'start,endp' <file>` 复核，再生成补丁。

## 进阶
- 批量替换：谨慎使用 `perl -pi -e`/`python -c`，先备份或小范围试跑。
- 日志保留：描述运行过的命令与文件修改，便于评审。
