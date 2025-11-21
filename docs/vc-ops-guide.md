# 运维/发布工程师手册（Vibe Coding）

面向运维/发布角色的独立文档，从接手变更到上线/回滚的完整流程，无需查阅其他模块。

## 前提检查（约10–15分钟）
- 读 `openspec/AGENTS.md`、`openspec/project.md`：了解格式、禁用命令和项目约定。
- 了解变更：`openspec list` 查看在研变更；阅读对应 `proposal.md`、`tasks.md`。
- 工具：可运行 CI 等价命令（如 `npm test`、`npm run lint`），具备部署权限/账户。
  - OpenSpec CLI 速查：[docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)
  - UI 冒烟（可选）：Playwright 详见 [docs/vc-playwright-guide.md](vc-playwright-guide.md)
- AI 协同心态：让助手先列假设/风险与需要的证据（命令/日志/截图），再给步骤；要求输出可执行命令和回滚口令，而非泛泛流程。

## 快速落地流程（约60分钟）
1) 理解变更与风险
   - 提问： “总结 <change-id> 的变更内容、受影响模块、主要风险（性能/数据/安全）。”
2) 设置 CI 步骤
   - 提问： “生成 GitHub Actions job：先跑 `openspec validate <change-id> --strict`，再 lint，再 tests；失败则终止。”
   - 校验本地/CI 命令：`openspec validate <id> --strict`，`npm run lint`，`npm test`（按项目替换）。
3) 部署前核查
   - 提问： “列出上线前检查清单（配置/密钥/迁移/依赖服务）。如缺项，给具体命令。”
   - AI 协同：要求附命令/查询示例并标注可接受阈值。
4) 部署与冒烟
   - 提问： “上线后需要的冒烟测试列表与命令（含后端 API、前端页面、日志/指标观察点）。”
   - 执行建议命令；记录结果。
   - 如含 UI 自动化冒烟，可调用 Playwright：`npx playwright test tests/ui/status.spec.ts --reporter=line`（详见 [docs/vc-playwright-guide.md](vc-playwright-guide.md)）。
5) 回滚预案
   - 提问： “在出错时的回滚步骤，含命令、数据保护、验证恢复的检查。”
   - AI 协同：要求助手输出触发条件→回滚动作→验证命令的对照表。
6) 交付
   - 输出：CI 变更、上线核查结果、冒烟/指标截图或日志、回滚步骤与触发条件。

## 端到端案例：状态面板
以 `add-project-status-surface` 为例：

1) CI 配置
   - Job 顺序：`openspec validate add-project-status-surface --strict` → `npm run lint` → `npm test`
   - 提示： “生成 Actions 片段，使用 node 18，cache 依赖，失败即退出。”
2) 部署前检查
   - 配置：确认返回的 `service/build/gitSha/time` 数据源存在；环境变量检查命令。
   - 依赖：确保状态端点的依赖服务可用（如版本文件/元数据服务）。
3) 冒烟
   - API：`curl <host>/status`，检查 HTTP 200 与 payload 字段。
   - UI：打开状态页，查看 StatusBadge 颜色与文案；截图保存。
   - 日志/指标：查看错误率/延迟，确认无新增 500。
4) 回滚
   - 提示： “若错误率上升，执行 <回滚命令或关闭 Feature Flag>；回滚后重跑冒烟并检查指标。”
5) 交付材料
   - CI 配置片段、运行结果
   - 上线核查清单与执行情况
   - 冒烟与监控截图或日志
   - 回滚步骤与触发阈值

## 常用提示语
- “给出 deploy 前的 10 项检查清单，按命令列出，并标注可跳过条件。”
- “为 <change-id> 生成回滚脚本/步骤，并附验证恢复的命令。”
- “总结本次上线需要关注的指标和阈值，给出查询命令。”

## Runbook 模板（可直接复制）
```
# Runbook: <feature/incident>

## 触发/症状
- 告警、日志、用户反馈

## 快速排查
- 查看 <dashboard/命令>，检查 Feature Flag/配置状态
- 复现命令：<command here>

## 缓解/回滚
- 回滚命令或关闭 Flag
- 必要时清缓存/重启（注明条件）

## 验证
- 重跑冒烟/故障用例
- 核对指标恢复基线

## 升级/求助
- 值班联系人 / 通讯渠道
```

## 自检清单
- CI 包含 openspec 校验、lint、tests，顺序正确
- 上线前检查执行并记录
- 冒烟/监控覆盖主要风险点
- 回滚步骤清晰、可执行、含验证
