---
name: uni-app-tooling
description: Lingman uni-app 工具链、运行与排障助手。当用户需要：(1) 探测并执行 uni-app 项目真实 dev/build/typecheck/lint scripts；(2) 启动或调试 APP、H5、微信/其他小程序；(3) 处理 env、接口地址、Vite 代理、localhost、真机联调和跨域；(4) 排查 uni.request、401 Token 刷新并发队列；(5) 使用 lm api 同步接口；(6) 维护 pages/manifest 生成配置；(7) 处理平台编译、Manifest、构建和产物问题 时触发。不要在以下场景触发：业务页面、组件、Pinia、样式和媒体功能实现（uni-app-feature）、后端 CRUD（crud-generator）、纯规范问答（doc-qa）、纯代码审查（code-reviewer）。任何启动项目、服务、模拟器、真机、开发者工具或 HBuilderX 的操作，执行前必须询问用户并获得明确同意。
---

# uni-app 工具链工作指南

## 参考规范

| 场景 | 文档 |
|---|---|
| 工程探测、环境、代理、请求、生成配置与构建 | [tooling-guide.md](../lingman-core/uni-app/tooling-guide.md) |
| uni-app 通用项目与跨端规范 | [uni-app-spec.md](../lingman-core/uni-app/uni-app-spec.md) |
| 仅当目标是 p708 或架构高度相似项目时 | [p708-verified-patterns.md](../lingman-core/uni-app/p708-verified-patterns.md) |

本 Skill 负责工程操作与排障流程，不复制共享规范。项目当前配置与共享索引不一致时，以当前项目为准并说明差异。

## 核心原则

1. **先探测后执行**：读取项目根目录、包管理器、依赖、scripts、配置源、目标平台和请求层。
2. **启动前必须询问**：dev/watch、前后端服务、浏览器、模拟器、真机、开发者工具、HBuilderX、预览和重启均需明确同意。
3. **只用真实命令**：命令必须来自 `package.json#scripts`、workspace 或项目文档，不编造。
4. **不切换包管理器**：根据 `packageManager`、锁文件和 workspace 判断，不制造第二种锁文件。
5. **平台分开诊断**：H5 proxy、APP 网络、小程序合法域名和各端构建不能混为一谈。
6. **不手改生成物**：自动 API、build 产物及项目标记的自动配置按真实生成流程维护。
7. **最小变更**：不顺带升级依赖、重写请求层、删除锁文件或格式化无关代码。
8. **结果如实汇报**：未执行、只静态推导、缺少 SDK/证书或命令不存在都要明确说明。

## Step 1 — 探测工程

### 根目录和技术路线

确认实际 uni-app 应用目录，尤其注意 monorepo。读取：

- `package.json`、锁文件、workspace 配置；
- `@dcloudio/*`、Vue、Vite/webpack、TypeScript；
- `src/App.vue`、`src/main.*`、`uni.scss`、`uni_modules/`；
- pages/manifest 的 JSON、TypeScript 配置和 Vite 插件；
- Pinia、UnoCSS、UI 库、请求层和自动 API；
- 项目文档中的 HBuilderX 或平台要求。

### scripts

完整列出真实开发、构建、类型检查、Lint、测试和同步脚本。常见名称只能用于搜索，不能直接执行。

如果目标命令不存在：

1. 报告项目未定义该 script；
2. 检查 workspace 根命令、HBuilderX 或其他入口；
3. 不创建或编造替代命令。

### 目标平台

确认 APP Android/iOS、H5、微信/其他小程序，以及开发、测试或生产 mode。用户已经明确时不重复询问；平台差异会改变方案但用户未说明时再询问。

## Step 2 — 启动确认

以下均属于启动操作：

- `dev:*`、serve/start/watch；
- Vite/webpack 开发服务器；
- HBuilderX 运行到浏览器、APP、模拟器或真机；
- 微信等开发者工具；
- 本地代理、mock、配套后端；
- 重启已有服务。

执行前展示：

```text
准备执行：<真实命令>
运行目录：<项目根目录>
目标平台：<平台与 mode>
该操作会启动常驻进程或外部工具，可能占用端口。是否现在启动？
```

用户明确同意后才能执行。普通 typecheck、lint 和无外部副作用的一次性 build 可直接执行；若 build 会打开 GUI、签名、云打包或持续监听，也必须先询问。

## Step 3 — 平台与网络排障

### H5

检查：

- Vite proxy 的匹配前缀、target、rewrite 和最终 URL；
- 路由 mode、base、静态资源路径；
- CORS、Cookie、SameSite、HTTPS；
- env 被哪个 mode 实际加载。

H5 开发代理只对浏览器开发服务器生效，不是 APP 或小程序方案。

### APP

检查：

- 真机/模拟器能否访问后端；
- `localhost` 是否错误指向设备自身；
- 后端监听地址、防火墙、同网段、HTTP/HTTPS 策略；
- Android/iOS 权限、模块和原生插件；
- APP build 的真实产物和后续 HBuilderX/云打包步骤。

