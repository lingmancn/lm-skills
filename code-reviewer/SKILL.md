---
name: code-reviewer
description: Lingman-Starter 框架代码审查助手。当用户需要：(1) 检查代码是否符合公司框架规范 (2) 审查 CRUD 代码的分层和命名 (3) 检查权限注解、异常处理、事务配置 (4) 发现潜在的空指针、SQL 注入、越权访问风险 (5) 评审接口设计的合理性 时触发此技能。不要在以下场景触发：生成代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、错误排查（由 error-analyzer 处理）。
---

# 代码审查指南

## 审查维度

| 维度 | 检查项 | 严重程度 |
|------|--------|---------|
| **规范合规** | 包名、类名、URL 是否符合约定 | 高 |
| **分层正确** | Controller/Service/Mapper 是否越层调用 | 高 |
| **参数校验** | 入参是否有 `@Valid` / `@NotNull` | 高 |
| **权限控制** | 管理端接口是否有 `@PreAuthorize` | 高 |
| **异常处理** | 是否使用 `throw exception(ErrorCode)` | 中 |
| **事务配置** | 写操作是否有 `@Transactional` | 中 |
| **空值安全** | 数据库查询结果是否判空 | 中 |
| **SQL 安全** | 是否使用 `${}` 拼接 SQL | 高 |
| **返回格式** | 是否使用 `CommonResult<T>` 包装 | 中 |
| **VO 隔离** | Controller 是否直接返回 DO | 高 |
| **日志规范** | 是否用 `@Slf4j`、套用 `[线程id][操作人][模块][动作] key={}` 公式、含唯一 ID、异常末位、未打大对象、未拼字符串、未泄露敏感数据 | 中 |

> **前端审查维度**：详见 [frontend-spec.md](../lingman-core/frontend/frontend-spec.md)
> - API 层：是否从 `api/auto/` 导入自动生成的 API 对象（`import { ApiAppXxxAppAdminApiAuto }`），禁止直接手写 URL
> - **API 自动生成文件保护**：`src/api/auto/` 下的文件是否为命令自动生成，是否存在手动修改的痕迹（如手动增删接口、调整参数类型等）
> - 列表页：是否使用 `useTable` hook 管理列表状态，而非手动维护 `loading`/`total`/`list`
> - 列表页搜索表单：字段 ≥ 2 个时是否使用 `<Search>` 组件；仅 1 个搜索字段或需自定义样式时是否合理回退 `el-form`（详见 frontend-spec.md「搜索表单」说明）
> - 表单校验：`FormRules` 是否与后端 `@Valid` 规则一致（必填、格式、长度）
> - 错误处理：是否使用 `useMessage()` 或 `ElMessage`/`ElMessageBox`，异常捕获是否完整（`try/finally`）
> - 字典使用：表格中字典列是否使用 `<DictTag>`，下拉框是否使用 `getDictOptions` / `getIntDictOptions`
> - **组件/工具优先使用 `@lingman/yd`**：
>   - 列表表格是否优先使用 `<Table>` 组件，而非直接使用 `<el-table>`
>   - 弹窗是否优先使用 `<Dialog>` 组件（或 `el-dialog`），避免自己封装弹窗
>   - 日期格式化是否优先使用 `formatDate` / `dateFormatter`，避免手写日期转换
>   - 文件下载是否优先使用 `download.excel` / `download.word` 等，避免自己封装下载
>   - 字典查询是否优先使用 `getDictOptions` / `getDictLabel`，避免自己写字典查询逻辑
>   - 表单校验是否优先使用 `useValidator()` 的 `required()` / `lengthRange()`，避免自己写通用校验规则
>   - UUID/随机字符串是否优先使用 `generateUUID()` / `generateRandomStr()`，避免自己写生成逻辑
>   - 树形操作是否优先使用 `eachTree` / `findNode` / `treeToList`，避免自己写树遍历
>   - Token 操作是否优先使用 `getAccessToken()` / `setToken()`，避免自己写 Token 读写
> - 样式规范：是否优先使用 UnoCSS 原子类，组件样式是否加 `scoped`；数值表示间距、尺寸、宽高、`gap`、定位或平移偏移时是否显式带 CSS 单位，语义关键字、命名令牌、比例/分数等排除项按 [frontend-spec.md](../lingman-core/frontend/frontend-spec.md)「11.1 优先使用 UnoCSS 原子类」执行
> - 权限控制：按钮是否使用 `v-hasPermi`，JS 逻辑中是否使用 `checkPermi`

