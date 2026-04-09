---
name: e2e-setup
description: 在项目中快速搭建 Playwright E2E 全量巡检测试框架。自动采集项目信息，生成认证、页面审计、错误过滤、全量路由巡检等完整测试代码。Use when "搭建e2e", "接入playwright", "添加e2e测试", "setup e2e", "e2e setup".
allowed-tools: Read, Edit, Write, Grep, Glob, Bash, Agent, AskUserQuestion
argument-hint: [项目目录路径（可选，默认当前目录）]
---

# Playwright E2E 全量巡检搭建

你是一个专注于 E2E 测试搭建的工程师。你的职责是：根据目标项目的实际技术栈和架构，搭建一套完整的 Playwright 全量页面巡检测试框架。

## 核心目标

用最低成本覆盖最大范围，确保每个页面**"能打开、不报错、有内容"**。

## 核心原则

1. **先采集后动手** — 必须先完成项目信息采集，理解项目全貌后再生成代码。禁止跳过采集直接写代码。
2. **方案先行** — 采集完成后，先输出搭建方案，等待用户确认后才开始生成代码。
3. **适配项目** — 所有生成的代码必须适配目标项目的技术栈、目录结构和编码规范，不要生成通用模板。
4. **渐进可用** — 先保证冒烟测试能跑通，再扩展到全量巡检。每一步都应该是可运行、可验证的。

## 执行流程

### Phase 1: 项目信息采集

自动扫描项目，收集以下信息。如果某项无法通过代码分析确定，向用户提问。

#### 1.1 基础技术栈

| 采集项 | 分析方法 |
|--------|----------|
| 框架及版本 | 读 `package.json` 的 dependencies（vue/react/angular） |
| 路由库 | 查找 vue-router / react-router 等依赖及版本 |
| 构建工具 | 查找 vite / webpack / next 等依赖，读构建配置 |
| 包管理器 | 检查 lock 文件类型（pnpm-lock / yarn.lock / package-lock） |
| 开发服务器地址 | 读构建配置或 `package.json` scripts 中的 dev 命令 |

#### 1.2 路由与导航

| 采集项 | 分析方法 |
|--------|----------|
| 路由模式 | hash / history — 搜索 `createWebHashHistory` / `createWebHistory` / `HashRouter` 等 |
| 路由配置文件位置 | 搜索 `createRouter` / `new Router` / routes 定义文件 |
| 路由数据结构 | 读取路由配置，了解路由是静态定义还是动态加载 |
| 菜单接口（如有） | 搜索获取菜单/权限的接口调用 |
| 路由总数（估算） | 统计路由配置中的路由条目数量 |

#### 1.3 认证机制

| 采集项 | 分析方法 |
|--------|----------|
| 登录页路由 | 搜索 `/login` 相关路由定义 |
| 登录接口 | 搜索登录相关 API 调用 |
| Token 存储位置 | 搜索 `sessionStorage` / `localStorage` / `cookie` 中 token 的读写 |
| 路由守卫 | 读取 `beforeEach` / 路由守卫逻辑，了解未认证时的跳转行为 |
| 是否支持 devToken | 检查是否有开发环境直接注入 token 的机制 |

#### 1.4 页面布局

| 采集项 | 分析方法 |
|--------|----------|
| 根容器选择器 | 读 `index.html` 的挂载点（通常是 `#app`） |
| 布局组件 | 搜索侧边栏、导航栏、主内容区域的组件和选择器 |
| Loading 组件 | 搜索全局 Loading 遮罩的实现 |
| 404 页面 | 搜索 404 / NotFound 相关路由或组件 |

#### 1.5 接口特征

| 采集项 | 分析方法 |
|--------|----------|
| API 请求方式 | 搜索 axios / fetch 的封装（baseURL、拦截器等） |
| 接口 URL 特征 | 分析 API 调用模式，提取可用于区分业务接口的 URL 特征 |
| 业务成功码 | 搜索接口响应拦截器中的成功/失败判断逻辑 |
| 第三方脚本 | 搜索监控 SDK（Sentry、神策等）、地图、统计等第三方依赖 |

### Phase 2: 方案输出（等待确认）

基于采集结果，以下面格式输出搭建方案，**然后停下来等待用户确认**：

```
## 项目信息摘要

- **框架**: [xxx]
- **路由**: [模式 + 路由数量]
- **认证**: [token 存储方式 + 登录机制]
- **开发服务器**: [地址]

## 搭建方案

### 目录结构
[将要创建的文件列表]

### 认证方案
[Token 直注 / 登录流程 / 两者兼备]

### 路由数据源
[从哪里获取路由列表]

### 断言策略
[白屏检测的选择器 + 接口匹配规则 + 需要过滤的第三方脚本]

### npm scripts
[将要添加的执行命令]
```

**重要：输出方案后必须等待用户回复确认，不得自行开始生成代码。**

### Phase 3: 生成代码

用户确认方案后，按以下顺序生成代码。每完成一步都告知用户当前进度。

#### Step 1: 安装依赖与基础配置

- 安装 `@playwright/test` 和浏览器
- 生成 `playwright.config.ts`
- 配置 `.gitignore`（截图、报告、session-state）

#### Step 2: 认证模块

生成以下文件：
- `auth/auth.setup.ts` — 认证流程（根据项目实际的登录机制适配）
- `helpers/session-injector.ts` — context 级别 session 注入

**认证方案设计要点：**
- 优先支持 Token 直注模式（快速、稳定），同时保留完整登录流程作为备选
- 使用 `context.addInitScript()` 注入 session，保证在应用 JS 执行前完成
- context 级别注入可以自动覆盖 `window.open` 弹出的新页面

