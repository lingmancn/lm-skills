---
name: crud-generator
description: Lingman-Starter 框架 CRUD 代码生成助手。当用户需要：(1) 生成业务模块的 CRUD 代码（Controller/Service/VO/Mapper）(2) 基于已有数据库表/DO 生成增删改查代码骨架 (3) 新建业务模块 (4) 生成完整的业务代码模板 时触发此技能。不要在以下场景触发：生成 DO/Mapper（由开发者自行创建维护）、纯文档查询（由 doc-qa 处理）、SQL 语句生成（由 sql-generator 处理）。
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
| 错误码常量 | `ErrorCodeConstants.java` 追加 `{NAME}_NOT_EXISTS` | 是（追加，不新建文件） |
| DO | 不生成（由开发者自行创建维护） | 否 |
| Mapper | 不生成（由开发者自行创建维护） | 否 |

## 文件路径模板

```
$MODULE_ROOT/src/main/java/com/lm/app/
├── controller/admin/{biz}/
│   ├── {Name}Controller.java
│   └── vo/
│       ├── {Name}SaveReqVO.java
│       ├── {Name}RespVO.java
│       └── {Name}PageReqVO.java
├── service/{biz}/
│   ├── {Name}Service.java
│   └── {Name}ServiceImpl.java
└── convert/{biz}/
    └── {Name}Convert.java
```

`$MODULE_ROOT` 为项目根目录。

> 单 ID 删除/操作请求**统一使用** `controller/admin/common_vo/AdminDeleteReqVO.java`（全项目共享的公共类，不为各业务单独生成；若项目尚无该类需先新增，详见「后端接口强制规则」）。
>
> ServiceImpl 中引用的错误码常量 `{NAME}_NOT_EXISTS` 须**追加到项目已有的** `enums/ErrorCodeConstants.java`（不新建文件），并选用未被占用的错误码值；详见 [code-template.md](references/code-template.md)「ErrorCodeConstants 模板」。

## 代码规范

参见 [framework.md](../lingman-core/framework.md)。

### 后端接口强制规则

> 完整规范参见 [framework.md](../lingman-core/framework.md)「Controller 层规范」。以下为生成代码时必须逐条遵循的硬性规则。

- **接口路径不可重复**：禁止"同路径不同请求方式"（如 `/app/task` 同时存在 GET 与 POST）。全项目所有接口路径必须唯一；新增接口前须确认路径未被占用。
- **请求参数约定**：
  - GET 请求可用 `@RequestParam`、`@PathVariable` 传参；
  - **POST / PUT / DELETE / PATCH 等非 GET 请求一律使用 `@RequestBody` 传参**，禁止 `@RequestParam`、`@PathVariable`、URL 查询参数或路径参数。
- **单 ID 请求（删除、单 ID 操作等）统一规则**：
  - 字段名**统一为 `id`**，禁止使用业务前缀（如 `taskId`、`userId`、`xxxId`）。
  - **统一使用公共请求类 `AdminDeleteReqVO`**（路径 `controller/admin/common_vo/AdminDeleteReqVO.java`），**禁止为各业务模块单独生成 `{Name}DeleteReqVO` 等同义类**。
  - Controller 方法签名：`delete(@Valid @RequestBody AdminDeleteReqVO reqVO)`，通过 `reqVO.getId()` 取值。
  - 生成业务代码前，**先确认 `AdminDeleteReqVO` 是否已存在**；若项目尚无该类，必须先按 [code-template.md](references/code-template.md)「AdminDeleteReqVO 模板」新增该类，再继续生成业务代码。
- GET 查询/分页仍按框架规范使用 `@RequestParam` 或 `@Valid PageReqVO`，不受上述非 GET 规则约束。

## 前置条件：DO/Mapper 由开发者自行维护

本 Skill **不生成 DO 和 Mapper**，这两层由**开发者自行创建和维护**。生成上层代码（VO/Convert/Service/Controller）前，须确保目标 DO/Mapper 已就绪——本 Skill 会读取 DO 字段作为生成依据。

### DO/Mapper 编写要点

参照 [framework.md](../lingman-core/framework.md) 的 DO/Mapper 规范：

- **DO**：继承 `BaseDO`（多租户场景用 `TenantBaseDO`），加 `@TableName("t_xxx")`、`@KeySequence`、`@TableId`、`@TableField`，配合 Lombok `@Data` / `@Builder` / `@NoArgsConstructor` / `@AllArgsConstructor` / `@EqualsAndHashCode(callSuper = true)`。
- **Mapper**：继承 `BaseMapperX<{Name}DO>`，加 `@Mapper` 注解。

### 关于 `lm mapper`（可选，不推荐）

`@lingman/cli` 提供的 `lm mapper` 可从数据库自动同步 DO/Mapper，但**该命令不重要，开发中尽量不使用**——以开发者自行创建和维护为准。原因：自动生成可能覆盖手写内容、字段命名与注释不一定符合团队规范、还会引入额外的 CLI 配置成本。

仅在**全新数据库批量初始化**时才考虑使用，且生成后必须人工核对调整。如确需使用：

```bash
# 需先安装 @lingman/cli 并通过 lm init java 生成 lingman.config.json（详见 README.md）
lm mapper       # 同步生成
lm mapper -n    # 强制重新生成
```

> 该命令必须在后端工程根目录（含 `pom.xml`）运行；为只读同步性质，可不经许可直接运行，但生成结果**不保证可直接采用，须人工核对**。CLI 安装与配置详见 [README.md](../README.md)。

### CLI 工具更新

```bash
lm u
```

定期执行更新保持 CLI 工具为最新版本。

### 接口变动与前端 API 同步

只要后端接口发生变动（请求参数、响应参数的增删改），必须按以下流程联动同步，不得跳过：

