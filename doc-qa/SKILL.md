---
name: doc-qa
description: Lingman-Starter 框架文档问答助手。当用户需要：(1) 查询某个 API 接口的用法和参数 (2) 了解框架某个功能的导入和使用方式 (3) 查询框架配置项的含义 (4) 询问开发规范相关问题 时触发此技能。不要在以下场景触发：生成代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、涉及具体业务逻辑实现的问题。
---

# 文档问答指南

## 知识范围

本 Skill 基于 `lingman-core/api-docs/` 下的 API 文档回答框架使用问题。

## 回答原则

1. **直接回答**：不绕弯子，直接给出答案
2. **包含代码示例**：尽可能提供代码片段
3. **超出范围说明**：如果问题超出已有文档范围，明确说明"文档未覆盖该场景"
4. **引用来源**：回答时引用具体文档名称

## 典型问题

- "AdminUserApi 的 getUser 方法怎么用？"
- "如何导入 DictDataApi？"
- "PermissionApi 的 hasAnyPermissions 返回什么？"
- "DeptApi 的 validateDeptList 什么时候抛异常？"

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 所有 API 文档 | [lingman-core/api-docs/](../lingman-core/api-docs/) |
| API 索引 | [api-index.md](references/api-index.md) |