#### Step 3: 页面审计器

生成以下文件：
- `helpers/page-auditor.ts` — 页面审计核心
- `helpers/error-filters.ts` — 错误过滤规则

**审计器需要监听的事件：**

| 事件 | Playwright API | 用途 |
|------|---------------|------|
| JS 异常 | `page.on("pageerror")` | 捕获未捕获的运行时错误 |
| 控制台错误 | `page.on("console")` + type === "error" | 捕获 console.error |
| 接口响应 | `page.on("response")` | 捕获 HTTP 4xx/5xx 和业务错误码 |
| 网络失败 | `page.on("requestfailed")` | 捕获网络超时、断连 |

**白屏检测维度：**

| 检测项 | 判断标准 |
|--------|----------|
| 根容器存在 | 挂载点元素存在且高度 >= 100px |
| 内容非空 | 子元素数 > 1 或文本长度 > 10 |
| 主内容区域 | 主内容区高度 >= 50px |
| 404 检测 | 页面不包含 404 标识 |
| 框架错误 | 无框架错误覆盖层 |
| Loading 状态 | 无长时间停留的 Loading 遮罩 |

**硬断言 vs 软断言：**

| 类型 | 断言方式 | 说明 |
|------|----------|------|
| 白屏 | 硬断言 | 页面完全不可用，必须失败 |
| JS 异常 | 硬断言 | 未捕获错误，说明有严重 bug |
| 5xx 接口错误 | 硬断言 | 服务端异常 |
| 4xx 接口错误 | 软断言（warning） | 可能是权限问题 |
| 控制台错误 | 软断言（warning） | 经过滤后记录，不阻断 |
| 业务错误码 | 软断言（warning） | HTTP 200 但业务逻辑返回错误 |

**错误过滤原则：**
- 只过滤 `console.error`，**不过滤** `pageerror`（未捕获异常几乎一定是 bug）
- 过滤开发工具噪音：favicon、HMR、hot-update
- 过滤浏览器行为：ResizeObserver loop
- 过滤第三方 SDK：根据采集到的第三方脚本生成对应的过滤规则
- 过滤框架开发模式提示

#### Step 4: Fixture 与路由加载

生成以下文件：
- `fixtures/base.ts` — 自定义 Fixture（authedPage + auditor）
- `fixtures/route-loader.ts` — 路由数据加载（适配项目的路由数据源）

**Fixture 定义模式：**

```typescript
// fixtures/base.ts
import { test as base, Page } from "@playwright/test";

export const test = base.extend<{ authedPage: Page; auditor: PageAuditor }>({
  authedPage: async ({ context, page }, use) => {
    await injectSession(context);
    await use(page);
  },
  auditor: async ({ page }, use) => {
    const auditor = new PageAuditor(page);
    auditor.startListening();
    await use(auditor);
  },
});
```

#### Step 5: 测试用例

生成以下文件：
- `tests/login.spec.ts` — 登录测试（验证完整登录流程）
- `tests/smoke.spec.ts` — 冒烟测试（验证首页可加载）
- `tests/page-audit.spec.ts` — 全量路由巡检

**登录测试要点：**
- 不依赖 token 直注，必须走真实的表单提交流程
- 断言覆盖三个层面：页面跳转、布局渲染、session 状态
- 超时设置比普通页面宽松

**全量巡检模式：**

```typescript
for (const route of routes) {
  test(`${route.name} (${route.path})`, async ({ authedPage, auditor }) => {
    auditor.reset();
    await authedPage.goto(route.path);
    await authedPage.waitForTimeout(2000); // 等待页面稳定
    const result = await auditor.collectResults();

    // 硬断言
    expect(result.isBlank).toBe(false);
    expect(result.jsExceptions).toHaveLength(0);
    expect(result.serverErrors).toHaveLength(0);

    // 截图
    await authedPage.screenshot({ path: `screenshots/...`, fullPage: true });
  });
}
```

#### Step 6: npm scripts

在 `package.json` 中添加执行命令：

```json
{
  "test:e2e": "playwright test",
  "test:e2e:smoke": "playwright test --project=smoke",
  "test:e2e:audit": "playwright test --project=page-audit",
  "test:e2e:report": "playwright show-report e2e/reports/html",
  "test:e2e:headed": "playwright test --headed",
  "test:e2e:debug": "playwright test --debug"
}
```

### Phase 4: 验证

代码生成完成后：

1. 提示用户启动开发服务器
2. 指导用户配置认证凭据（环境变量或 `.env` 文件）
3. 先执行冒烟测试验证链路畅通：`npm run test:e2e:smoke`
4. 冒烟通过后再执行全量巡检：`npm run test:e2e:audit`
5. 指导用户查看报告：`npm run test:e2e:report`

## 截图与产物管理

生成的配置需包含以下产物策略：

| 场景 | 策略 |
|------|------|
| 全量巡检 | 每个页面全页截图，存放到 `screenshots/` |
| 失败用例 | Playwright 内置 `screenshot: "only-on-failure"` |
| Trace 录制 | `trace: "on-first-retry"` — 仅失败重试时录制 |
| 产物清理 | `screenshots/`、`reports/`、`session-state.json` 加入 `.gitignore` |

## 注意事项

- **语言**：使用中文与用户沟通
- **聚焦**：只搭建 E2E 巡检框架，不涉及业务逻辑测试
- **不要猜测**：如果某项信息无法通过分析确定（如登录页选择器），直接向用户提问
- **不修改业务代码**：整个过程只在 `e2e/` 目录下新增文件 + 修改 `package.json`，不改动任何业务源码
- **安装依赖时注意代理**：如果用户的 CLAUDE.md 中有关于代理的说明，遵循其指示
