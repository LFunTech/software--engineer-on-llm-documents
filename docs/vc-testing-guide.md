# 测试工程师手册（Vibe Coding）

面向测试/QA，独立文档。按本文即可从环境准备到执行测试，无需阅读其他模块。

## 前提检查（约10分钟）
- 读 `openspec/AGENTS.md`、`openspec/project.md`：了解命名/格式/禁止操作。
- 探索规格：`openspec list`、`openspec list --specs`、`rg -n "Requirement:|Scenario:" openspec/specs`
- 环境：能在仓库根目录运行 `npm test`（或项目等价命令）；安装 Git/Node。

## 快速上手流程（约60–90分钟）
1) 获取上下文
   - 提问： “总结与 <feature> 相关的规格和场景；标出潜在高风险边界。”
2) 推导测试矩阵
   - 提问： “根据这些场景，列出单元/集成/e2e 用例；指出缺失的边界。”
   - 产出：测试用例列表（含成功/失败/边界）。
3) 准备夹具/Mock
   - 提问： “为 <feature> 所需的依赖/网络/配置提供夹具/Mock，优先复用已有助手。”
4) 生成测试（按层次）
   - 单元示例： “为 `StatusBadge` 生成 RTL/Jest 测试：三种状态、空 message 兜底。”
   - 集成示例： “为 `GET /status` 生成 supertest 测试：200 返回数据；配置缺失返回 500。”
   - e2e 示例： “为状态页写 Playwright 测试：加载中、成功、失败三种视图。”
5) 运行与修复
   - 命令示例：`npm test StatusBadge.test.tsx`，`npm test api/status.test.ts`，`npm test`（全量），`npm run lint`（如有）。
   - 提问： “根据测试失败日志，给出三步修复建议。”
6) 报告与交付
   - 记录：运行的命令、结果、未覆盖风险。
   - 输出：测试矩阵、测试文件/路径、执行截图或日志。

## 端到端案例：状态面板
目标：为 `add-project-status-surface` 变更完成测试。

1) 读规格
   - 路径：`openspec/changes/add-project-status-surface/specs/status-surface/spec.md`（假设存在）。
   - 提取场景：`GET /status` 成功/配置缺失；StatusBadge 三状态。
2) 矩阵示例
   - 单测：StatusBadge 渲染文案、颜色；空 message；错误态文案。
   - 集成：`GET /status` 200 payload；配置缺失 500；响应头校验。
   - e2e（可选）：状态页展示正确颜色；后端 500 时展示错误提示。
3) 生成测试
   - 提问： “为上述矩阵生成 Jest/supertest/RTL 测试，使用已有测试工具；返回补丁。”
4) 执行与修复
   - 命令：`npm test api/status.test.ts`，`npm test StatusBadge.test.tsx`，`npm test`
   - 提问： “针对失败用例，给出修改建议和最小补丁。”
5) 交付
   - 提供测试文件路径、命令输出摘要、未覆盖风险与建议。

## 常用提示语
- “列出 <file> 里已有的测试助手/fixtures，并示例如何使用。”
- “把这段失败日志转成三步修复建议。”
- “只输出补丁，不要解释。”

## 自检清单
- 用例覆盖了规格中的成功和主要失败场景
- 夹具/Mock 复用现有工具，无硬编码秘密
- 测试命令可重复，无快排依赖
- 结果与风险有记录，可附在 PR/报告

## 自动化测试场景与流程
以下流程示例使用 Codex CLI（命令行生成/运行补丁）与 Claude/Cursor（大文件推理/补丁预览），可换成你有权限的 AI 工具。两种典型场景：有完整 OpenSpec + 大模型实现（信息充分），以及遗留/需求不全（信息缺失）。

### 场景 A：已有完整 OpenSpec + 大模型实现
目标：从规格直接生成覆盖度高的自动化测试。
1) 读取规格与变更
   - 命令：`openspec list --specs`，`rg -n "Requirement:|Scenario:" openspec/specs`
   - 提问： “提炼 <capability> 的需求与场景，按单元/集成/e2e 分类，列高风险边界。”
2) 生成测试骨架（分层）
   - 提问（Claude/Cursor）： “为这些场景生成 Jest 单测骨架（3–5 条）、supertest 集成测试骨架（2–3 条）、Playwright 骨架（1–2 条）；每条包含描述和 TODO 占位；返回补丁。”
