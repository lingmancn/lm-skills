# 代码审查检查清单

## Controller 层检查项

### 必须项（阻塞性问题）

- [ ] **包名正确**：`com.lm.app.controller.admin.{biz}` 或 `com.lm.app.controller.app.{biz}`
- [ ] **类注解完整**：`@Tag` + `@RestController` + `@RequestMapping` + `@Validated`
- [ ] **URL 前缀正确**：
  - 管理端：`/admin/{biz}`
  - 用户端：`/app/{biz}`
- [ ] **权限注解**：管理端接口必须有 `@PreAuthorize("@ss.hasPermission('xxx')")`
- [ ] **入参校验**：POST/PUT 请求体必须有 `@Valid`
- [ ] **返回包装**：返回类型必须是 `CommonResult<T>`

### 建议项（警告）

- [ ] **Swagger 注解**：方法有 `@Operation(summary = "...")`
- [ ] **参数命名**：`@RequestParam` 有明确的 value，如 `@RequestParam("id")`
- [ ] **URL 风格**：使用 kebab-case，如 `/user-list` 而非 `/userList`
- [ ] **幂等注解**：可能重复点击的提交操作（创建、支付、审批）是否加了 `@Idempotent`

## Service 层检查项

### 必须项

- [ ] **接口位置**：`service/admin/{Name}Service.java` 或 `service/app/{Name}Service.java`
- [ ] **实现位置**：`service/admin/impl/{Name}ServiceImpl.java`
- [ ] **实现注解**：类上有 `@Service`
- [ ] **事务注解**：写操作有 `@Transactional(rollbackFor = Exception.class)`
- [ ] **异常处理**：使用 `throw exception(ErrorCodeConstants.XXX)`
- [ ] **查询判空**：`selectById` 后判空，不存在时抛异常
- [ ] **对象转换**：使用 MapStruct `@Mapper` + `convert` 方法，不使用 BeanUtils
- [ ] **查询方式**：使用 `Wrappers.lambdaQuery(EntityDO.class)` 或 `LambdaQueryWrapperX`

### 建议项

- [ ] **注入方式**：使用 `@Resource`（与项目风格一致）
- [ ] **日志记录**：复杂操作添加操作日志 `@LogRecord`
- [ ] **批量操作**：批量插入使用 `insertBatch`
- [ ] **租户设置**：插入时是否设置了 `tenantId`（通过 `TenantContextHolder.getTenantId()`）
- [ ] **BPM 集成**：审批类业务是否正确启动了 Flowable 流程

## VO 层检查项

### 必须项

- [ ] **类注解**：`@Data` + `@Schema(description = "...")`
- [ ] **分页继承**：分页 VO 必须继承 `PageParam`
- [ ] **时间格式**：`LocalDateTime` 字段有 `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")`

### 建议项

- [ ] **校验注解**：必填字段有 `@NotNull` / `@NotBlank` / `@Size`
- [ ] **字段注释**：所有字段有 `@Schema(description = "...")`
- [ ] **敏感字段**：密码等敏感信息加 `@JsonIgnore`
- [ ] **数据翻译**：需要字典翻译的字段有 `@Trans` 注解，VO 实现 `VO` 接口

## 管理后台 Web 样式检查项

### 必须项（阻塞性问题）

- [ ] **数值尺寸类单位**：新建或生成代码中的间距、尺寸、宽高、`gap`、定位偏移、平移偏移等数值 UnoCSS 类必须显式带 CSS 单位；发现 `p-4`、`top-0` 等无单位写法时阻塞通过
- [ ] **排除项**：语义关键字、比例/分数等排除项不按尺寸类报错，具体分类以 [frontend-spec.md](../../lingman-core/frontend/frontend-spec.md)「11.1 优先使用 UnoCSS 原子类」为准
- [ ] **组件样式作用域**：组件内样式已添加 `scoped`

## uni-app 检查项

