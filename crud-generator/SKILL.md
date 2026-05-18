---
name: crud-generator
description: Lingman-Starter 框架 CRUD 代码生成助手。当用户需要：(1) 生成业务模块的 CRUD 代码（Controller/Service/VO/Mapper）(2) 基于已有数据库表/DO 生成增删改查代码骨架 (3) 新建业务模块 (4) 生成完整的业务代码模板 时触发此技能。不要在以下场景触发：从数据库生成 DO/Mapper（由 CLI 工具处理）、纯文档查询（由 doc-qa 处理）、SQL 语句生成（由 sql-generator 处理）。
---

# CRUD 代码生成指南

## 生成范围

本 Skill 负责生成业务模块的上层代码：

| 层 | 文件 | 是否生成 |
|---|------|---------|
| Controller | `{Name}Controller.java` | 是 |
| Service 接口 | `{Name}Service.java` | 是 |
| Service 实现 | `{Name}ServiceImpl.java` | 是 |
| VO | `{Name}SaveReqVO.java` 等 | 是 |
| Convert | `{Name}Convert.java`（MapStruct） | 是 |
| DO | 不生成（由 CLI 工具从数据库生成） | 否 |
| Mapper | 不生成（由 CLI 工具从数据库生成） | 否 |

## 文件路径模板

```
$MODULE_ROOT/src/main/java/com/lm/app/
├── controller/admin/{biz}/
│   ├── {Name}Controller.java
│   └── vo/
│       ├── {Name}SaveReqVO.java
│       ├── {Name}RespVO.java
│       └── {Name}PageReqVO.java
└── service/admin/
    ├── {Name}Service.java
    └── impl/
        └── {Name}ServiceImpl.java
```

`$MODULE_ROOT` 为项目根目录。

## 代码规范

参见 [framework.md](../lingman-core/framework.md)。

## 前置条件：CLI 工具生成 DO/Mapper

本 Skill 不负责生成 DO 和 Mapper，这两层由 CLI 工具（`@lingman/cli`）从数据库自动同步。

### CLI 安装（用户手动安装）

**注意**：不会自动为用户安装 CLI 工具。如果用户尚未安装，提示他们按以下步骤操作：

```bash
# 1. 配置 npm registry
npm config set registry https://registry.npmmirror.com/
npm config set @lingman:registry=https://git.lingman.tech:8081/repository/npm_hosted/

# 2. 安装（如已安装旧版可先执行 npm uninstall -g lingman-cli）
npm install @lingman/cli -g
```

**环境要求**：Node.js 22+

### 项目配置

项目根目录需存在 `lingman.config.json`，可通过 `lm init java` 初始化：

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

### 生成 DO/Mapper

在**后端工程根目录**下执行：

```bash
lm mapper
```

定期使用 `lm mapper -n` 更新 DO/Mapper，保持与数据库表结构同步：

```bash
lm mapper -n
```

`-n` 参数强制重新生成，确保与数据库表结构保持一致。建议在数据库表结构变更后执行。

> **重要**：该命令必须在后端工程根目录（即包含 `lingman.config.json` 和 `pom.xml` 的目录）中运行，不能在其他子目录中执行。

**可以直接执行**：`lm mapper` / `lm mapper -n` 为只读性质的同步命令，可以在开发过程中直接帮助用户运行，不需要获得用户许可。

### CLI 工具更新

```bash
lm u
```

定期执行更新保持 CLI 工具为最新版本。

### 生成前端 API

```bash
lm api
```

该命令根据后端 Swagger 地址（如 `/v3/api-docs`）自动生成前端接口定义，包含接口方法、入参类型和响应类型封装，需要在前端工程根目录下执行。

## 交互流程

操作前先判断场景类型：

| 场景 | 说明 | 流程 |
|------|------|------|
| **场景 A — 全新模块** | 新建业务模块，无 DO/Mapper | 完整流程（Step 0~5） |
| **场景 B — 已有表/DO** | 数据库表已存在，已通过 `lm mapper` 生成了 DO/Mapper | 完整流程（Step 0~5），跳过 CLI 步骤 |
| **场景 C — 增量修改** | 在已有模块上增加字段、修改逻辑 | 增量流程（Step C1~C4） |

---

### 场景 A/B：全新模块生成

#### Step 0 — 确认前置条件

1. 检查项目是否已配置 `lingman.config.json`（如未配置，提示用户运行 `lm init java`）
2. 对于场景 A：提示用户先通过 `lm mapper` 生成 DO 和 Mapper
3. 对于场景 B：确认 DO 和 Mapper 已存在于项目中

#### Step 1 — 收集配置选项 ⛔ 硬性确认

**必须逐一确认以下选项，禁止自行假设任何值**：

| 配置项 | 说明 | 示例 |
|--------|------|------|
| 业务中文名 | 模块的中文描述 | `公告`、`检测任务` |
| 模块包名 | URL 路径标识（kebab-case） | `announcement`、`detection-task` |
| 实体类名 | 驼峰大写 | `Announcement`、`DetectionTask` |
| 数据库表名 | 对应的表名 | `t_announcement` |
| ID 策略 | 序列名或自增 | `announcement_seq` |
| 是否多租户 | 影响 BaseDO vs TenantBaseDO | `true` / `false` |

#### Step 2 — 展示字段清单 ⛔ 硬性确认