3) 夹具/Mock 对齐实现
   - 命令辅助：`rg -n "fixture|mock|factory" test src`
   - 提问（Codex CLI）： “列出可复用的 fixtures/mocks（含文件路径、导出名），示例如何在骨架中引用。”
4) 填充断言与数据
   - 提问： “按规格中的字段 `{ service, build, gitSha, time }` 补充断言和输入/输出样例；对失败场景添加错误码/信息断言。”
5) 运行与修复
   - 命令示例：`npm test api/status.test.ts`，`npm test StatusBadge.test.tsx`，`npm test`
   - 提问： “根据以下失败日志，生成最小修复补丁（限定单文件、<50 行）。”
6) 交付物
   - 测试文件路径、运行命令与结果、覆盖的规格场景列表、未覆盖风险。

### 场景 B：遗留系统/需求不全
目标：在需求缺失时用测试刻画现状，并逐步规范化。
1) 快速构建“临时场景”
   - 命令探索：`rg -n "TODO|FIXME" src test`，`rg -n "function" src/api | head`
   - 提问： “根据这些文件/接口，推测关键路径与预期行为，输出轻量场景列表（成功/失败），标注不确定项。”
2) 风险优先选择覆盖
   - 优先：数据写入/删除、鉴权/权限、接口协议、钱/计费/安全相关逻辑。
   - 提问： “按风险排序列出 5 条必须覆盖的测试，说明原因。”
3) 生成带假设标注的测试
   - 提问： “基于上述推测场景生成测试补丁，在注释中标明假设与需确认点；缺数据时生成 minimal fixture 示例。”
4) 实测校准
   - 命令：`npm test <scope>` 或项目等价命令。
   - 提问： “实际输出 vs 推测预期的差异，给出 3 条可行动项（更新需求/修复代码/调整测试其一）。”
5) 回写发现与待确认
   - 在测试文件或变更备注中记录：已验证行为、未确认假设、需产品/开发决定的点。
6) 交付物
   - 测试文件、命令与结果、不确定性清单、建议的正式场景补全。

### UI 自动化测试（两种场景通用）
> 如需 Playwright 的详细安装、配置、调试、CI 片段和稳定性技巧，请参考 [docs/vc-playwright-guide.md](vc-playwright-guide.md)。
> OpenSpec CLI 常用命令：[docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)（便于校对场景与变更）。

1) 选择框架
   - 优先复用现有（如 Playwright/Cypress）。命令示例：`npx playwright test`，`npx cypress run`。
2) 设计路径
   - 有规格：按场景列关键路径（加载、交互、错误态）；提问： “为以下场景生成 Playwright 测试步骤（启动、导航、断言 text/aria/截图），限制单文件。”
   - 无规格：先用手工操作或日志推测关键用户路径，记录假设；提问： “基于这些页面行为，推测 3 条关键 e2e 测试并标注假设。”
3) 生成测试
   - 提问： “生成 Playwright 测试补丁，包含选择器策略（prefer role/text），在失败时截图；对缺失数据使用 fixture/seed。”
   - 对遗留系统，在注释标注：依赖的假设、需确认的元素/行为。
4) 稳定性处理
   - 提问： “给这些 Playwright 用例加稳定位等待/断言策略，避免硬编码 sleep。”
   - 使用断言式等待（`await expect(locator).toBeVisible()`），少用 `waitForTimeout`。
5) 运行与产出
   - 命令示例：`npx playwright test tests/status.spec.ts --headed --debug`（调试），`npx playwright test`
   - 交付：测试文件、运行截图/视频、未确认假设清单。

### 提示语模板（自动化专用）
- “依据这些场景，生成 Jest 单测补丁；若缺少数据依赖，写出需要的 fixture 结构和示例。”
- “为 supertest 集成测试生成补丁，包含 200/4xx/5xx 覆盖；复用项目里的 request 工具；限制改动在单文件。”
- “将实际响应与推测的预期对比，列出三条修改建议（代码/测试/需求其一），并生成对应补丁或注释。”
- “为以下界面流程生成 Playwright 测试，使用 role/text 定位，失败时截图，限制单文件。”

## 从手工到自动化：可执行落地方案
面向目前以手工为主的团队，提供 3 个阶段的执行计划和具体命令/提示语。默认选型 Playwright（Node），可替换为团队现有框架。

