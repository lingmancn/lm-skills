---
name: api-generator
description: Lingman-Starter 框架接口设计生成助手。当用户需要：(1) 根据业务需求设计 REST API 接口 (2) 生成接口文档（URL、请求参数、响应结构）(3) 设计复杂业务场景的接口契约 (4) 评审现有接口设计的合理性 (5) 生成前后端对接所需的接口定义 时触发此技能。不要在以下场景触发：生成完整 CRUD 代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、错误排查（由 error-analyzer 处理）。
---

# 接口设计生成指南

## 设计范围

本 Skill 负责生成接口设计文档，不生成具体代码实现：

| 输出 | 说明 |
|------|------|
| URL 设计 | RESTful 路径、HTTP 方法 |
| 请求参数 | Path/Query/Body 参数定义 |
| 响应结构 | 成功/失败返回格式 |
| 权限标识 | 接口所需权限点 |
| Swagger 注解 | `@Tag`、`@Operation` 等 |
| 前端接口定义 | `lm api` 生成的前端 API 封装 | 预留 |

## URL 设计规范

### 管理端接口
```
/app/{biz}/{action}

示例：
POST   /app/announcement/create    # 创建公告
POST   /app/announcement/update    # 更新公告
GET    /app/announcement/delete    # 删除公告
GET    /app/announcement/get       # 查询详情
POST   /app/announcement/page      # 分页查询
```

### 用户端（小程序/App）接口
```
/app/{biz}/{action}

示例：
POST   /app/announcement/list      # 列表查询（用户端）
GET    /app/announcement/detail    # 详情查询（用户端）
```

### HTTP 方法规范

| 方法 | 用途 |
|------|------|
| `GET` | 查询、删除（简单操作） |
| `POST` | 创建、更新、复杂查询（分页） |
| `PUT` | 全量更新（少用） |
| `DELETE` | 删除（少用，项目多用 GET） |

## 请求参数规范

### Path 参数
```java
@GetMapping("/get")
public CommonResult<AnnouncementRespVO> get(@RequestParam("id") Long id)
```

### Body 参数（创建/更新）
```java
@PostMapping("/create")
public CommonResult<Long> create(@Valid @RequestBody AnnouncementCreateReqVO reqVO)
```

### 分页查询
```java
@PostMapping("/page")
public CommonResult<PageResult<AnnouncementRespVO>> page(@RequestBody AnnouncementPageReqVO reqVO)
```

## 响应结构规范

### 成功响应
```json
{
  "code": 0,
  "data": { ... },
  "msg": ""
}
```

### 分页响应
```json
{
  "code": 0,
  "data": {
    "list": [ ... ],
    "total": 100
  },
  "msg": ""
}
```

### 失败响应
```json
{
  "code": 1_100_007_000,
  "data": null,
  "msg": "公告不存在"
}
```

## 接口设计模板

```
## 公告管理接口

### 1. 创建公告
- **URL**: POST /app/announcement/create
- **权限**: app:announcement:create
- **请求**: AnnouncementCreateReqVO
  - title (string, required): 公告标题
  - content (string, required): 公告内容
  - type (int): 公告类型：1通知 2公告
- **响应**: CommonResult<Long> (公告ID)

### 2. 更新公告
- **URL**: POST /app/announcement/update
- **权限**: app:announcement:update
- **请求**: AnnouncementUpdateReqVO
  - id (long, required): 公告编号
  - title (string, required): 公告标题
  - content (string, required): 公告内容
  - type (int): 公告类型
  - status (int): 状态
- **响应**: CommonResult<Boolean>

### 3. 删除公告
- **URL**: GET /app/announcement/delete
- **权限**: app:announcement:delete
- **请求**: id (long, query)
- **响应**: CommonResult<Boolean>

### 4. 查询详情
- **URL**: GET /app/announcement/get
- **权限**: app:announcement:query
- **请求**: id (long, query)
- **响应**: CommonResult<AnnouncementRespVO>

### 5. 分页查询
- **URL**: POST /app/announcement/page
- **权限**: app:announcement:query
- **请求**: AnnouncementPageReqVO
  - pageNo (int): 页码
  - pageSize (int): 每页条数
  - title (string): 标题（模糊查询）
  - type (int): 类型
  - status (int): 状态
- **响应**: CommonResult<PageResult<AnnouncementRespVO>>
```

## 配合 CLI 工具

设计接口前，建议先用 CLI 工具从数据库生成 DO/Mapper，了解已有的数据表结构，确保接口定义与数据模型一致。

### CLI 安装与使用

```bash
npm config set registry https://registry.npmmirror.com/
npm config set @lingman:registry=https://git.lingman.tech:8081/repository/npm_hosted/
npm install @lingman/cli -g
```

项目根目录需配置 `lingman.config.json`（通过 `lm init java` 初始化）：

```json
{
  "lang": "java",
  "template": "",
  "db": {
    "url": "jdbc:postgresql://<host>:<port>/<database>",
    "username": "<username>",
    "password": "<password>",
    "hasBase": false
  }
}
```

在**后端工程根目录**下运行 `lm mapper` 可自动同步 DO/Mapper；在**前端工程根目录**下运行 `lm api` 可根据 Swagger 生成前端接口定义。

> **重要**：两个命令都必须在对应工程根目录中运行，不能在其他子目录中执行。

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 接口设计示例 | [api-examples.md](references/api-examples.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
| 代码模板 | [code-template.md](../crud-generator/references/code-template.md) |
