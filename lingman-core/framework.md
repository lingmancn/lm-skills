# Lingman-Starter 框架核心规范

> 面向业务开发者的"怎么用"指南，不解释框架设计原理。

## 技术栈

- JDK 21
- Spring Boot 3
- MyBatis Plus
- PostgreSQL
- Redis
- Spring Security 6

## 包名规范

| 层级 | 包名前缀 |
|------|----------|
| 业务代码 | `com.lm.app` |
| 框架能力 | `com.lm.starter` |

Maven 坐标：`lingman-module-{biz_name}`

## 分层与文件位置

```
src/main/java/com/lm/app/
├── controller/
│   ├── admin/              # 管理端 Controller
│   │   └── {biz}/
│   │       └── {Name}Controller.java
│   │       └── vo/
│   │           └── {Name}PageReqVO.java
│   │           └── {Name}SaveReqVO.java
│   │           └── {Name}RespVO.java
│   └── app/                # 用户端 Controller（如有）
├── service/
│   └── {biz}/              # 业务服务接口 + ServiceImpl
│       └── {Name}Service.java
│       └── {Name}ServiceImpl.java
├── convert/
│   └── {biz}/              # MapStruct 转换器
│       └── {Name}Convert.java
├── models/
│   ├── entity/             # DO 实体类
│   ├── mapper/             # MyBatis Mapper
│   └── service/            # MP IService/ServiceImpl 包装（按需）
├── enums/                  # 枚举和错误码
└── config/                 # 配置类（AppDataPermissionConfiguration 等）
```

## DO 实体类

```java
@TableName("t_{name}")
@KeySequence("{name}_seq")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
public class {Name}DO extends BaseDO {

    @TableId(value = "id")
    private Long id;

    @TableField("xxx")
    private String xxx;
}
```

**规范要点**：
- 继承 `BaseDO`（未启用多租户）或 `TenantBaseDO`（启用了多租户，含 `tenant_id`）
- **ID 策略选择**（根据数据库和业务场景）：
  - **PostgreSQL/Oracle 序列**：`@KeySequence("{name}_seq")` + `@TableId(value = "id")`（推荐）
  - **MySQL 自增**：`@TableId(type = IdType.AUTO)`（简单场景）
  - **雪花算法**：`@TableId(type = IdType.ASSIGN_ID)`（分布式/需要全局唯一/严格审计场景）
- 必须提供 `@Builder`、`@NoArgsConstructor`、`@AllArgsConstructor`
- `@EqualsAndHashCode(callSuper = true)`
- `@TableName` 命名：`t_` 前缀 + 小写蛇形
- 字段用 `@TableField` 映射

## Mapper

```java
@Mapper
public interface {Name}Mapper extends BaseMapperX<{Name}DO> {
}
```

**规范要点**：
- 继承 `BaseMapperX<T>`
- 有 `@Mapper` 注解
- 无 XML 文件（全注解方式）

## Convert（MapStruct）

```java
@Mapper
public interface {Name}Convert {

    {Name}Convert INSTANCE = Mappers.getMapper({Name}Convert.class);

    {Name}DO convert({Name}SaveReqVO bean);

    {Name}RespVO convert({Name}DO bean);

    default PageResult<{Name}RespVO> convertPage(PageResult<{Name}DO> page) {
        if (page == null) {
            return null;
        }
        List<{Name}RespVO> list = page.getList() == null
            ? Collections.emptyList()
            : page.getList().stream().map(this::convert).toList();
        return new PageResult<>(list, page.getTotal());
    }
}
```

**规范要点**：
- 使用 MapStruct `@Mapper` 接口
- 单例模式：`INSTANCE = Mappers.getMapper(...)`
- 提供分页转换方法 `convertPage`
- 不要手写 set/get，也不要用 BeanUtils

## VO 定义

### SaveReqVO（创建/更新共用）

```java
@Data
@Schema(description = "管理后台 - {BizName}保存 Request VO")
public class {Name}SaveReqVO {

    @Schema(description = "编号（更新时必填）")
    private Long id;

    @Schema(description = "名称", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "名称不能为空")
    private String name;
}
```

