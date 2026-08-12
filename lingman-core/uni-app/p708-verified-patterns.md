# p708-app 已验证模式索引

> 本文记录从 p708-app 源码中核验过、对 AI 开发决策有价值的实现模式。
> 它不是目标项目的实时镜像。每次执行任务前必须重新读取下列事实源；若当前代码与本文不一致，以当前代码和用户要求为准。
> 禁止从目标项目复制环境地址、密钥、账号、AppID、证书、签名或其他敏感值到 Skill、回答和日志中。

## 工程定位

p708 是 APP 优先、同时保留 H5 与小程序构建能力的 uni-app 视频检测项目。技术栈、版本、包管理器和脚本以目标项目 `package.json` 为准。

开发时重点区分：

- `src/pages/`：主包业务页面；
- `src/pages-core/`：认证、错误页和用户中心等分包；
- `src/components/`：告警、监控等业务组件；
- `src/template/`：请求、路由、Store、布局、TabBar 和工具等模板基础设施；
- `src/api/auto/`：自动生成 API，禁止手改；
- `src/api/types/` 与 `src/api/meta/`：接口同步配套内容，是否可维护需按当前项目确认。

事实源：`package.json`、`CLAUDE.md`、`src/` 目录。

## 页面与 Manifest 生成关系

已验证模式：

- 页面在 SFC 中使用 `definePage()` 声明元信息；
- `pages.config.ts` 提供全局页面配置；
- Vite 页面插件生成 `src/pages.json`；
- `manifest.config.ts` 是 Manifest 配置源；
- Manifest 插件生成 `src/manifest.json`。

规则：

- 修改页面或平台配置时优先修改页面声明和 TypeScript 配置源；
- `src/pages.json`、`src/manifest.json` 虽为生成结果，但 p708 当前要求纳入版本管理；
- 不得因为是生成物就删除、忽略或拒绝提交；
- 修改 TabBar 等生成相关配置后，需要重新运行对应生成流程。若该流程会启动项目，必须先询问用户。

事实源：`pages.config.ts`、`manifest.config.ts`、`vite.config.ts`、`src/pages.json`、`src/manifest.json`、`.gitignore`。

## 自动 API 与请求桥接

已验证调用链：

```text
src/api/auto 中的 API 对象
  → 全局 Get/Post/Put/Delete
  → src/template/http/bridge.ts
  → httpGet/httpPost/httpPut/httpDelete
  → uni.request
  → 请求拦截器与统一响应处理
```

业务页面通常：

- 从 `src/api/auto/` 导入自动 API 对象；
- 为长对象名设置局部别名；
- 使用 `Awaited<ReturnType<typeof Api.method>>` 推导返回类型；
- 不重复手写 URL、请求 DTO 和响应 VO；
- 不绕开现有桥接另建 axios 客户端。

接口缺失时，应使用项目既有接口同步流程更新自动目录，不得手工补空壳文件。

事实源：`lingman.config.json`、`src/api/auto/`、`src/api/login.ts`、`src/template/http/bridge.ts`、`src/template/http/http.ts`、`src/template/http/interceptor.ts`。

## 双 Token 与统一错误处理

当前请求层兼容 HTTP 401 和业务码 401，并按认证模式处理：

- 单 Token：清理登录态并进入登录页；
- 双 Token 且存在 refreshToken：单次刷新、等待队列、刷新成功后重放请求；
- 无 refreshToken 或刷新失败：清理用户状态并避免并发重复跳转。

开发页面时：

- 不直接读写裸 Token；
- 不在页面复制刷新 Token 或登录跳转逻辑；
- 底层已经展示统一请求错误时，不重复弹相同 Toast；
- 登出需要同步清理用户、字典、告警等与身份相关的共享状态。

修改请求层时必须完整检查等待请求是否都能 resolve/reject、刷新接口是否会递归、原请求是否可能无限重试。具体实现以当前源码为准。

事实源：`src/template/http/http.ts`、`src/template/http/interceptor.ts`、`src/template/store/token.ts`、`src/template/store/user.ts`。

## 路由与 TabBar

当前项目默认页面需要登录，通过 `uni.addInterceptor` 拦截导航 API，并为 H5 首屏直达提供单独检查。

