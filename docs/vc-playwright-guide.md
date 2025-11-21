# Playwright 使用说明（从入门到进阶）

面向初次使用 Playwright 的测试/研发人员，按本稿可完成安装、首个用例、稳健定位、数据/夹具、调试、CI 接入与常见优化。默认 Node 环境，可结合 Codex CLI/Claude/Cursor 生成补丁。

## 环境与安装
- 前置：Node 18+、npm、可运行 `npx`.
- 安装依赖：
  - `npm install -D @playwright/test`
  - `npx playwright install chromium`（需要多浏览器可去掉 `chromium`）
- 推荐目录：`tests/ui/` 存放 e2e 用例；`playwright.config.ts` 放项目根部。

## 快速入门：首个用例
1) 创建用例 `tests/ui/example.spec.ts`：
```ts
import { test, expect } from '@playwright/test';

test('status page shows ok state', async ({ page }) => {
  await page.goto(process.env.APP_BASE_URL || 'http://localhost:3000/status');
  await expect(page.getByRole('heading', { name: /status/i })).toBeVisible();
  await expect(page.getByText(/service:/i)).toBeVisible();
  await expect(page.getByText(/state: ok/i)).toBeVisible();
});
```
2) 运行：
   - 调试模式：`npx playwright test tests/ui/example.spec.ts --headed --debug`
   - 常规：`npx playwright test tests/ui/example.spec.ts`

## 选择器与等待策略
- 优先使用语义定位：`getByRole`、`getByText`、`getByLabel`。无语义时添加 `data-testid`。
- 避免硬等待：用断言式等待 `await expect(locator).toBeVisible()`、`toHaveText`、`toHaveAttribute`。
- 网络/状态等待：`await page.waitForLoadState('networkidle')` 仅在必要时使用。
- 不建议：`page.waitForTimeout`, 使用会降低稳定性。

## 夹具与数据准备
- 配置 `playwright.config.ts` 中的 `use.baseURL`、`use.storageState`（如需登录态）。
- 为测试数据写 seed/fixture 脚本，在 test 前调用；或使用 `test.beforeEach` 清理/准备数据。
- 记录环境变量需求（例如 `APP_BASE_URL`、`API_TOKEN`），在 CI 中注入。

## 常用配置示例（playwright.config.ts）
```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: 'tests/ui',
  use: {
    baseURL: process.env.APP_BASE_URL || 'http://localhost:3000',
    headless: true,
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  reporter: [['line'], ['html', { outputFolder: 'playwright-report' }]],
});
```

## 运行与过滤
- 指定用例/文件：`npx playwright test tests/ui/status.spec.ts`
- 按标题过滤：`npx playwright test -g "status page"`
- 并行/分片：`npx playwright test --workers=4`，CI 可按文件/标签分组。

## 调试技巧
- `--debug` 进入 Inspector，逐步执行。
- 查看 trace：`npx playwright show-trace trace.zip`
- HTML 报告：`npx playwright show-report playwright-report`
- 截图/视频存档：配置中开启；定位失败可直接查看截图/视频。

## Mock 与网络拦截
- 使用 `page.route` 拦截特定接口，提供伪数据以稳定 UI：
```ts
await page.route('**/status', route =>
  route.fulfill({ status: 200, contentType: 'application/json', body: JSON.stringify(mockData) })
);
```
- 对关键写操作谨慎 Mock，尽量用沙箱环境。

## CI 集成示例（GitHub Actions）
```yaml
jobs:
  ui-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '18' }
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm test -- --runInBand --selectProjects api || true  # 如有后端测试
      - run: npx playwright install --with-deps
      - run: npx playwright test
    artifacts:
      paths:
        - playwright-report/**
        - test-results/**
```
- 在 CI 中开启 trace/screenshot 便于复盘。
- PR 流程跑冒烟集；夜间可跑全量。

## 稳定性与最佳实践
- 数据隔离：测试前后清理或使用独立账户/租户。
- 失败重试：对易抖用例可开启 `retries: 1`，避免全局大重试掩护问题。
- 页面对象适度：为稳定元素封装 getter/动作，避免过度抽象导致调试困难。
- 将 `Requirement/Scenario` 关联记录在测试描述/注释中，便于追溯。

## 有规格 vs 遗留系统的 UI 测试
- 有规格：按场景生成用例，标题中含场景名；缺少场景先更新规格再写用例。
- 遗留：先手工梳理关键路径→生成用例并在注释标注假设→实测校准，将确认结果回写到规格/文档。

## 典型提示语（可复制）
- “在 `tests/ui/status.spec.ts` 生成 Playwright 用例，覆盖加载/成功/失败三态，使用 role/text 定位，失败时截图并保留 trace，返回补丁。”
- “为这个用例补充网络拦截，让 `/status` 返回 mock 数据，加入断言确保使用 mock 数据。”
- “优化这些用例的等待方式，移除硬等待，改为断言式等待，返回补丁。”

## 进一步阅读
- 官方文档：https://playwright.dev/docs/intro
- 断言指南：https://playwright.dev/docs/test-assertions
- Trace/报告：https://playwright.dev/docs/trace-viewer
