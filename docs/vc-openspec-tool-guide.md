# OpenSpec CLI 使用指南（规格与变更管理）

面向需要编写/检查 OpenSpec 的同事（BA、开发、测试、运维）。按本文可从零完成变更脚手架、写规格、校验，并结合 AI 工具（Codex/Claude/Cursor）加速。

## 核心命令速览
- 列变更：`openspec list`
- 列规格：`openspec list --specs`
- 查看详情：`openspec show <item>`（变更或规格），`openspec show <change> --json --deltas-only`
- 校验：`openspec validate <change-id> --strict`
- 搜索场景：`rg -n "Requirement:|Scenario:" openspec`

## 从想法到可验证变更（示例流程）
1) 取 change-id/capability（动词-kebab）
   - 例：`add-status-surface`，capability `status-surface`
2) 脚手架
   - `mkdir -p openspec/changes/add-status-surface/specs/status-surface/`
   - 创建 `proposal.md`、`tasks.md`、`specs/status-surface/spec.md`
3) 写 proposal.md
   - 模板：`# Change` / `## Why` / `## What Changes` / `## Impact`
4) 写 delta（spec.md）
   - 用 `## ADDED|MODIFIED|REMOVED Requirements`，每条 `#### Scenario:` 至少一条
   - 修改现有需求时复制原文到 MODIFIED 再改
5) 写 tasks.md
   - 5–7 条可验证任务（规格/实现/测试/上线）
6) 校验
   - `openspec validate add-status-surface --strict`
   - 失败时运行 `openspec show add-status-surface --json --deltas-only` 定位问题

## 文件结构速查
```
openspec/
  AGENTS.md           # 规则/禁忌（必读）
  project.md          # 项目约定（补充细节）
  changes/<id>/       # 在研变更（proposal/tasks/spec deltas）
  specs/<capability>/ # 已落地规格
```

## AI 辅助写作（提示语例）
- 摘要与冲突检查：  
  “总结 <area> 相关规格/变更，列潜在冲突或重复点，不要改文件。”
- 生成 proposal 草稿：  
  “按 openspec/AGENTS.md 模板，为 <feature> 写 proposal.md 要点（Why/What/Impact）。”
- 生成 ADDED 需求：  
  “为 `<capability>` 写 ADDED 需求，至少 2 个场景（成功/失败），用 OpenSpec 格式。”
- 修改需求：  
  “复制并改写 `specs/<capability>/spec.md` 中的 <Requirement>，加入 <new behavior>，保持 MODIFIED 格式。”
- 纠正校验错误：  
  “根据以下 validate 报错，生成修复步骤和补丁。”

## 常见错误与排查
- “Change must have at least one delta”：确认 `changes/<id>/specs/.../spec.md` 存在且含 ADDED/MODIFIED。
- “Requirement must have at least one scenario”：检查 `#### Scenario:` 是否四个 `#`，至少一条。
- MODIFIED 丢内容：始终拷贝原需求全文再改，避免部分替换。
- 空格/标题：`## ADDED Requirements` 匹配精确文本（前后空格可忽略），场景标题用冒号。

## 与角色指南的衔接
- BA：写/改需求、补场景 → [docs/vc-ba-guide.md](vc-ba-guide.md)
- 规划作者：脚手架与流程 → [docs/vc-openspec-planning.md](vc-openspec-planning.md)
- 实现/测试/运维：在执行前用 `openspec validate <id> --strict` 确认规格格式正确

## 进阶技巧
- JSON 输出：`openspec show <change> --json --deltas-only | jq '.'` 便于脚本检查。
- 批量校验：`openspec validate --strict`（全量）。
- 模板复用：参考 `openspec/changes/archive`（如有）复制结构/措辞，避免重复发明。
