# 常见错误模式及解决方案

## Java 运行时异常

### NullPointerException

**日志特征**：
```
java.lang.NullPointerException: Cannot invoke "..." because "..." is null
    at com.lm.app.service.admin.XxxServiceImpl.method(XxxServiceImpl.java:42)
```

**根因分析**：
1. `@Resource` / `@Autowired` 注入失败（bean 未扫描到）
2. 数据库查询返回 null，后续直接调用方法
3. 请求参数未传，但代码未判空

**修复方案**：
```java
// 注入检查
@Resource
private XxxMapper xxxMapper;  // 确认 Mapper 有 @Mapper 注解

// 查询判空
XxxDO entity = xxxMapper.selectById(id);
if (entity == null) {
    throw exception(ErrorCodeConstants.XXX_NOT_EXISTS);
}

// 参数判空
if (reqVO.getName() == null || reqVO.getName().isBlank()) {
    throw exception(ErrorCodeConstants.XXX_PARAM_EMPTY);
}
```

### IllegalArgumentException

**日志特征**：
```
java.lang.IllegalArgumentException: Result Maps collection does not contain value for...
```

**根因分析**：MyBatis Mapper XML 中的 resultMap 引用错误，或 Mapper 接口与 XML 不匹配。

**修复方案**：检查 Mapper 接口名与 XML 文件名是否一致，确认 XML 放在 `resources/mapper/` 目录下。

## Spring Boot 启动异常

### BeanCreationException

**日志特征**：
```
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'xxxServiceImpl': 
Injection of autowired dependencies failed
```

**根因分析**：
1. 循环依赖（A 依赖 B，B 依赖 A）
2. `@Autowired` 的 bean 不存在
3. 配置属性缺失

**修复方案**：
- 循环依赖：使用 `@Lazy` 延迟注入，或重构代码消除循环
- Bean 不存在：检查包扫描路径 `@SpringBootApplication(scanBasePackages = "com.lm")`
- 配置缺失：检查 `application.yaml` 中相关配置

### UnsatisfiedDependencyException

**日志特征**：
```
UnsatisfiedDependencyException: Error creating bean with name 'xxx' 
dependency 'yyy' expected at least 1 bean of type...
```

**修复方案**：确认缺失的 Bean 所在类有 `@Component`、`@Service` 或 `@Configuration` 注解，且包路径在扫描范围内。

## MyBatis Plus 异常

### Invalid bound statement

**日志特征**：
```
org.apache.ibatis.binding.BindingException: 
Invalid bound statement (not found): com.lm.app.models.mapper.XxxMapper.selectPage
```

**根因分析**：
1. Mapper 接口上没有 `@Mapper` 注解
2. Mapper XML 文件路径与配置不匹配
3. Maven 编译时未将 XML 文件打包

**修复方案**：
```java
@Mapper  // 确认有此注解
public interface XxxMapper extends BaseMapperX<XxxDO> { }
```

检查 `application.yaml`：
```yaml
mybatis-plus:
  mapper-locations: classpath*:mapper/**/*.xml
```

### BadSqlGrammarException

**日志特征**：
```
BadSqlGrammarException: ### Error querying database. 
Cause: org.postgresql.util.PSQLException: ERROR: column "xxx" does not exist
```

**根因分析**：
1. DO 字段名与数据库字段不匹配
2. `@TableField` 映射错误
3. 数据库表结构未更新

**修复方案**：
```java
@TableField("user_name")  // 确认与数据库字段一致
private String username;
```

执行 Flyway 迁移或手动更新表结构。

## PostgreSQL 错误

### 连接失败

**日志特征**：
```
PSQLException: Connection to localhost:5432 refused
```

**修复方案**：
1. 检查 PostgreSQL 服务是否启动
2. 检查 `application.yaml` 数据库配置
3. 检查网络/防火墙

### 死锁

**日志特征**：
```
PSQLException: ERROR: deadlock detected
```

**修复方案**：
1. 检查事务中多个表更新顺序是否一致
2. 缩小事务范围
3. 添加重试机制

## 业务异常（错误码）

### 常见错误码速查

| 错误码 | 含义 | 排查方向 |
|--------|------|---------|
| 1_100_000_004 | 用户不存在 | 检查 userId 是否有效 |
| 1_100_001_000 | 外出记录不存在 | 检查记录 ID |
| 1_100_001_008 | 外出时间冲突 | 检查是否有重叠的外出记录 |
| 1_100_002_000 | 车辆不存在 | 检查 vehicleId |
| 1_100_002_005 | 车辆非空闲 | 检查车辆状态 |
| 1_100_003_000 | 吐槽内容不存在 | 检查 contentId |
| 1_100_007_000 | 公告不存在 | 检查 announcementId |
| 1_100_200_004 | 会话不存在 | 检查 conversationId |

## Nginx 错误

### 502 Bad Gateway

**排查方向**：
1. 后端服务是否启动
2. Nginx upstream 配置的后端端口是否正确
3. 后端服务是否健康

### 504 Gateway Timeout

**排查方向**：
1. 接口处理时间是否过长
2. 数据库查询是否慢
3. Nginx `proxy_read_timeout` 配置

## Docker 问题

### 容器启动失败

**排查命令**：
```bash
docker logs <container_id>
docker inspect <container_id>
```

### 端口冲突

**排查方向**：
```bash
netstat -tlnp | grep <port>
```
