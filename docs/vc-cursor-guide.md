# Cursor 使用指南（IDE 内联助手）

适用对象：使用 Cursor（VS Code 分支）进行行内修改、重构、测试生成的开发/测试人员。

## 安装与接入
- 安装 Cursor，使用账户登录。
- 打开仓库根目录，允许读取文件。
- 启用 “Ask Cursor”/行内编辑功能。

## 核心用法
- 行内选区改写：选中代码 → “Ask Cursor” → 描述期望 → 逐块接受/拒绝。
- 聊天面板：提供文件/目录上下文，获取计划、补丁或解释。
- 多文件建议：Cursor 可引用相关文件生成改动，但建议一次关注单个文件/功能。

## 工作流
1) 先要计划
   - 提问：“为 <feature> 给出 3–5 步计划，列要改的文件，不要写代码。”
2) 小步修改
   - 选中函数/组件，提问：“将此改为 <desired change>，保持风格，限制 50 行。”
   - 检查每个 hunk，拒绝超范围改动。
3) 生成测试
   - 提问：“为该组件生成 RTL/Jest 测试，放在 <file>.test.tsx，限制 80 行。”
4) 修复错误
   - 将错误日志贴入聊天，提问：“给出三步修复并生成补丁，限单文件。”

## 提示语模板
- “将选中代码改为使用 <library>，保持类型/风格，限制 30 行。”
- “为选中组件生成单测，覆盖成功/错误路径，使用现有测试工具。”
- “列出与 <term> 相关的文件和测试，勿修改。”
- “按 OpenSpec 场景 <scenario> 更新此函数的行为，给出补丁。”

## 最佳实践
- 一次改一处，减少上下文冲突。
- 保持与项目工具链一致（lint/formatter）；改完后在终端跑测试。
- 对大改动先在聊天获取方案，再分块应用。

## 与其他指南的衔接
- 规划与规格：[docs/vc-openspec-planning.md](vc-openspec-planning.md)
- 实现流程：[docs/vc-implementation-cookbook.md](vc-implementation-cookbook.md)
- 测试/自动化：[docs/vc-testing-guide.md](vc-testing-guide.md)，UI 自动化详见 [docs/vc-playwright-guide.md](vc-playwright-guide.md)
