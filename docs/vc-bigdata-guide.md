# 大数据工程师手册（Vibe Coding + OpenSpec）

面向大数据/数据平台工程师，指导在 OpenSpec 下用 Codex/Claude/Cursor 辅助管道设计、数据模型、质量/回填、性能与监控。

## 前置
- 读 `openspec/AGENTS.md`、`openspec/project.md`。
- 查看变更/规格：`openspec list`，`openspec list --specs`。
- 工具：Codex CLI（终端补丁/命令）、Claude Code（多文件 diff 规划）、Cursor（行内修改）；OpenSpec CLI 速查 [docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)。

## 工作流（示例）
1) 理解需求与数据契约
   - 提问： “总结 <pipeline/feature> 需求，列输入/输出表、SLA、数据质量要求，标存在的空白。”
   - 在 OpenSpec delta 中补充/更新数据契约、SLA、质量场景。
2) 管道/模型设计
   - 提问： “为 <source→sink> 生成分层设计（staging/cleaned/serving），列字段映射、去重/校准策略。”
   - 输出初稿到 `design.md`（如需要），并在 specs 中添加 ADDED/MODIFIED 场景。
3) 数据质量与监控
   - 提问： “列出需要的质量检查（空/重复/越界/延迟），生成对应 SQL/代码片段。”
   - 为延迟/丢失添加场景，便于测试与告警。
4) 回填/修复计划
   - 提问： “生成回填步骤（分批/幂等/对账），含命令和验证点。”
   - 在 tasks 中加入“回填”和“对账”子任务。
5) 性能与成本
   - 提问： “现有查询/作业的瓶颈在哪里（I/O/倾斜/Shuffle），给出 3 条优化建议，并生成最小改动补丁。”
6) 部署与验证
   - 对应运行命令（依赖项目栈：如 `spark-submit`、`flink run`、`dbt run/test` 等）；记录结果。
   - 复用测试指南：将质量检查纳入自动化/CI。

## OpenSpec 场景示例（管道）
```
## ADDED Requirements
### Requirement: 事件流水清洗管道
清洗原始事件流，去重、校准时区，输出到 cleaned.events。

#### Scenario: 去重与时区处理
- **WHEN** 相同 event_id 多次到达且 timestamp 为 UTC
- **THEN** 输出仅一条记录，并将时间转换为项目时区

#### Scenario: 延迟与丢失监测
- **WHEN** 5 分钟内未收到事件
- **THEN** 触发数据延迟告警并记录缺口窗口
```

## 常用提示语
- “为 <source table> → <target table> 生成字段映射和校验策略（去重、空值、越界），输出设计草稿。”
- “为 <pipeline> 编写质量检查 SQL/代码，覆盖空、重复、越界、延迟，限制在单文件补丁。”
- “生成回填计划：分批范围、并发、幂等方案、对账步骤，输出命令和验证点。”
- “分析这段 Spark/Flink 查询可能的性能瓶颈，给出 3 条最小改动建议，生成补丁。”

## 交付清单
- 规格/场景（OpenSpec）覆盖契约、质量、SLA、回填
- 设计草稿（可选）：分层/字段映射/依赖
- 质量检查脚本/配置，性能优化建议
- 回填/修复计划及验证步骤
- 运行结果与未决风险
