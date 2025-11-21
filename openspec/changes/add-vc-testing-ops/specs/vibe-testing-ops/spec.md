## ADDED Requirements
### Requirement: 分角色测试与运维指南
The repository SHALL provide separate vibe-coding guides for testing engineers and operations/release engineers, each可独立使用，从环境准备到上线/回滚均有步骤、提示语和示例。

#### Scenario: 测试工程师指南可直接使用
- **WHEN** a teammate opens `docs/vc-testing-guide.md`
- **THEN** they find a self-contained manual covering前提检查、从规格推导测试矩阵、夹具/Mock、生成与运行单元/集成/e2e测试的提示语与命令，并附一个端到端案例（状态面板）

#### Scenario: 覆盖 OpenSpec 与遗留系统的自动化测试
- **WHEN** reading the testing guide
- **THEN** it explains如何用 Codex CLI/Claude 等工具生成并执行自动化测试，分别针对“有完整 OpenSpec + 大模型实现”的系统和“遗留/需求不全”系统，给出分步流程与提示语

#### Scenario: 运维/发布工程师指南可直接使用
- **WHEN** a teammate opens `docs/vc-ops-guide.md`
- **THEN** they find a self-contained manual covering前提、CI/CD 接入（含 `openspec validate` 与项目测试）、上线核查、回滚步骤与 runbook 模板，并附同一端到端案例

#### Scenario: UI 自动化测试指导
- **WHEN** reading the testing guide
- **THEN** it provides具体步骤、提示语与命令示例，帮助生成和运行 UI 自动化测试（如 Playwright/Cypress），并区分有完整规格与遗留系统的处理方式

#### Scenario: 手工到自动化的落地方案
- **WHEN** reading the testing guide
- **THEN** it includes一个可执行的渐进计划，指导主要做手工测试的团队从零搭建自动化（含工具选择、安装命令、首条用例生成、CI 接入与里程碑）

#### Scenario: 一人一文档即可交付
- **WHEN** sharing with a role-specific teammate
- **THEN** the corresponding doc is sufficient without其它模块文件, listing所需命令、提示语、产出清单和交付标准
