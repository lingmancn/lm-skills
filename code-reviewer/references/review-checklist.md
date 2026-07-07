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
