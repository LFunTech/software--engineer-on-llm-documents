# Claude Code 使用指南（IDE/桌面助手）

适用对象：在 Claude Code（应用或 IDE 插件）中操作仓库的开发/测试/运维。目标：安全阅读、生成多文件 diff，并局部应用。

## 安装与接入
- 安装 Claude Code 应用或 IDE 插件（VS Code 等）。
- 打开仓库根目录，授予文件访问权限。
- 如需登录，使用企业账户并确认模型权限。

## 核心能力
- 读取大文件、多文件上下文，生成 diff 预览。
- 对话式调试：提供日志/错误，获取修复建议与补丁。
- 多文件修改：在 diff 视图中逐块接受/拒绝。

## 操作流程
1) 明确范围
   - 提问：“总结 <feature> 相关文件与风险，不要修改。”
2) 请求变更
   - 提问：“在 <file1> 与 <file2> 增加 <change>，保持现有风格，生成 diff。”
   - 要求“小补丁”“限制行数”，避免过大改动。
3) 审查并应用
   - 在 diff 视图逐块审查，拒绝不需要的部分。
4) 跑命令（若支持终端）
   - 运行 `npm test <scope>`、`openspec validate <id> --strict` 等；如受限，可回到终端执行。

## 提示语模板
- “列出与 <term> 相关的文件和行号，不要改动。”
- “按现有风格在 <file> 添加 <function/component>，限制 80 行，生成 diff。”
- “用 ADDED/MODIFIED 的 OpenSpec 需求为依据，更新 <file>，并生成对应测试。分文件输出。”
- “根据此错误日志，提出 3 步修复并生成补丁。”

## 安全与最佳实践
- 小步应用：一次处理一类改动。
- 避免生成大规模重构；如需重构，先让 Claude 给拆分计划再执行。
- 确认路径：确保修改的是仓库内正确文件。
- 生成代码后仍需本地跑测试/校验。

## 与其他指南的衔接
- 规格/规划：见 `docs/vc-openspec-planning.md`
- 实现与测试流程：`docs/vc-implementation-cookbook.md`、`docs/vc-testing-guide.md`
- Playwright 深入：`docs/vc-playwright-guide.md`（用于 UI 自动化测试）