> 仅在确认目标是 uni-app 后使用。通用规则以 [uni-app-spec.md](../../lingman-core/uni-app/uni-app-spec.md) 为准；p708 或相似工程再核对 [p708-verified-patterns.md](../../lingman-core/uni-app/p708-verified-patterns.md)。

### 页面与路由（必须项）

- [ ] **配置源正确**：先识别 `definePage()`、`pages.config.*`、`manifest.config.*` 和生成插件；未直接修补会被覆盖的生成 JSON
- [ ] **页面注册完整**：页面路径、主包/分包、导航栏和页面样式与真实配置源一致
- [ ] **TabBar 导航正确**：TabBar 页面使用 `uni.switchTab()`；非 TabBar 页面按项目路由封装选择 `navigateTo`、`redirectTo` 或 `reLaunch`
- [ ] **路由参数最小化**：只传稳定 ID 或必要标识，不在 URL 中携带完整对象、Token、播放密钥或敏感地址
- [ ] **登录检查一致**：复用项目已有路由拦截与 H5 首屏检查，未在单页重复实现另一套鉴权跳转

### API、请求与认证（必须项）

- [ ] **自动目录保护**：项目存在自动 API 体系时，未手改 `src/api/auto/` 或其他受保护生成目录，也未创建空壳文件伪造同步结果
- [ ] **复用请求桥接**：页面和 Store 使用项目已有 API 对象、`Get/Post/Put/Delete` 或等价桥接，未另建 axios/fetch 请求链绕过拦截器
- [ ] **类型来源可靠**：优先从生成 API 返回值或项目已有类型推导，未复制一份易漂移的响应接口
- [ ] **错误分层正确**：区分网络失败、HTTP 非 2xx、业务码失败和 401；未将 `uni.request` 进入 success 误判为业务成功
- [ ] **401 single-flight**：并发 401 最多触发一次刷新，等待请求可重放或拒绝，原请求有重试上限，刷新接口不会递归刷新
- [ ] **刷新失败收口**：只清理、提示和跳转一次；所有等待 Promise 都能结束；成功失败均释放刷新状态
- [ ] **敏感信息安全**：未记录或展示完整 Token、Authorization、密码、证书、签名、AppID 或敏感响应体

### 状态、列表与生命周期

- [ ] **Pinia 边界合理**：跨页面、跨 Tab 或会话状态进入 Store；页面局部 loading、表单草稿和弹层状态保留在页面/组件
- [ ] **账号状态可清理**：退出登录或账号切换时清除持久化的账号相关状态，未串用上一账号数据
- [ ] **列表竞态防护**：搜索有防抖或等价控制；新筛选重置分页；旧响应不能覆盖新结果；追加数据按稳定 ID 去重
- [ ] **错误状态分离**：首屏失败与加载更多失败分开处理；记录级操作使用记录级 loading，未锁死整页
- [ ] **生命周期完整**：`onHide`、`onUnload` 或组件卸载时清理定时器、watch、请求失效标记、事件监听和长连接
- [ ] **缓存 Tab 页适配**：自定义缓存 TabBar 场景考虑页面只触发 `onHide` 而不卸载，资源不会在后台继续占用

### UI、样式与跨端

- [ ] **复用项目 UI 库**：Wot Design Uni、uni-ui 或项目已有组件优先，未把 Element Plus、管理后台 `<Table>/<Dialog>` 或 `@lingman/yd` Web 模板套入移动端
- [ ] **UnoCSS 单位正确**：数值尺寸类显式带单位，包括零值，如 `p-24rpx`、`top-0px`；未使用 `p-24`、`top-0` 等无单位尺寸类
- [ ] **主题与安全区**：使用项目语义变量和主题 Store，处理顶部/底部安全区、TabBar 高度与键盘顶起，未散落硬编码主题色
- [ ] **动态图标可生成**：动态拼接的图标或类名已加入 safelist 或采用项目可静态识别方案
- [ ] **条件编译隔离**：APP/H5/小程序专属代码置于最小范围条件编译，浏览器 API、DOM 和原生插件未泄漏到其他端
- [ ] **平台能力有兜底**：目标端不支持的能力有明确降级、提示或替代方案，未静默失败

