---
name: lingman-core
type: library
description: Lingman-Starter 共享知识层，包含框架规范、建表规范、API 文档、管理后台前端规范、uni-app 开发与工具链规范等。供所有 Skill 引用，不作为独立 Skill 触发。
hidden: true
---

# Lingman-Starter 共享知识层

本目录不是 Skill，是共享知识层，供以下 Skill 引用：

| Skill | 引用内容 |
|-------|---------|
| crud-generator | framework.md, code-template.md, frontend/frontend-spec.md |
| uni-app-feature | uni-app/uni-app-spec.md（p708-verified-patterns.md 仅 p708 或极相似项目按需） |
| uni-app-tooling | uni-app/tooling-guide.md, uni-app/uni-app-spec.md（p708-verified-patterns.md 仅 p708 或极相似项目按需） |
| sql-generator | db-spec.md |
| doc-qa | api-docs/, api-index.md, uni-app/ |
| api-generator | framework.md |
| code-reviewer | framework.md, frontend/frontend-spec.md, uni-app/uni-app-spec.md |
| permission-generator | framework.md, permission-examples.md |
| dict-generator | framework.md |
| error-analyzer | framework.md |
| test-generator | framework.md |
| project-md-generator | framework.md, uni-app/uni-app-spec.md |

## 目录

- [framework.md](framework.md) — 框架核心规范
- [db-spec.md](db-spec.md) — 数据库建表规范
- [api-docs/](api-docs/) — 模块 API 文档（20 个文档，10 个模块）
- [frontend/](frontend/) — 管理后台 Web 开发规范（frontend-spec.md，列表页/表单弹窗/Search/@lingman/yd 等）
- [uni-app/](uni-app/) — uni-app 通用开发规范、p708 已验证模式和工具链排障指南
