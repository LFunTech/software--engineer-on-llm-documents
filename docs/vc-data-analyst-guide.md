# 数据分析师手册（Vibe Coding + OpenSpec）

面向数据分析/BI 人员，指导如何在 OpenSpec 框架下用 Codex/Claude/Cursor 辅助需求澄清、数据理解、查询/脚本生成、指标验收和可视化交付。

## 前置
- 读 `openspec/AGENTS.md`、`openspec/project.md`，了解规格格式与项目约定。
- 查看变更/规格：`openspec list`，`openspec list --specs`；必要时用 `rg -n "Requirement:|Scenario:" openspec/specs` 查场景。
- 工具：推荐 Claude Code（长文档/多文件总结）、Codex CLI（查询/补丁/脚本）、Cursor（行内改写）。OpenSpec CLI 速查 [docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)。

## 工作流（从需求到交付）
1) 澄清需求与数据来源
   - 提问： “总结 <feature/metric> 相关的规格/变更，列数据来源和维度/指标定义的空白点。”
   - 输出：需确认的业务口径、缺失字段、时间/粒度要求。
2) 数据理解与样本抽取
   - 命令：用 `rg <table/field>` 查代码/模型；必要时运行安全的只读查询（按项目规范）。
   - 提问： “为表 <table> 生成数据概览/字段含义假设，列可能的值域和异常值检查。”
3) 设计指标与验收
   - 在 OpenSpec delta 中增加/更新指标要求与场景（成功/异常），放入 `openspec/changes/<id>/specs/<capability>/spec.md`。
   - 提问： “为指标 <metric> 写 ADDED 需求与场景（含异常检测）。”
4) 生成查询/脚本
   - 提问： “生成计算 <metric> 的 SQL（含过滤/分组/时间窗），并给出示例输入输出；限制为只读查询。”
   - 如需数据校验，提示： “生成数据质量检查（空、重复、越界）SQL/脚本。”
5) 验证与可视化草稿
   - 运行查询（遵守项目规则），获得样本结果。
   - 提问： “根据结果，生成可视化方案（图表类型、维度/度量），输出仪表盘草稿描述。”
6) 交付
   - 更新 `tasks.md`：列出查询/验证/可视化任务。
   - 在变更备注中附查询、样本结果、已知数据风险。

## OpenSpec 模板示例（指标）
```
## ADDED Requirements
### Requirement: 指标 - 每日活跃用户 (DAU)
定义 DAU 为每日去重用户数，去重键为 user_id，时区 UTC。

#### Scenario: 计算成功
- **WHEN** 按日聚合用户访问日志
- **THEN** 返回去重 user_id 的计数及日期

#### Scenario: 数据缺失
- **WHEN** 某日无日志
- **THEN** 返回 0 并标记数据缺失
```

## 常用提示语
- “总结 <metric> 相关的字段和表，列缺失信息与需确认的业务口径。”
- “生成计算 <metric> 的 SQL，并添加空值/重复值检查。”
- “根据这段规格，补充 2 个失败场景（数据缺失、越界）。用 OpenSpec 格式输出。”
- “为仪表盘生成 3 个图表建议（类型、维度、过滤），说明适用场景。”

## 数据质量与测试
- 轻量校验：空值、越界、重复、时间跨度缺口；用 SQL/脚本生成检查。
- 对接测试工程师：将质量检查纳入自动化（可参考 [docs/vc-testing-guide.md](vc-testing-guide.md)）。

## 交付清单
- 规格/场景（OpenSpec）
- 查询/脚本 + 示例结果
- 可视化草稿或描述
- 数据风险/假设清单