### 阶段 0：准备（1–2 天）
- 先跑常规命令：`openspec list --specs`、`npm -v` / `node -v` 确认环境。
- 安装依赖（如项目已用 Node）：
  - `npm install -D @playwright/test`
  - `npx playwright install chromium`（或 `--with-deps` 视 CI 环境而定）
- 目录规划：创建 `tests/ui/`（UI 自动化）与 `tests/api/`（集成）。
- 约定选择器：优先 aria/role/text；必要时统一使用 `data-testid`。

### 阶段 1：首批用例（1 周，覆盖 1–2 条关键路径）
- 选用例：从 OpenSpec/变更中挑 1–2 条最重要场景（如登录/状态查询）；遗留系统则选风险最高路径（写操作/鉴权）。
- 生成基础用例（AI 提示）：
  - “在 `tests/ui/status.spec.ts` 生成 Playwright 用例：打开状态页，断言加载态→成功态→错误态，使用 role/text 定位，失败时截图。”
  - “在 `tests/api/status.test.ts` 生成 supertest 用例：200/500 覆盖，复用现有 request 工具。”
- 运行命令：`npx playwright test tests/ui/status.spec.ts`，`npm test tests/api/status.test.ts`（按项目改）。
- 交付：提交测试文件、运行结果截图/trace、已知假设列表。

### 阶段 2：扩展覆盖（2–4 周）
- 目标：每周增加 3–5 条自动化（UI/集成混合），优先高风险路径。
- 固定节奏：每个变更的核心场景→1 个 UI e2e + 1–2 个集成/单元。
- 稳定性：
  - 去除硬等待，改用 `await expect(locator).toBeVisible()` 等断言式等待。
  - 数据隔离：使用 fixtures/seed，测试前后清理。
- 报告：开启 Playwright trace/screenshot，CI 留存产物。

### 阶段 3：CI 接入与守护（并行进行）
- CI 任务顺序：openspec validate → lint → unit/integration → UI e2e 冒烟。
- 提示生成 CI 片段： “为 Playwright+Jest 添加 GitHub Actions job，先跑 `openspec validate <id> --strict`，再 lint、tests，失败即退出；缓存 node_modules。”
- 分层执行：PR 跑冒烟（1–2 条关键 UI + 集成），夜间跑全量。
- 可观测性：失败用例强制上传 trace/screenshot；报告中附变更 ID 与场景名。

### 场景化细节（有规格 vs 无规格）
- 有规格/大模型实现：
  - 每个 `Scenario` 对应至少一个自动化用例；用例描述中引用场景名，便于追溯。
  - 在 PR/变更备注中列“覆盖了哪些 Scenario”。
- 遗留/需求不全：
  - 在用例顶部注释假设与未确认点；实测后将事实回写到注释和待办列表。
  - 每完成一批用例，就把“已确认行为”同步到补充的规格/文档中。

### 可直接执行的命令/片段示例
- 安装 Playwright：`npm install -D @playwright/test`；`npx playwright install chromium`
- 创建示例测试（粘贴到 `tests/ui/status.spec.ts`）：
```ts
import { test, expect } from '@playwright/test';

test('status page shows ok state', async ({ page }) => {
  await page.goto(process.env.APP_BASE_URL || 'http://localhost:3000/status');
  await expect(page.getByRole('heading', { name: /status/i })).toBeVisible();
  await expect(page.getByText(/service:/i)).toBeVisible();
  await expect(page.getByText(/state: ok/i)).toBeVisible();
});
```
- 运行与调试：`npx playwright test tests/ui/status.spec.ts --headed --debug`
- 生成 AI 补丁提示： “为上述测试增加错误态断言，并在失败时保存 screenshot 到 artifacts 目录，返回补丁。”

### 交付检查清单（自动化新增）
- 有规格的用例：列出覆盖的 `Requirement/Scenario`
- 遗留用例：注释假设与待确认点
- 命令/环境：可重复执行的命令、所需 env/config、数据准备说明
- 产物：测试文件、trace/screenshot、运行结果  
> 工具参考：OpenSpec CLI [docs/vc-openspec-tool-guide.md](vc-openspec-tool-guide.md)，终端/IDE/行内助手（Codex/Claude/Cursor）可见对应指南，UI 自动化深潜 [docs/vc-playwright-guide.md](vc-playwright-guide.md)。