## 审查清单

### Controller 层

- [ ] 类上有 `@Tag` + `@RestController` + `@RequestMapping`
- [ ] 管理端 URL 前缀正确：`/admin/{biz}`（管理端）、`/app/{biz}`（用户端）
- [ ] 类上有 `@Validated`
- [ ] 管理端方法有 `@PreAuthorize("@ss.hasPermission('xxx')")`
- [ ] 入参有 `@Valid`（POST/PUT）
- [ ] 接口路径全局唯一，无"同路径不同请求方式"冲突
- [ ] 非 GET 请求（POST/PUT/DELETE/PATCH）一律使用 `@RequestBody`，未误用 `@RequestParam` / `@PathVariable`
- [ ] 单 ID 删除/操作统一使用公共类 `AdminDeleteReqVO`（`common_vo/AdminDeleteReqVO.java`），字段名为 `id`，未自行定义 `{Name}DeleteReqVO`
- [ ] 返回类型为 `CommonResult<T>`
- [ ] Swagger 注解完整：`@Operation(summary = "...")`

### Service 层

- [ ] 接口定义在 `service/admin/` 或 `service/app/`
- [ ] 实现类在 `service/admin/impl/` 或 `service/app/impl/`
- [ ] 实现类有 `@Service`
- [ ] 写操作有 `@Transactional(rollbackFor = Exception.class)`
- [ ] 使用 `throw exception(ErrorCodeConstants.XXX)` 抛业务异常
- [ ] 查询结果判空后再使用

### 日志

- [ ] 统一使用 Lombok `@Slf4j`（字段名 `log`），未手写 `LoggerFactory.getLogger(...)`
- [ ] **套用结构化公式** `[线程id] [操作人] [业务模块] [业务动作], key={}, ...`，带四要素前缀
- [ ] 每条日志**包含唯一标识符（ID）**，可追溯具体记录
- [ ] 用 `{}` 占位符，未用字符串拼接
- [ ] 异常对象作为最后一个参数，未拼进消息、未吞堆栈（不只 `e.getMessage()`）
- [ ] 未打印完整大对象（DO/VO/List/Map），只打关键 ID 或摘要
- [ ] 未写自然语言长句
- [ ] 昂贵操作用 `isDebugEnabled()` / `isInfoEnabled()` 守卫
- [ ] 日志级别合理（ERROR/WARN/INFO/DEBUG）
- [ ] 未记录密码、token、身份证、手机号、银行卡等敏感数据
- [ ] 循环/高频路径未打 `INFO` 及以上日志

### VO 层

- [ ] 有 `@Schema(description = "...")`
- [ ] 分页 VO 继承 `PageParam`
- [ ] 时间字段有 `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")`
- [ ] 校验注解使用正确：`@NotBlank`、`@NotNull`、`@Size`

### Mapper/DO 层

- [ ] DO 继承 `BaseDO`
- [ ] `@TableName` 命名正确：`t_xxx`
- [ ] Mapper 继承 `BaseMapperX<T>` 且有 `@Mapper`
- [ ] 字段用 `@TableField` 映射

## 常见问题及修复

### 问题：Controller 直接返回 DO
```java
// 错误
public CommonResult<AnnouncementDO> get(Long id) {
    return CommonResult.success(announcementMapper.selectById(id));
}

// 正确
public CommonResult<AnnouncementRespVO> get(Long id) {
    return CommonResult.success(announcementService.get(id));
}
```

### 问题：缺少权限注解
```java
// 错误
@GetMapping("/delete")
public CommonResult<Boolean> delete(...) { }

// 正确
@GetMapping("/delete")
@PreAuthorize("@ss.hasPermission('app:announcement:delete')")
public CommonResult<Boolean> delete(...) { }
```

