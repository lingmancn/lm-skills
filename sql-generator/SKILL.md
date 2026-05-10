---
name: sql-generator
description: Lingman-Starter 框架 SQL 生成助手。当用户需要：(1) 生成建表 SQL（CREATE TABLE）(2) 生成改表 SQL（ALTER TABLE）(3) 生成统计查询 SQL (4) 生成 MyBatis Mapper XML 中的 SQL 片段 (5) 生成数据库迁移脚本 时触发此技能。不要在以下场景触发：生成 Java 代码类（由 crud-generator 处理）、纯文档查询（由 doc-qa 处理）。
---

# SQL 生成指南

## 生成范围

本 Skill 负责生成数据库相关的 SQL 语句：

| SQL 类型 | 示例 |
|---------|------|
| DDL - 建表 | `CREATE TABLE t_xxx (...)` |
| DDL - 改表 | `ALTER TABLE t_xxx ADD COLUMN ...` |
| DML - 查询 | `SELECT ... FROM ... WHERE ...` |
| DML - 统计 | 聚合查询、分组统计 |
| Mapper XML | MyBatis 的 `<select>` / `<update>` 片段 |

## 建表规范

参见 [db-spec.md](../lingman-core/db-spec.md)。

## 生成步骤

1. 用户描述业务实体和字段需求
2. 生成符合规范的建表 SQL（含必备字段）
3. 生成对应的索引
4. 如有需要，生成示例查询 SQL

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 建表规范、字段约定 | [db-spec.md](../lingman-core/db-spec.md) |
| 常见 SQL 示例 | [sql-examples.md](references/sql-examples.md) |
