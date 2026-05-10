---
name: error-analyzer
description: Lingman-Starter 框架错误日志分析助手。当用户需要：(1) 分析 Java 异常堆栈并定位根因 (2) 分析 Spring Boot / MyBatis Plus 框架报错 (3) 分析 PostgreSQL 数据库错误 (4) 分析 Nginx / Docker 部署问题 (5) 根据错误码定位业务错误 (6) 排查接口 400/500 错误 时触发此技能。不要在以下场景触发：生成代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、文档查询（由 doc-qa 处理）。
---

# 错误日志分析指南

## 分析范围

| 错误类型 | 示例 |
|---------|------|
| Java 运行时异常 | `NullPointerException`、`IllegalArgumentException`、`ClassCastException` |
| Spring Boot 启动异常 |  Bean 创建失败、循环依赖、配置缺失 |
| MyBatis Plus 异常 | SQL 语法错误、字段映射失败、Mapper 未找到 |
| PostgreSQL 错误 | 连接失败、表不存在、字段类型不匹配、死锁 |
| 业务异常 | 错误码抛出的业务逻辑异常 |
| Nginx 错误 | 502/504 网关超时、SSL 证书问题 |
| Docker 问题 | 容器启动失败、端口冲突、内存不足 |

## 分析步骤

1. **识别错误类型**：从日志中提取异常类名或错误码
2. **定位文件位置**：找到抛出异常的文件名和行号
3. **结合框架规范**：检查是否符合 lingman-starter 约定
4. **给出修复建议**：提供具体的代码修改或配置调整方案

## 错误码体系

项目使用分模块错误码，格式：`{模块前缀}_{序号}`

| 模块 | 错误码前缀 | 示例 |
|------|-----------|------|
| 注册/登录 | `1_100_000_XXX` | `USER_NOT_EXISTS`(1_100_000_004) |
| 外出报备 | `1_100_001_XXX` | `OUTBOUND_RECORD_NOT_EXISTS`(1_100_001_000) |
| 车辆派车 | `1_100_002_XXX` | `VEHICLE_NOT_EXISTS`(1_100_002_000) |
| 吐槽/表白 | `1_100_003_XXX` | `ROAST_CONTENT_NOT_EXISTS`(1_100_003_000) |
| 微信模板 | `1_100_004_XXX` | `WECHAT_MP_TEMPLATE_NOT_EXISTS`(1_100_004_000) |
| 公告 | `1_100_007_XXX` | `ANNOUNCEMENT_NOT_EXISTS`(1_100_007_000) |
| IM | `1_100_200_XXX` | `IM_CONVERSATION_NOT_EXISTS`(1_100_200_004) |

## 常见异常速查

### NullPointerException
- **排查方向**：检查 `@Resource` / `@Autowired` 注入是否成功、空值校验是否缺失
- **修复**：使用 `Objects.requireNonNull()` 或提前判空

### MyBatis Plus "Invalid bound statement"
- **排查方向**：Mapper 接口未被扫描、XML 文件位置错误
- **修复**：确认 Mapper 上有 `@Mapper` 注解，或检查 `@MapperScan` 配置

### PostgreSQL "relation does not exist"
- **排查方向**：表名错误、schema 不对、Flyway 迁移未执行
- **修复**：检查 `@TableName` 值，确认 Flyway 脚本已执行

### "TenantId cannot be null"
- **排查方向**：请求头缺少 tenant-id、未配置多租户上下文
- **修复**：检查请求是否携带租户标识，或接口是否标记 `@TenantIgnore`

### 502 Bad Gateway (Nginx)
- **排查方向**：后端服务未启动、端口配置错误、健康检查失败
- **修复**：检查服务状态、Nginx upstream 配置

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 常见错误模式 | [error-patterns.md](references/error-patterns.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