已验证规则：

- TabBar 页面使用 `uni.switchTab()`；
- 普通页面根据页面栈使用 `navigateBack`、`navigateTo`、`redirectTo` 或 `reLaunch`；
- 登录页携带 redirect，登录成功后返回目标页；
- 自定义 TabBar 使用缓存策略，页面可能只触发隐藏而不卸载；
- TabBar badge 按页面路径更新，不依赖固定下标；
- 动态图标必须考虑 UnoCSS safelist。

事实源：`src/config/router.ts`、`src/config/tabbar.ts`、`src/template/router/`、`src/template/tabbar/`、`src/App.vue`、`src/App.ku.vue`。

## Wot Design Uni、UnoCSS 与主题

已验证模式：

- Wot Design Uni 通过 easycom 使用；
- 全局 ConfigProvider、Toast 和 MessageBox 已在应用根容器挂载；
- 业务代码优先复用 Wot 组件与 `useToast()` 等能力；
- UnoCSS 使用 uni-app 预设，并包含 APP/低端 Android 兼容配置；
- 页面尺寸大量使用 `rpx`；
- 主题 Store 同时驱动 Wot theme vars 与 `--yd-*` 业务语义变量；
- 新页面优先使用语义变量，不继续扩散固定深色值；
- 动态图标或类名需要加入 `uno.config.ts` safelist。

事实源：`pages.config.ts`、`uno.config.ts`、`src/App.ku.vue`、`src/template/store/theme.ts`、`src/template/style/index.scss`、`src/uni.scss`。

## 列表与请求竞态

检测任务页面提供了可复用的移动列表模式：

- 搜索输入防抖；
- 新搜索/筛选统一清空旧分页；
- 请求序号与请求条件共同判断旧响应；
- 首屏和加载更多错误分离；
- 追加数据按 ID 去重；
- 状态修改使用记录级处理中状态；
- 页面卸载时清除定时器并使旧请求失效。

该模式是参考，不要求所有列表逐字复制；应根据页面复杂度采用最小充分实现。

事实源：`src/pages/detection-task/index.vue`、`src/components/alarm/AlarmListView.vue`。

## 跨端视频播放

p708 的视频能力按平台分支：

- H5：通过专用组件处理 WebRTC/ZLMediaKit 信令和浏览器 video；
- APP：把项目后端提供的流地址转换为 APP 可播放地址，再交给原生 `<video>`；
- 其他端：使用平台 `<video>` 播放项目提供的地址。

必须保留：

- H5/APP/其他平台的条件编译边界；
- 页面或 TabBar 隐藏时停止播放；
- 卸载时停止 Track、关闭 PeerConnection、清理 `srcObject`、计时器和监听器；
- 切换设备或流时防止旧异步结果回写；
- 不把完整设备对象或敏感流信息塞入路由参数，优先传稳定 ID 后重新查询。

事实源：`src/components/monitor/MonitorPlayer.vue`、`src/components/monitor/H5MonitorVideo.vue`、`src/template/utils/monitor.ts`、`src/template/utils/zlmWebrtc.ts`、监控相关页面。

## APP 联调与构建约束

- APP 请求不经过 H5 Vite 开发代理；
- 真机/模拟器中的 `localhost` 指当前设备自身，不等同于开发电脑；
- 运行和构建使用哪个 mode 必须从真实 script 判断；
- production 地址、原生插件、证书和签名等发布条件必须从当前配置确认；
- Vite 插件顺序具有依赖关系，不随意调整 pages、manifest、KuRoot、uni 和 UnoCSS 等插件顺序；
- `App.ku.vue` 属于当前根容器体系，不是可随意删除的备用文件。

事实源：`package.json`、`vite.config.ts`、`env/`、`manifest.config.ts`、`vite-plugins/`、`src/App.ku.vue`。

## 每次任务必须实时确认的内容

以下内容不在本文固化：

- 依赖版本和全部 scripts；
- 页面、分包和 TabBar 完整清单；
- API 方法、字段和状态值；
- 环境地址、代理目标和环境变量值；
- AppID、包名、权限、原生插件、证书和签名；
- 当前主题色、图标 safelist 和业务常量。