### RespVO

```java
@Data
@Schema(description = "管理后台 - {BizName} Response VO")
public class {Name}RespVO {

    @Schema(description = "编号")
    private Long id;

    @Schema(description = "名称")
    private String name;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @Schema(description = "创建时间")
    private LocalDateTime createTime;
}
```

### PageReqVO

```java
@Data
@EqualsAndHashCode(callSuper = true)
@Schema(description = "管理后台 - {BizName}分页 Request VO")
public class {Name}PageReqVO extends PageParam {

    @Schema(description = "名称")
    private String name;
}
```

**规范要点**：
- 创建/更新共用 `SaveReqVO`（id 为空则创建，有值则更新）
- VO 用 `@Data` + `@Schema(description = "...")`
- 时间字段用 `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")`
- 分页 VO 继承 `PageParam`

## Service

```java
public interface {Name}Service {
    Long create({Name}SaveReqVO createReqVO);
    void update({Name}SaveReqVO updateReqVO);
    void delete(Long id);
    {Name}RespVO get(Long id);
    PageResult<{Name}RespVO> page({Name}PageReqVO pageReqVO);
}

@Service
public class {Name}ServiceImpl implements {Name}Service {

    @Resource
    private {Name}Mapper {name}Mapper;
    @Resource
    private {Name}Convert {name}Convert;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create({Name}SaveReqVO createReqVO) {
        {Name}DO entity = {name}Convert.convert(createReqVO);
        {name}Mapper.insert(entity);
        return entity.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void update({Name}SaveReqVO updateReqVO) {
        validate{Name}Exists(updateReqVO.getId());
        {Name}DO updateObj = {name}Convert.convert(updateReqVO);
        updateObj.clean();
        {name}Mapper.updateById(updateObj);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void delete(Long id) {
        validate{Name}Exists(id);
        {name}Mapper.deleteById(id);
    }

    @Override
    public {Name}RespVO get(Long id) {
        {Name}DO entity = {name}Mapper.selectById(id);
        if (entity == null) {
            throw exception(ErrorCodeConstants.{NAME}_NOT_EXISTS);
        }
        return {name}Convert.convert(entity);
    }

    @Override
    public PageResult<{Name}RespVO> page({Name}PageReqVO pageReqVO) {
        LambdaQueryWrapperX<{Name}DO> queryWrapper = new LambdaQueryWrapperX<>();
        queryWrapper.likeIfPresent({Name}DO::getName, pageReqVO.getName());
        queryWrapper.orderByDesc({Name}DO::getId);
        return {name}Mapper.selectPage(pageReqVO, queryWrapper)
            .convert({name}Convert::convert);
    }

    private void validate{Name}Exists(Long id) {
        if (id == null || {name}Mapper.selectById(id) == null) {
            throw exception(ErrorCodeConstants.{NAME}_NOT_EXISTS);
        }
    }
}
```

**规范要点**：
- 接口定义在 `service/{biz}/`，实现类在 `service/{biz}/{Name}ServiceImpl.java`
- 注入方式：`@Resource`
- 对象转换：MapStruct `convert` 方法
- 更新前调用 `updateObj.clean()` 清除审计字段
- 分页查询用 `LambdaQueryWrapperX` + `selectPage`
- 业务异常：`throw exception(ErrorCodeConstants.XXX)`
- 需要事务的方法加 `@Transactional(rollbackFor = Exception.class)`
- 私有校验方法：`validateXxxExists`

## Controller

