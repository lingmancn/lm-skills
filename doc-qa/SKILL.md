---
name: doc-qa
description: Lingman-Starter 多端框架文档问答助手。当用户需要：(1) 查询 API 接口的用法和参数 (2) 了解框架功能的导入和使用方式 (3) 查询框架或 uni-app 配置项的含义 (4) 询问后端、管理后台 Web、uni-app 开发规范 (5) 查看框架底层代码或理解 p708 已验证工程模式 时触发。只回答规范、配置、机制和原因；生成代码转对应生成 Skill，uni-app 页面/组件实现转 uni-app-feature，启动/环境/代理/401/编译构建排障转 uni-app-tooling。
---

# 文档问答指南

## 知识范围

本 Skill 基于以下内容回答问题：
- `lingman-core/api-docs/` 下的 API 文档
- `lm` CLI 工具的安装、配置与使用
- 后端框架和管理后台 Web 规范
- [uni-app-spec.md](../lingman-core/uni-app/uni-app-spec.md)：uni-app 通用页面、状态、样式、跨端和资源生命周期规范
- [tooling-guide.md](../lingman-core/uni-app/tooling-guide.md)：环境、代理、请求认证、配置生成和构建排障方法
- [p708-verified-patterns.md](../lingman-core/uni-app/p708-verified-patterns.md)：p708 已验证模式索引；回答前仍需以目标项目当前文件为准

## 问答与实施分流

| 用户目标 | 处理方式 |
|---|---|
| 查询规范、配置含义、机制、为什么这样设计 | 本 Skill 回答并引用来源 |
| 查询 p708 页面生成、请求桥接、TabBar、主题或视频模式 | 先基于已验证模式回答；涉及当前值时读取目标项目源文件 |
| 新建/修改 uni-app 页面、组件、Pinia、样式或视频业务 | 转 `uni-app-feature` |
| 启动、env、代理、localhost、401、`lm api`、编译或构建排障 | 转 `uni-app-tooling` |
| 后端或管理后台 CRUD 代码生成 | 转 `crud-generator` |
| 纯代码审查 | 转 `code-reviewer` |

纯问答不得顺带启动项目、修改业务代码或把文档中的示例当作当前项目事实。若问题依赖版本、script、环境地址、页面/API 清单或生成配置，先读取当前项目；不得引用或回显密钥、Token、AppID、证书和签名。

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

**`lm mapper`（可选，不推荐日常使用）**：

> DO/Mapper 以开发者自行创建维护为准；`lm mapper` 仅在全新库批量初始化时可考虑，且生成后须人工核对。

```bash
lm mapper -n   # 强制重新生成
```

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
| 管理后台 Web 规范 | [frontend-spec.md](../lingman-core/frontend/frontend-spec.md) |
| uni-app 通用规范 | [uni-app-spec.md](../lingman-core/uni-app/uni-app-spec.md) |
| uni-app 工具链指南 | [tooling-guide.md](../lingman-core/uni-app/tooling-guide.md) |
| p708 已验证模式 | [p708-verified-patterns.md](../lingman-core/uni-app/p708-verified-patterns.md) |
| CLI 工具安装与配置 | [README.md](../README.md) |
