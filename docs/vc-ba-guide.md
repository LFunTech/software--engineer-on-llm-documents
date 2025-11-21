# 业务分析/BA 手册（Vibe Coding + OpenSpec）

面向业务分析/产品/需求人员，指导如何用 Codex/Claude/Cursor 辅助阅读、澄清、补充 OpenSpec，输出可执行的需求与验收场景。按本文可独立完成从输入到产出。

## 目标
- 快速理解当前规格/变更
- 补充或澄清需求与场景（含开放问题）
- 生成初稿任务列表，供研发/测试执行
- 与研发/测试共用同一来源（OpenSpec）

## 前置
- 阅读 `openspec/AGENTS.md`（格式、禁忌）、`openspec/project.md`（项目规则）
- 查看现状：`openspec list`、`openspec list --specs`
- 工具：建议使用 Claude Code（大文件总结）、Codex CLI（命令/搜索）、Cursor（快速改写）；OpenSpec CLI 速查 `docs/vc-openspec-tool-guide.md`

## 工作流
1) 获取上下文（只读）
   - 命令：`openspec list --specs`，`rg -n "Requirement:|Scenario:" openspec/specs`
   - 提问（Claude/Codex）： “总结与 <domain> 相关的需求与场景，列可能冲突点，不要修改文件。”
2) 识别空白与风险
   - 提问： “列出需求中的歧义、高风险（数据/权限/性能），按优先级排序，给出需确认问题。”
3) 补充/编写需求草稿
   - 按 OpenSpec 格式（ADDED/MODIFIED + `#### Scenario:`）撰写，放在对应 `openspec/changes/<id>/specs/<capability>/spec.md`
   - 提问： “为 <feature> 生成 ADDED 需求和至少两个场景（成功/失败），用 OpenSpec 格式。”
4) 核对前后端影响
   - 提问： “此需求涉及的前端视图/状态、后端接口/数据字段是哪些？列验收要点。”
   - 输出验收清单，方便测试/研发对齐。
5) 任务分解（非实现）
   - 在 `tasks.md` 草拟 5–7 条可验证任务（规格完善、设计、前后端、测试、上线说明）。
6) 回合评审
   - 与研发/测试对齐歧义；确认后再进入实现。

## 模板与示例
- 需求/场景模板（OpenSpec）：
```
## ADDED Requirements
### Requirement: <能力名>
<一句话需求>

#### Scenario: <成功/失败名称>
- **WHEN** ...
- **THEN** ...
```
- 任务模板：
```
## 1. 规格与设计
- [ ] 1.1 补充 <capability> 场景
- [ ] 1.2 澄清 <开放问题列表>
## 2. 实现/测试/运维
- [ ] 2.1 ...
```

## 状态面板示例（与其他文档一致）
- 需求：展示系统状态（ok/degraded/failed），后端提供 `/status`。
- 场景示例：
```
#### Scenario: 状态查询成功
- **WHEN** 展示状态页
- **THEN** 显示 service/build/gitSha/time，状态为 ok/degraded/failed 之一
#### Scenario: 后端错误
- **WHEN** /status 返回错误
- **THEN** 显示错误提示，状态标记为 failed
```
- 验收要点：前端显示正确颜色/文案；后端返回字段完整；错误场景有回退文案；测试覆盖成功+失败。

## 提示语（BA 常用）
- “用 5 句话解释 <feature> 的现有规格，并列出缺失场景。”
- “为 <能力> 补充 3 个失败场景（权限/配置缺失/依赖异常），用 OpenSpec 格式输出。”
- “生成验收清单，按前端/后端/测试分别列 3–5 条。”
- “用通俗语言写一段需求摘要，供评审用，不超过 8 行。”

## 工具选型（BA 用途）
- Claude Code：长文档/多文件总结、生成规格草稿、梳理问题清单。
- Codex CLI：快速搜索规格/任务、查看文件、运行 `openspec validate` 检查格式。
- Cursor：对单个文档/规格行内改写、补充场景。

> 进阶：当规格稳定后，让测试工程师按 `docs/vc-testing-guide.md` 生成自动化；前后端按 `docs/vc-implementation-cookbook.md` 取任务。