```java
@Tag(name = "管理后台 - {BizName}")
@RestController
@RequestMapping("/admin/{biz}")
public class {Name}Controller {

    @Resource
    private {Name}Service {name}Service;

    @PostMapping("/create")
    @Operation(summary = "创建{BizName}")
    public CommonResult<Long> create(@Valid @RequestBody {Name}SaveReqVO createReqVO) {
        return success({name}Service.create(createReqVO));
    }

    @PutMapping("/update")
    @Operation(summary = "更新{BizName}")
    public CommonResult<Boolean> update(@Valid @RequestBody {Name}SaveReqVO updateReqVO) {
        {name}Service.update(updateReqVO);
        return success(true);
    }

    @DeleteMapping("/delete")
    @Operation(summary = "删除{BizName}")
    public CommonResult<Boolean> delete(@Valid @RequestBody AdminDeleteReqVO deleteReqVO) {
        {name}Service.delete(deleteReqVO.getId());
        return success(true);
    }

    @GetMapping("/get")
    @Operation(summary = "获得{BizName}详情")
    public CommonResult<{Name}RespVO> get(@RequestParam("id") Long id) {
        return success({name}Service.get(id));
    }

    @GetMapping("/page")
    @Operation(summary = "获得{BizName}分页")
    public CommonResult<PageResult<{Name}RespVO>> page(@Valid {Name}PageReqVO pageReqVO) {
        return success({name}Service.page(pageReqVO));
    }
}
```

**规范要点**：
- 必须加 `@Tag`、`@Operation` Swagger 注解
- 入参用 VO，出参用 `CommonResult<T>`
- 需要校验的入参加 `@Valid`
- URL 前缀统一 `/admin/{biz}`，kebab-case（如 `/detection-task`）
- RESTful HTTP 方法：POST 创建、PUT 更新、DELETE 删除、GET 查询/分页
- **接口路径不可重复**：禁止"同路径不同请求方式"（同一 URL 不得同时存在 GET 与 POST 等），全项目所有接口路径必须唯一
- **请求参数约定**：GET 请求可用 `@RequestParam` / `@PathVariable`；POST / PUT / DELETE / PATCH 等非 GET 请求**一律使用 `@RequestBody` 传参**，禁止 `@RequestParam` / `@PathVariable` / URL 查询参数 / 路径参数
- **单 ID 请求统一规则**：删除等仅需传递一个主键 id 的非 GET 接口，字段名**统一为 `id`**（禁止业务前缀如 `taskId`），**统一使用公共请求类 `AdminDeleteReqVO`**（路径 `controller/admin/common_vo/AdminDeleteReqVO.java`），通过 `deleteReqVO.getId()` 取值；禁止各业务模块自行定义 `{Name}DeleteReqVO` 等同义类，若项目尚无该类需先新增
- 注入方式：`@Resource`
- 使用 `CommonResult.success(...)` 静态导入
- 分页接口用 `@GetMapping` + `@Valid PageReqVO`（不是 `@RequestBody`），如果存在列表或者参数过多的情况下需要调整为`Post`请求

## 错误码定义

```java
public interface ErrorCodeConstants {
    ErrorCode {NAME}_NOT_EXISTS = new ErrorCode(1_100_000_001, "XXX不存在");
}
```

- 错误码前段为工程标识，本工程为 `1_100_000_xxx`
- 使用方式：`throw exception(ErrorCodeConstants.XXX)`

## 常用工具类

| 工具类 | 用途 |
|--------|------|
| `exception(ErrorCode)` | 抛业务异常 |
| `{Name}Convert.INSTANCE.convert(...)` | 对象转换 |
| `new LambdaQueryWrapperX<>()` | 构建查询条件 |

## 配置类命名

| 配置 | 类名 |
|------|------|
| 数据权限 | `AppDataPermissionConfiguration` |
| Flyway | `AppFlywayConfiguration` |
| Swagger | `AppSwaggerConfiguration` |

## Flyway 迁移

- 脚本位置：`src/main/resources/db/migration/app/postgresql/`
- 命名：`V{n}__{desc}.sql`
- 历史表：`flyway_history_app`

## 时间规范

- 统一使用 `LocalDateTime`
- 时间格式：`yyyy-MM-dd HH:mm:ss`

## 分页规范

- 分页 VO 继承 `PageParam`
- 返回 `PageResult<T>`
- 默认页码从 1 开始
