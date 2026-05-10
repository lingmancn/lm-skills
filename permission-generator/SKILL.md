---
name: permission-generator
description: Lingman-Starter 框架权限配置生成助手。当用户需要：(1) 生成菜单权限 SQL (2) 生成按钮/接口权限标识 (3) 生成角色权限绑定 SQL (4) 生成数据权限配置代码 (5) 为新业务模块配置完整的权限体系 时触发此技能。不要在以下场景触发：生成业务 CRUD 代码（由 crud-generator 处理）、生成字典（由 dict-generator 处理）、文档查询（由 doc-qa 处理）。
---

# 权限配置生成指南

## 权限体系概述

lingman-starter 采用 RBAC + 数据权限模型：

| 权限类型 | 存储位置 | 配置方式 |
|---------|---------|---------|
| 菜单权限 | `system_menu` 表 | SQL 插入 |
| 按钮权限 | `system_menu` 表 (type=2) | SQL 插入 |
| 接口权限 | Controller `@PreAuthorize` | 代码注解 |
| 角色权限 | `system_role_menu` 表 | SQL 插入 |
| 数据权限 | `AppDataPermissionConfiguration` | Java 配置类 |

## 生成范围

本 Skill 负责生成：

1. **菜单权限 SQL** — `system_menu` 插入语句
2. **按钮权限 SQL** — 操作按钮权限记录
3. **接口权限标识** — `@PreAuthorize("@ss.hasPermission('xxx')")` 注解
4. **数据权限配置** — `DeptDataPermissionRuleCustomizer` 规则
5. **角色菜单绑定 SQL** — `system_role_menu` 关联语句

## 权限标识命名规范

```
{模块}:{功能}:{操作}

示例：
- app:announcement:query    # 公告查询
- app:announcement:create   # 公告创建
- app:announcement:update   # 公告更新
- app:announcement:delete   # 公告删除
```

## Controller 权限注解模板

```java
@PreAuthorize("@ss.hasPermission('app:announcement:query')")
public CommonResult<PageResult<AnnouncementRespVO>> page(...) { }

@PreAuthorize("@ss.hasPermission('app:announcement:create')")
public CommonResult<Long> create(...) { }

@PreAuthorize("@ss.hasPermission('app:announcement:update')")
public CommonResult<Boolean> update(...) { }

@PreAuthorize("@ss.hasPermission('app:announcement:delete')")
public CommonResult<Boolean> delete(...) { }
```

## 数据权限配置模板

```java
// 在 AppDataPermissionConfiguration 中添加
rule.addUserColumn({Entity}DO.class, "user_id");
rule.addDeptColumn({Entity}DO.class, "dept_id");
```

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 权限配置示例 | [permission-examples.md](references/permission-examples.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
| API 文档 | [api-index.md](../doc-qa/references/api-index.md) |