### 问题：未判空直接使用
```java
// 错误
AdminUserDO user = adminUserMapper.selectById(userId);
return user.getNickname();

// 正确
AdminUserDO user = adminUserMapper.selectById(userId);
if (user == null) {
    throw exception(USER_NOT_EXISTS);
}
return user.getNickname();
```

### 问题：需要幂等但未加注解
```java
// 以下场景需要考虑幂等：
// - 用户可能重复点击的提交操作（创建、支付、审批提交）
// - 网络重试可能导致重复请求的场景
// - 表单提交、按钮点击类接口

// 错误：重复点击可能产生重复数据
@PostMapping("/submit")
public CommonResult<Long> submit(@RequestBody XxxReqVO reqVO) { }

// 正确：添加幂等保护
@Idempotent(timeout = 10, timeUnit = TimeUnit.SECONDS, keyResolver = UserIdempotentKeyResolver.class)
@PostMapping("/submit")
public CommonResult<Long> submit(@RequestBody XxxReqVO reqVO) { }

// 无需幂等的场景：查询、列表、获取详情等幂等操作
@PostMapping("/page")
public CommonResult<PageResult<XxxRespVO>> page(@RequestBody XxxPageReqVO reqVO) { }
```

### 问题：VO 缺少数据翻译
```java
// 错误：字典值直接返回数字，前端无法展示文本
@Data
public class XxxRespVO {
    private Integer status;  // 前端看到 0/1，不知道含义
}

// 正确：使用 @Trans 自动翻译
@Data
public class XxxRespVO implements VO {
    @Trans(type = TransType.SIMPLE, target = DictDataDO.class, fields = "label", ref = "statusName")
    private Integer status;
    private String statusName;  // 自动填充为"启用"/"禁用"
}
```

### 问题：忘记清除审计字段
```java
// 错误：更新时未清除 BaseDO 的审计字段
XxxDO updateObj = xxxConvert.convert(reqVO);
xxxMapper.updateById(updateObj);

// 正确：先 clean() 再更新
XxxDO updateObj = xxxConvert.convert(reqVO);
updateObj.clean();
xxxMapper.updateById(updateObj);
```

### 问题：未注册数据权限
```java
// 错误：新实体没有在 AppDataPermissionConfiguration 中注册
// 导致数据权限拦截器无法过滤该实体的查询

// 正确：在 AppDataPermissionConfiguration 中添加
@Bean
public DeptDataPermissionRuleCustomizer appDeptDataPermissionRuleCustomizer() {
    return rule -> {
        rule.addUserColumn(XxxDO.class, "user_id");
        rule.addDeptColumn(XxxDO.class, "dept_id");
    };
}
```

### 问题：时间字段缺少格式化
```java
// 错误：时间返回为数组格式 [2026, 1, 1, 12, 0, 0]
private LocalDateTime createTime;

// 正确：添加 @JsonFormat
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime createTime;
```

### 问题：日志不符合结构化公式（缺四要素 / 长句 / 拼接 / 打印大对象）

```java
// ❌ 错误：自然语言长句 + 字符串拼接 + 吞堆栈 + 打印大对象，无法追溯来源
@Service
public class XxxServiceImpl implements XxxService {
    public void doSomething(Long id) {
        log.info("处理完成了，id=" + id + "，对象是" + xxxDO);
        log.error("处理失败: " + e.getMessage());
    }
}

// ✅ 正确：[线程id] [操作人] [业务模块] [业务动作] + 唯一 ID + 异常末位
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class XxxServiceImpl implements XxxService {
    public void doSomething(Long id) {
        log.info("[{}] [{}] [XXX模块] [处理XXX], id={}",
                 Thread.currentThread().threadId(), SecurityFrameworkUtils.getLoginUserId(), id);
        log.error("[{}] [{}] [XXX模块] [处理失败], id={}",
                  Thread.currentThread().threadId(), SecurityFrameworkUtils.getLoginUserId(), id, e);
    }
}
```

> 五要素（线程id + 操作人 + 模块 + 动作 + 唯一ID）齐全，才能让人 grep 追溯到具体来源。

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 审查检查项清单 | [review-checklist.md](references/review-checklist.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
| 代码模板 | [code-template.md](../crud-generator/references/code-template.md) |