> 区分：本节的 `lm api` 是 CLI 命令，从后端 Swagger 反向生成前端接口调用代码（工程同步动作）；`api-generator` skill 用于设计阶段产出接口设计文档，二者用途不同。

1. **重启后端服务**：确保 Swagger 文档（`/v3/api-docs`）反映最新接口。可由开发者手动重启，或由 AI 自动重启——**AI 自动重启前必须征得用户确认**。
2. **同步前端接口**：在前端工程根目录下执行 `lm api`，从后端 Swagger 地址自动生成前端接口定义（接口方法、入参/响应类型封装）。`lm api` 为只读同步命令，可直接执行无需额外许可。
3. **引用 API**：前端代码引用接口时，必须使用 `lm api` 同步生成的方法，统一从 `src/api/auto/` 导入；该目录由命令全量覆盖，**禁止手动修改**。

```bash
# 在前端工程根目录下执行
lm api
```

## 交互流程

操作前先判断场景类型：

| 场景 | 说明 | 流程 |
|------|------|------|
| **场景 A — 新建模块** | 新建业务模块；DO/Mapper 可能已由开发者自行创建，也可能尚不存在（需先提示开发者创建） | 完整流程（Step 0~5） |
| **场景 C — 增量修改** | 在已有模块上增加字段、修改逻辑 | 增量流程（Step C1~C4） |

---

### 场景 A：新建模块生成

#### Step 0 — 确认前置条件

1. 确认 DO 和 Mapper 已就绪：若项目中已存在（开发者已自行创建），直接进入下一步；若尚不存在，提示开发者先参照「DO/Mapper 编写要点」自行创建——本 Skill 不生成这两层
2. 确认公共单 ID 请求类 `controller/admin/common_vo/AdminDeleteReqVO.java` 已存在；若不存在，先按 [code-template.md](references/code-template.md)「AdminDeleteReqVO 模板」新增
3. 确认 `enums/ErrorCodeConstants.java` 已存在（通常项目已有该文件）；查清已用错误码值，避免与本次新增的 `{NAME}_NOT_EXISTS` 冲突

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

参照「文件路径模板」，将 `{biz}` / `{Name}` 替换为本次实际值，向用户逐条展示将要生成的完整文件路径（含 Controller、各 VO、Service/ServiceImpl、Convert）。

> **用户必须明确回复"确认"后，才能进入 Step 4。禁止在用户确认前生成任何代码。**  
> 如果用户回复"开始"、"生成"、"OK"等，都视为确认。

#### Step 4 — 执行生成

按以下顺序生成文件，同批次内的文件可并行写入：

1. VO 类（SaveReqVO → RespVO → PageReqVO）
2. Convert
3. 向 `enums/ErrorCodeConstants.java` 追加 `{NAME}_NOT_EXISTS` 常量（用 Edit 精确插入，错误码值须未被占用）
4. Service 接口 + 实现
5. Controller

#### Step 5 — 输出清单

列出所有已生成的文件路径（参照「文件路径模板」），并提示后续操作：

```
后续操作建议：
1. 如需权限控制，使用 permission-generator 生成权限配置
2. 如需生成建表 SQL，使用 sql-generator
3. 接口已变动，执行「接口变动与前端 API 同步」流程（重启后端 → 前端 `lm api`）
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

生成前端页面（index.vue、XxxForm.vue）时，必须严格遵守 `lingman-core/frontend/frontend-spec.md` 的完整规范。下方仅列出对生成代码骨架有强制影响的硬性约束；完整模板、Schema 示例、组件清单、字典/消息/工具函数使用规范等均以前端规范为准。

### 骨架级强制规则

1. **列表页必须使用 `useTable` hook 管理状态**，禁止手动维护 `loading` / `total` / `tableList`。
2. **列表页必须使用 `<Table>` 组件**，禁止直接使用 `<el-table>`。
3. **列表页搜索表单按复杂度选择实现**：搜索字段较多或为多列布局时**使用 `<Search>` 组件**（基于 `FormSchema`）；搜索字段仅 1 个、或需要 `<Search>` 无法表达的自定义样式/联动时，**回退使用原生 `el-form`**。
4. **搜索区与表格区必须分别包裹在 `<ContentWrap>` 中**。
5. **表单弹窗（新增/编辑）必须使用 `<Dialog>` + `<Form>` 组件**，禁止直接使用 `<el-dialog>` / `<el-form>`。
6. **优先使用 `@lingman/yd` 提供的组件、Hooks 和工具函数**，已有能力禁止重复封装。
7. **API 文件优先引用 `src/api/auto/` 下的自动生成对象**（`import { ApiAppXxxAppAdminApiAuto }`）；仅当需要手写扩展时才使用 `src/api/` 下的自定义路径。

### 生成顺序

以前端规范的「列表页规范」和「表单弹窗规范」章节为骨架，根据后端 VO 字段填充：
- `index.vue`：根据 `RespVO` 生成 `columns`；根据 `PageReqVO` 生成 `searchSchema`（字段多于 1 个时用 `<Search>`）；日期列用 `formatDate` formatter；状态/字典列预留插槽用 `<DictTag>`。
- `XxxForm.vue`：根据 `SaveReqVO` 生成 `FormSchema[]` 和 `FormRules`。
- **代码注释**：生成的前端代码中，复杂逻辑方法必须添加中文注释说明"为什么这么做"；不易理解的条件渲染、嵌套插槽、特殊数据处理必须添加行内注释。

### 参考

| 场景 | 参考文档 |
|------|----------|
| 完整前端规范（列表页/表单弹窗/Search 组件/字典/消息/@lingman/yd 全量组件清单） | [frontend-spec.md](../lingman-core/frontend/frontend-spec.md) |
