---
name: doc-qa
description: Lingman-Starter 框架文档问答助手。当用户需要：(1) 查询某个 API 接口的用法和参数 (2) 了解框架某个功能的导入和使用方式 (3) 查询框架配置项的含义 (4) 询问开发规范相关问题 时触发此技能。(5) 查看框架底层代码 不要在以下场景触发：生成代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、涉及具体业务逻辑实现的问题。注意:所有项目都采用共同的框架---基于芋道源码二次开发而来的框架!
---

# 文档问答指南

## 知识范围

本 Skill 基于以下内容回答问题：
- `lingman-core/api-docs/` 下的 API 文档
- `lm` CLI 工具的安装、配置与使用
- 框架规范与配置说明

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

## CLI 工具问答

用户可能询问 `lm` CLI 工具相关问题，以下为常见问答：

### 安装

```bash
npm config set registry https://registry.npmmirror.com/
npm config set @lingman:registry=https://git.lingman.tech:8081/repository/npm_hosted/
npm install @lingman/cli -g
```

### 常用命令

| 命令 | 说明 | 运行目录 |
|------|------|---------|
| `lm init java` | 初始化项目配置，生成 `lingman.config.json` | 后端工程根目录 |
| `lm mapper` | 从数据库自动同步 DO 和 Mapper | 后端工程根目录 |
| `lm api` | 根据 Swagger 生成前端接口定义、入参与响应封装 | 前端工程根目录 |
| `lm u` | 更新 CLI 工具到最新版本 | 任意目录 |

**`lm mapper` 高级用法**：

```bash
# 定期使用：更新 DO/Mapper（推荐定期执行以同步数据库变更）
lm mapper -n
```

`-n` 参数强制重新生成，确保与数据库表结构保持同步。建议在数据库表结构变更后执行。

### 配置文件格式

项目根目录需存在 `lingman.config.json`：

```json
{
  "lang": "java",
  "template": "",
  "db": {
    "url": "jdbc:postgresql://<host>:<port>/<database>",
    "username": "<username>",
    "password": "<password>",
    "hasBase": false
  },
  "fileOverride": false
}
```

- `fileOverride`：控制是否覆盖已存在的文件。`true` — 覆盖更新；`false` — 不覆盖（默认）

数据库连接信息需根据项目 `application.yaml` 中的数据源配置替换。

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 所有 API 文档 | [lingman-core/api-docs/](../lingman-core/api-docs/) |
| API 索引 | [api-index.md](references/api-index.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
| CLI 工具安装与配置 | [README.md](../README.md) |
