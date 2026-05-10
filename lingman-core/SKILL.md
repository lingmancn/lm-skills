---
name: lingman-core
type: library
description: Lingman-Starter 共享知识层，包含框架规范、建表规范、API 文档、前端规范等。供所有 Skill 引用，不作为独立 Skill 触发。
hidden: true
---

# Lingman-Starter 共享知识层

本目录不是 Skill，是共享知识层，供以下 Skill 引用：

| Skill | 引用内容 |
|-------|---------|
| crud-generator | framework.md, code-template.md |
| sql-generator | db-spec.md |
| doc-qa | api-docs/, api-index.md |
| api-generator | framework.md |
| code-reviewer | framework.md |
| permission-generator | framework.md, permission-examples.md |
| dict-generator | framework.md |
| error-analyzer | framework.md |
| test-generator | framework.md |

## 目录

- [framework.md](framework.md) — 框架核心规范
- [db-spec.md](db-spec.md) — 数据库建表规范
- [api-docs/](api-docs/) — 模块 API 文档（20 个文档，10 个模块）
- [frontend/](frontend/) — 前端开发规范（待补充）