根据 DO 字段，列出将生成的 VO 字段清单，标注哪些字段为必填、哪些支持模糊查询。

> **用户必须明确回复"确认"后，才能进入 Step 3。禁止在用户确认前生成任何代码。**

#### Step 3 — 展示文件清单 ⛔ 硬性确认

列出将要生成的全部文件路径，格式如下：

```
将生成以下文件：
- src/main/java/com/lm/app/controller/admin/{biz}/{Name}Controller.java
- src/main/java/com/lm/app/controller/admin/{biz}/vo/{Name}SaveReqVO.java
- src/main/java/com/lm/app/controller/admin/{biz}/vo/{Name}RespVO.java
- src/main/java/com/lm/app/controller/admin/{biz}/vo/{Name}PageReqVO.java
- src/main/java/com/lm/app/service/{biz}/{Name}Service.java
- src/main/java/com/lm/app/service/{biz}/impl/{Name}ServiceImpl.java
- src/main/java/com/lm/app/convert/{biz}/{Name}Convert.java
```

> **用户必须明确回复"确认"后，才能进入 Step 4。禁止在用户确认前生成任何代码。**  
> 如果用户回复"开始"、"生成"、"OK"等，都视为确认。

#### Step 4 — 执行生成

按以下顺序生成文件，同批次内的文件可并行写入：

1. VO 类（SaveReqVO → RespVO → PageReqVO）
2. Convert
3. Service 接口 + 实现
4. Controller

#### Step 5 — 输出清单

列出所有已生成的文件路径，并提示后续操作：

```
已生成以下文件：
- ... (列表)

后续操作建议：
1. 在 ErrorCodeConstants.java 中添加错误码
2. 如需权限控制，使用 permission-generator 生成权限配置
3. 如需生成建表 SQL，使用 sql-generator
```

---

### 场景 C：增量修改

#### Step C1 — 定位已有模块

确认要修改的模块名，读取该模块现有的全部文件（DO、VO、Service、Controller、Convert），了解当前结构。

#### Step C2 — 确认修改内容

确认要新增/修改/删除的字段或逻辑。

#### Step C3 — 展示变更清单 ⛔ 硬性确认

列出每个文件将发生的变更：

```
变更清单：
- {Name}SaveReqVO.java — 新增字段：{fieldName}（{类型}）
- {Name}RespVO.java — 新增字段：{fieldName}（{类型}）
- {Name}PageReqVO.java — 无变更
- {Name}Convert.java — 无变更（MapStruct 自动映射）
- {Name}ServiceImpl.java — page 方法新增查询条件：{fieldName}
```

> **用户必须明确回复"确认"后，才能进入 Step C4。**  
> 增量修改只改受影响的文件，禁止无关的格式化或重构。

#### Step C4 — 执行修改

只修改列出的文件，使用 `Edit` 工具进行精确替换，避免整文件重写。

---

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 框架分层、命名规范 | [framework.md](../lingman-core/framework.md) |
| 真实代码示例 | [crud-examples.md](references/crud-examples.md) |
| 各层代码模板 | [code-template.md](references/code-template.md) |

## 前端对接

前端开发规范详见 [frontend-spec.md](../lingman-core/frontend/frontend-spec.md)，核心要点：

| 后端输出 | 前端对接 | 规范 |
|----------|---------|------|
| `SaveReqVO` 字段 + `@Valid` 注解 | 表单弹窗 `XxxForm.vue` | `el-form` + `FormRules`，校验规则与后端注解保持一致 |
| `RespVO` 字段 | 表格列配置 `TableColumn[]` | 日期列用 `formatDate` / `dateFormatter`，字典列用 `<DictTag>` |
| `PageReqVO` 字段 | 搜索表单 `queryParams` | 初始值统一为 `undefined`，分页由 `useTable` 自动管理 |
| `CommonResult<T>` | API 响应处理 | 统一使用 `useMessage()` 或 `ElMessage` 进行成功/错误提示 |
| `ErrorCodeConstants` | 错误码映射 | 前端通过 `message.error()` 或 `ElMessageBox.confirm()` 处理 |

**API 生成**：`lm api` 根据后端 Swagger 自动生成前端 API 文件，存放于 `src/api/` 下，按模块分目录。手写扩展的 API 也统一放在 `src/api/` 下。生成后按对象导入：

```ts
import { ApiAppXxxAppAdminApiAuto } from '@/api/app/xxx'
const data = await ApiAppXxxAppAdminApiAuto.pageXxxAppAdminApi(queryParams)
```

**列表页**：使用 `useTable` hook 自动管理 `loading`、`total`、`tableList`、`pageSize`、`currentPage`：

```ts
const { register, tableObject, methods } = useTable({
  getListApi: (params) => ApiAppXxxAppAdminApiAuto.pageXxxAppAdminApi({ ...queryParams, ...params })
})
```

**组件/工具优先使用 `@lingman/yd`**：
- 组件：`ContentWrap`、`Table`、`Dialog`、`Icon`、`DictTag`、`Form`、`XButton` 等
- Hooks：`useTable`、`useMessage`、`useForm`、`useCrudSchemas`
- 工具：`formatDate`、`dateFormatter`、`download.excel`、`getDictOptions`

详见前端规范文档第 4 节（API 层规范）、第 5~6 节（列表页/表单弹窗规范）和第 13 节（`@lingman/yd` 插件规范）。