### 视频与媒体资源（涉及时必须项）

- [ ] **平台播放链正确**：按项目真实架构保留各端播放方案（如 H5 浏览器 video/WebRTC、APP 原生 video 等），未为统一代码强行改写协议
- [ ] **切换竞态受控**：快速切换媒体源/数据源时，旧信令或旧请求完成后不能回写当前播放器
- [ ] **媒体资源释放**：停止 PeerConnection、MediaStreamTrack，清空 `srcObject` 和 video src，并释放定时器、网络请求、watch 与事件监听
- [ ] **显隐生命周期覆盖**：页面隐藏、组件卸载、路由切换和播放失败路径均能执行清理，不只依赖 `onUnload`

### 验证范围

- [ ] **静态与运行验证区分**：typecheck/lint/build 结果未冒充真机、模拟器或浏览器运行验证
- [ ] **平台分别验证**：H5 成功未推断 APP/小程序成功；APP 优先需求至少执行真实 APP 构建或如实说明未验证原因
- [ ] **启动权限合规**：启动 dev server、后端、模拟器、真机、HBuilderX 或开发者工具前已获得用户明确同意

## Mapper/DO 层检查项

### 必须项

- [ ] **DO 基类**：根据项目多租户配置选择 `BaseDO`（未启用）或 `TenantBaseDO`（启用）
- [ ] **表名注解**：`@TableName("t_xxx")` 小写蛇形
- [ ] **Mapper 注解**：接口上有 `@Mapper`
- [ ] **Mapper 继承**：继承 `BaseMapperX<T>`
- [ ] **ID 策略**：根据数据库和业务场景选择：
  - PostgreSQL/Oracle：`@KeySequence("xxx_seq")` + `@TableId(value = "id")`
  - MySQL 自增：`@TableId(type = IdType.AUTO)`
  - 分布式/全局唯一：`@TableId(type = IdType.ASSIGN_ID)`（雪花算法）

### 建议项

- [ ] **字段映射**：所有字段有 `@TableField`
- [ ] **构造器**：类上有 `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor`

## 数据权限检查项

### 必须项

- [ ] **配置注册**：新实体是否在 `AppDataPermissionConfiguration` 中注册了数据权限规则
- [ ] **字段映射**：`rule.addUserColumn(EntityDO.class, "user_id")` 用户字段是否正确
- [ ] **部门映射**：`rule.addDeptColumn(EntityDO.class, "dept_id")` 部门字段是否正确

### 示例

```java
rule.addUserColumn(AnnouncementDO.class, "creator");
rule.addDeptColumn(AnnouncementDO.class, "dept_id");
```

## 权限配置检查项

### 必须项

- [ ] **Controller 权限**：每个管理端方法有对应的 `@PreAuthorize`
- [ ] **权限标识**：权限标识格式为 `stuff:{biz}:{action}`（或 `app:{biz}:{action}`）
- [ ] **菜单 SQL**：新模块是否在 `system_menu` 中插入了菜单和按钮权限

## 安全检查项

### 必须项

- [ ] **SQL 注入**：禁止 `${}` 拼接 SQL，使用 `#{}`
- [ ] **越权访问**：查询时是否带了租户过滤（框架自动，但自定义 SQL 需检查）
- [ ] **敏感数据**：返回 VO 不包含密码、token 等敏感字段
- [ ] **幂等控制**：重复提交场景是否有 `@Idempotent` 保护

## 性能检查项

### 建议项

- [ ] **N+1 查询**：循环中是否查询数据库
- [ ] **分页查询**：列表接口是否使用分页
- [ ] **批量操作**：批量插入/更新是否使用批量接口
- [ ] **联表查询**：复杂查询是否使用 `MPJLambdaWrapperX` 而非多次单表查询
