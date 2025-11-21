# 用 Vibe Coding 工具进行 OpenSpec 规划

本指南展示如何在 AI 辅助下编写 OpenSpec 的 proposal、设计和 delta，并保持格式可验证。

## 预期收益
- 知道何时做 proposal，何时直接修复
- 能脚手架 `openspec/changes/<id>/` 所需文件
- 会用提示语编写 proposal 与规格 delta
- 能在严格模式下验证并迭代

## 快速规则
- 先读 `openspec/AGENTS.md` 和 `openspec/project.md`。
- `change-id` 用动词开头的 kebab case（例：`add-status-endpoint`）。
- 每条需求在 `## ADDED|MODIFIED|REMOVED Requirements` 下必须有至少一个 `#### Scenario:`。
- 始终用严格模式验证：`openspec validate <id> --strict`。

## 工作流
1) 查看当前状态
   - 命令：`openspec list`，`openspec list --specs`，`rg -n "Requirement:|Scenario:" openspec/specs`。
2) 选择变更范围
   - 新功能/破坏性/架构变更 → 写 proposal。
   - Bug/错别字/恢复预期行为 → 直接改。
3) 脚手架
   - `mkdir -p openspec/changes/<id>/specs/<capability>/`
   - 创建 `proposal.md`、`tasks.md`，必要时 `design.md`。
   - 在 `specs/<capability>/` 下创建 delta `spec.md`。
4) 写 proposal
   - 结构：`# Change`，`## Why`，`## What Changes`，`## Impact`。
5) 写规格 delta
   - 使用 `## ADDED|MODIFIED|REMOVED Requirements`。
   - 若用 MODIFIED，请复制原需求全文再改。
6) 早验证
   - 运行 `openspec validate <id> --strict`，修复格式/场景标题。
   - 若需要调试解析，运行 `openspec show <id> --json --deltas-only | jq '.'` 查看 delta 结构。

## 提示语模板
- 总结上下文： “总结 <area> 相关规格并列潜在冲突，不要改文件。”
- 生成 change id： “为新增 <feature> 提 3 个动词开头 change ID 和 capability 名。”
- 草拟 proposal： “按 openspec/AGENTS.md 模板，为 <feature> 写 proposal.md 要点。”
- 新增 delta： “为 `<capability>` 写 `## ADDED Requirements`，每条需求至少一个场景。”
- 修改 delta： “重写 `specs/<capability>/spec.md` 中的现有需求以包含 <new behavior>，返回完整需求块。”
- 歧义检查： “列出本 proposal 不清晰的行为或边界，建议要补的场景。”
- 预测校验错误： “根据这个 delta，预测 `openspec validate --strict` 的常见报错。”

## 示例：从想法到通过校验
目标：增加 `GET /status` 端点。

1) 取 change id 与 capability
   - 提示： “为状态端点建议 change ID 和 capability 名。” → 选 `add-status-endpoint`，capability `status-api`。
2) 脚手架
   - `mkdir -p openspec/changes/add-status-endpoint/specs/status-api/`
   - 创建 `proposal.md`、`tasks.md`。
3) 草拟 proposal（AI 辅助）
   - 提示： “为 add-status-endpoint 写 proposal.md：返回服务状态和构建信息，提及 CI 影响。”
4) 草拟规格 delta
   - 提示： “为 GET /status 写 ADDED 需求：返回 `{ service, build, gitSha, time }`，包含成功/失败场景，使用正确的场景标题。”
   - 保存到 `openspec/changes/add-status-endpoint/specs/status-api/spec.md`。
5) 列任务
   - 提示： “列出 4–6 个任务实现 GET /status，含测试和 CI 更新。”
6) 校验
   - 运行：`openspec validate add-status-endpoint --strict`
   - 调试（如有报错）：`openspec show add-status-endpoint --json --deltas-only | jq '.'`
   - 若报错，提问： “把这些校验错误翻译成需要的编辑。”
7) 提审
   - 说明 change id 与 capability，附上验证结果。

## 分角色操作流（沿用状态端点案例）
- 新人/作者
  - 写满 proposal 的 Why/What/Impact，确保列出受影响的 spec/代码。
  - 用 ADDED/MODIFIED 清晰区分新增与改动，避免混写。
  - 在 tasks 中拆成 5–7 个可验证小任务（路由、校验、日志、测试、CI）。
- 评审/负责人
  - 检查 delta 是否覆盖成功+失败场景；若缺边界，要求补场景。
  - 确认 tasks 顺序合理（先规格后实现），避免一次覆盖过多模块。
  - 要求在实现前先通过 `openspec validate --strict`。
- QA/测试
  - 从场景中提取测试矩阵；提前标记高风险（配置缺失、网络错误）。
  - 在 tasks 中加入测试生成与执行步骤。

## 情境提示语（可直接用）
- 获取现状： “列出当前 `specs` 下与 status 相关的需求及场景。”
- 补充边界： “为 GET /status 增加 3 个失败场景（配置缺失、依赖超时、权限不足），用 ADDED/MODIFIED 给出。”
- 任务拆分： “把实现拆成 6 个任务，标明顺序和可验证方式，每项不超过两行。”
- 校验前检查： “根据当前变更内容，预测 `openspec validate <id> --strict` 可能的报错，并示范修复。”

## 处理歧义
- 让 AI 列举边界（超时、鉴权、限流）。
- 每个边界补充场景，让行为明确。
- 不确定时把问题记录到 `design.md` 或 `proposal.md`。

## 快速迭代技巧
- delta 要小；关注点分离时拆成不同 capability。
- 有归档示例时复用结构与措辞。
- 每个需求至少一个场景，避免校验失败。

> 工具配合：终端驱动请见 `docs/vc-codex-guide.md`；OpenSpec CLI 速查 `docs/vc-openspec-tool-guide.md`；IDE/多文件 diff 使用 `docs/vc-claude-guide.md`；行内修改参考 `docs/vc-cursor-guide.md`。