不能宣称 APP build 必然产出 APK/IPA。

### 小程序

检查：

- 具体平台 script、appid、输出目录；
- HTTPS、合法域名、证书和端口；
- 分包、权限和包体限制；
- 开发者工具调试开关与真机差异。

开发者工具可用不代表真机可用；H5 proxy 不能解决真机合法域名问题。

## Step 4 — env 与地址

读取项目真实 `.env*`、构建 mode、`loadEnv`、`envPrefix` 和代码读取方式。确认：

- script 加载哪个 mode；
- 变量名和前缀是否正确；
- env 覆盖顺序；
- H5 是否使用代理；
- APP/小程序最终请求地址；
- 是否出现重复路径或斜杠；
- 客户端环境变量是否含不可公开的密钥。

不得根据常见命名直接断言某变量生效，也不得把模拟器专用地址设为全平台默认。

## Step 5 — 401 刷新队列

修改前完整读取请求封装、拦截器、Token Store、刷新 API、退出登录和登录跳转。

检查并发 401 是否满足：

1. 同时最多一次刷新；
2. 其他请求等待同一结果；
3. 更新新 Token 后再重放；
4. 原请求最多自动重试一次；
5. refresh/login 接口不触发刷新自身；
6. 刷新失败统一清理会话并拒绝全部等待请求；
7. 提示和跳转只执行一次；
8. 所有等待 Promise 都能结束；
9. 成功失败都清理刷新状态；
10. 不记录 Token 或 Authorization。

优先保持项目现有协议和桥接，只做最小修复。可使用共享 `refreshPromise` 或项目已有等价 single-flight 机制。

## Step 6 — API 同步

执行 `lm api` 或项目其他同步命令前确认：

- 当前目录是前端根目录；
- `lingman.config.json` 或真实 CLI 配置存在；
- CLI 已安装；
- Swagger 地址来自配置且文档已更新；
- 自动目录是否有未提交修改。

规则：

- 自动目录禁止手改；
- uni-app 适配维护在自动目录外；
- 生成后检查 git diff 和 typecheck；
- Swagger 需要启动/重启后端时先询问用户；
- CLI 未安装时报告，不擅自全局安装；
- 不创建空壳生成文件绕过同步失败。

## Step 7 — pages 与 Manifest

识别项目采用：

- 直接维护 JSON；
- `pages.config.*` / `manifest.config.*`；
- 页面 `definePage()`；
- Vite 插件或其他生成脚本；
- HBuilderX 可视化配置。

若为生成式配置：

- 修改真实配置源；
- 不直接修补最终 JSON；
- 用项目真实流程重新生成；
- 生成流程属于 dev/watch 时先询问；
- 生成后检查 JSON 和 git diff；
- JSON 是否提交以项目规则为准。

页面变更检查真实路径、主包/分包、TabBar 和导航方式。Manifest 只修改目标平台节点，不覆盖用户 AppID、证书、签名和其他平台配置。

## Step 8 — 校验与构建

只执行真实存在的命令，建议顺序：

1. 局部检查；
2. typecheck；
3. lint；
4. 受影响平台的一次性 build；
5. 检查生成配置、产物和日志；
6. 需要运行态、模拟器或真机验收时询问用户。

规则：

- 默认不执行会产生额外修改的 `lint:fix`；
- 未经用户要求不新增测试文件或测试框架，可运行项目已有测试；
- H5 build 成功不能代表 APP/小程序成功；
- 缺少 SDK、证书、HBuilderX 或 script 时如实报告；
- 不通过删锁文件、全量升级依赖或清空配置碰运气。

## 标准场景

### 运行某个平台

1. 定位根目录并读取真实 script；
2. 确认平台与 mode；
3. 展示命令和目录；
4. 询问用户；
5. 获得同意后启动；
6. 报告端口、日志和停止方式。

### 网络/代理/localhost

1. 确认平台和运行设备；
2. 读取 env、Vite 配置和请求基地址；
3. 展开最终 URL；
4. 判断请求从哪台设备发出；
5. 分别检查网络、CORS、合法域名、HTTPS 和权限；
6. 做最小修改，运行态验证前询问。

### 401 并发刷新

1. 画出请求、401、刷新、重放和失败链路；
2. 检查 single-flight、重试上限、刷新接口绕过和队列清理；
3. 最小范围修改；
4. 执行 typecheck/lint；
5. 验证并发刷新成功、刷新失败和重放仍 401。

### pages/manifest 不更新

1. 查找真实配置源和生成插件；
2. 修改源文件；
3. 确认生成命令是否会启动；
4. 生成并检查最终 JSON；
5. 按项目规则纳入变更。

## 输出要求

```text
项目探测：
- 根目录：
- 包管理器：
- uni-app 路线：
- 目标平台：
- 真实 scripts：

完成内容：
- ...

验证结果：
- typecheck：
- lint：
- 目标平台 build：
- 运行态验证：未执行/已执行（原因或结果）

注意事项：
- ...
```

任何未执行项必须说明原因，不得写成已通过。
