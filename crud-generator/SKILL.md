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
│       ├── {Name}PageReqVO.java
│       └── {Name}DeleteReqVO.java  # 仅当项目没有公共删除/ID 请求 VO 时生成
└── service/admin/
    ├── {Name}Service.java
    └── impl/
        └── {Name}ServiceImpl.java
```

`$MODULE_ROOT` 为项目根目录。

## 代码规范

参见 [framework.md](../lingman-core/framework.md)。

### 后端接口强制规则

- DELETE 请求禁止使用 `@RequestParam`、`@PathVariable` 或 URL 查询参数/路径参数传参。
- DELETE 请求参数必须使用 `@RequestBody` 承载，通过 `deleteReqVO.getId()` 获取编号。
- 单 ID 删除优先复用项目已有的公共删除/ID 请求 VO（如 `IdReqVO`、`DeleteReqVO`、`BaseIdReqVO` 等）；如果项目不存在可复用的公共 VO，才生成业务专属 `{Name}DeleteReqVO`。
- DELETE Controller 方法必须使用 `@Valid @RequestBody {DeleteReqVO} deleteReqVO`，其中 `{DeleteReqVO}` 代表已存在的公共 VO 或兜底生成的 `{Name}DeleteReqVO`。
- GET 查询/分页仍按框架规范使用 `@RequestParam` 或 `@Valid PageReqVO`，不要套用 DELETE 规则。

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

1. VO 类（SaveReqVO → RespVO → PageReqVO；若项目没有公共删除/ID 请求 VO，则额外生成 DeleteReqVO）
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

本 Skill 生成前端页面时，必须严格遵守以下规范。所有规范来源于 `lingman-core/frontend/frontend-spec.md`，此处内联完整模板以保证生效。

### 强制规则

1. **列表页必须使用 `useTable` hook 管理状态**，禁止手动维护 `loading`/`total`/`tableList`。
2. **列表页必须使用 `<Table>` 组件**，禁止直接使用 `<el-table>`。
3. **列表页搜索区优先使用 `<Search>` 组件**（基于 Schema），只有搜索项为单个或极少数、以及 `<Search>` 无法实现的自定义样式时，才回退到原生 `el-form`。
4. **列表页搜索区和表格区必须分别包裹在 `<ContentWrap>` 中**。
5. **表单弹窗必须使用 `<Dialog>` 组件**，禁止直接使用 `<el-dialog>`。
6. **优先使用 `@lingman/yd` 提供的组件和 Hooks**，已有功能禁止重复封装。
7. **API 文件优先引用 `src/api/auto/` 下的自动生成对象**；仅当手写扩展时才使用 `src/api/` 下的自定义路径。

### 列表页标准模板（index.vue）

```vue
<template>
  <!-- 搜索区域 -->
  <ContentWrap>
    <Search :schema="searchSchema" :model="queryParams" @search="handleQuery" @reset="handleReset">
      <template #actionMore>
        <el-button type="primary" @click="handleCreate">
          <Icon class="mr-5px" icon="ep:plus" />新增
        </el-button>
      </template>
    </Search>
  </ContentWrap>

  <!-- 列表区域 -->
  <ContentWrap>
    <Table
      :columns="columns"
      border
      :data="tableObject.tableList"
      :loading="tableObject.loading"
      :pagination="{ total: tableObject.total }"
      v-model:pageSize="tableObject.pageSize"
      v-model:currentPage="tableObject.currentPage"
      @register="register"
    >
      <!-- 自定义列：slot name 为列的 field 值 -->
      <template #status="{ row }">
        <DictTag :type="DICT_TYPE.COMMON_STATUS" :value="row.status" />
      </template>
      <!-- 操作列 -->
      <template #action="{ row }">
        <el-button type="primary" @click="handleEdit(row)">编辑</el-button>
        <el-button type="danger" @click="handleDelete(row)">删除</el-button>
      </template>
    </Table>
  </ContentWrap>

  <!-- 表单弹窗 -->
  <XxxForm ref="formRef" @success="methods.getList" />
</template>

<script lang="ts" setup>
import { reactive, ref, onMounted } from 'vue'
import type { TableColumn } from '@/types/table'
import type { FormSchema } from '@/types/form'
import XxxForm from './XxxForm.vue'

// 1. API 导入（优先使用 auto 目录的自动生成文件）
import { ApiAppXxxAppAdminApiAuto } from '@/api/auto/app/xxx'

// 2. @lingman/yd 工具导入
import { formatDate, DICT_TYPE } from '@lingman/yd'

// 3. 组件名声明
defineOptions({ name: 'Xxx' })

// 4. 搜索表单配置（基于 Schema，由 PageReqVO 字段决定）
const searchSchema = reactive<FormSchema[]>([
  { field: 'name', label: '名称', component: 'Input', componentProps: { placeholder: '请输入名称', clearable: true } },
  { field: 'status', label: '状态', component: 'Select', componentProps: { placeholder: '请选择状态', clearable: true, options: getIntDictOptions(DICT_TYPE.COMMON_STATUS) } }
])

// 5. 表格列定义（静态，置于响应式状态之前）
const columns: TableColumn[] = [
  { field: 'name', label: '名称', minWidth: 150 },
  { field: 'status', label: '状态', width: 100 },
  { field: 'createTime', label: '创建时间', width: 170, formatter: (_row, _col, val) => val ? formatDate(val) : '' },
  { field: 'action', label: '操作', width: 200, fixed: 'right' }
]

// 6. 搜索参数（初始值统一为 undefined）
const queryParams = reactive({
  name: undefined as string | undefined,
  status: undefined as number | undefined
})

const formRef = ref()

// 7. useTable hook：自动管理 loading、total、list、分页
const { register, tableObject, methods } = useTable({
  getListApi: (params) =>
    ApiAppXxxAppAdminApiAuto.pageXxxAppAdminApi({
      ...queryParams,
      ...params
    })
})
methods.getList()

// 8. 搜索与重置
const handleQuery = () => {
  methods.setSearchParams({ ...queryParams })
}

const handleReset = () => {
  queryParams.name = undefined
  queryParams.status = undefined
  methods.setSearchParams({})
}

// 9. 新增/编辑
const handleCreate = () => {
  formRef.value?.open('create')
}
const handleEdit = (row: any) => {
  formRef.value?.open('update', row)
}

// 10. 删除（先弹确认框，删除后刷新列表）
const handleDelete = async (row: any) => {
  await message.delConfirm('确认删除该记录吗？')
  await ApiAppXxxAppAdminApiAuto.deleteXxxAppAdminApi({ id: row.id })
  message.success('删除成功')
  // 重新拉取列表，确保数据最新
  methods.getList()
}

onMounted(() => {
  // 加载字典、选项等初始化逻辑
})
</script>
```

### 表单弹窗标准模板（XxxForm.vue）

> **说明**：新增/编辑表单**必须使用 `<Form>` 组件**（基于 Schema 的动态表单）。列表页搜索表单**优先使用 `<Search>` 组件**，仅当搜索项极少或自定义样式难以实现时才使用原生 `el-form`。

```vue
<template>
  <Dialog v-model="visible" :title="title" width="680px">
    <Form ref="formRef" :schema="schema" :rules="rules" label-width="90px" />
    <template #footer>
      <el-button @click="visible = false">取 消</el-button>
      <el-button type="primary" :loading="submitting" @click="submitForm">确 定</el-button>
    </template>
  </Dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
import type { FormRules } from 'element-plus'
import type { FormSchema } from '@/types/form'
import type { FormExpose } from '@/components/Form'
import { ApiAppXxxAppAdminApiAuto } from '@/api/auto/app/xxx'

defineOptions({ name: 'XxxForm' })

const emits = defineEmits(['success'])

const visible = ref(false)
const title = ref('')
const formType = ref<'create' | 'update'>('create')
const submitting = ref(false)
const formRef = ref<FormExpose>()

const schema = reactive<FormSchema[]>([
  { field: 'name', label: '名称', component: 'Input' },
  { field: 'code', label: '编码', component: 'Input' },
  { field: 'status', label: '状态', component: 'Radio', componentProps: { options: [{ label: '启用', value: 1 }, { label: '禁用', value: 0 }] } },
  { field: 'remark', label: '备注', component: 'Input', componentProps: { type: 'textarea', rows: 3 } }
])

const rules = reactive<FormRules>({
  name: [{ required: true, message: '名称不能为空', trigger: 'blur' }],
  code: [{ required: true, message: '编码不能为空', trigger: 'blur' }]
})

const open = (type: 'create' | 'update', row?: any) => {
  visible.value = true
  formType.value = type
  title.value = type === 'create' ? '新增' : '修改'
  resetForm()
  if (row) {
    formRef.value?.setValues(row)
  }
}
defineExpose({ open })

// 提交表单：手动校验 → 根据 formType 调用新增或修改接口
const submitForm = async () => {
  const elForm = formRef.value?.getElFormRef()
  if (!elForm) return
  await elForm.validate(async (valid) => {
    if (!valid) return
    submitting.value = true
    try {
      const data = formRef.value?.formModel
      // 根据表单类型调用不同接口：create → 新增, update → 修改
      if (formType.value === 'create') {
        await ApiAppXxxAppAdminApiAuto.createXxxAppAdminApi(data)
        message.success('新增成功')
      } else {
        await ApiAppXxxAppAdminApiAuto.updateXxxAppAdminApi(data)
        message.success('修改成功')
      }
      visible.value = false
      emits('success')
    } finally {
      submitting.value = false
    }
  })
}

const resetForm = () => {
  formRef.value?.setValues({ id: undefined, name: '', code: '', status: 1, remark: '' })
}
</script>
```

### 关键组件与 Hooks 速查表

| 场景 | 必须使用的 `@lingman/yd` 能力 |
|------|------------------------------|
| 列表页状态管理 | `useTable({ getListApi })` |
| 列表表格 | `<Table :columns :data :loading :pagination>` |
| 搜索表单 | `<Search :schema :model @search @reset>` |
| 内容区域包裹 | `<ContentWrap>` |
| 弹窗 | `<Dialog>` |
| 表单（新增/编辑） | `<Form :schema :rules>` |
| 消息提示 | `useMessage()`（`message.success` / `message.delConfirm` / `message.confirm`） |
| 日期格式化 | `formatDate` / `dateFormatter` |
| 字典标签 | `<DictTag>` |
| 图标 | `<Icon icon="ep:xxx" />` |

### 前端生成规则

1. **模板优先**：生成前端代码时，必须以上述"列表页标准模板"和"表单弹窗标准模板"为基础骨架，只修改业务相关的字段和逻辑。
2. **表格列动态生成**：根据 `RespVO` 字段生成 `TableColumn[]`；日期列使用 `formatter: (_row, _col, val) => val ? formatDate(val) : ''`；状态/字典列预留 `#status` 插槽使用 `<DictTag>`；操作列固定为 `{ field: 'action', label: '操作', width: 200, fixed: 'right' }`。
3. **搜索表单动态生成**：列表页搜索区**优先使用 `<Search>` 组件**（基于 `FormSchema`），根据 `PageReqVO` 字段生成 `searchSchema`，初始值统一为 `undefined`；仅当搜索项极少（1 个）或自定义样式 `<Search>` 无法实现时，才回退到原生 `el-form`。
4. **表单弹窗动态生成**：新增/编辑弹窗使用 `<Form>` 组件，根据 `SaveReqVO` 字段生成 `FormSchema[]` 和 `FormRules`。
5. **API 路径**：优先使用 `@/api/auto/app/xxx` 的自动生成 API 对象；仅当用户明确要求手写 API 时才使用 `@/api/xxx` 自定义路径。
6. **代码注释**：生成的前端代码中，复杂逻辑方法必须添加注释说明其用途和关键步骤；不易理解的模板片段（如条件渲染、嵌套插槽、特殊数据处理）必须添加行内注释。注释使用中文，简洁明了，说明"为什么这么做"而非"做了什么"。

---

## Search 组件使用规范

### 基础用法

`<Search>` 是基于 `FormSchema` 的搜索表单组件，由 `@lingman/yd` 提供。它与 `<Form>` 共用同一套 `FormSchema` 类型，但专为列表页搜索场景封装了**搜索**、**重置**按钮及布局。

```vue
<Search
  :schema="searchSchema"
  :model="queryParams"
  @search="handleQuery"
  @reset="handleReset"
>
  <template #actionMore>
    <el-button type="primary" @click="handleCreate">
      <Icon class="mr-5px" icon="ep:plus" />新增
    </el-button>
  </template>
</Search>
```

### 常用 Props

| Prop | 说明 | 类型 | 默认值 |
|------|------|------|--------|
| `schema` | 搜索项 Schema 配置 | `FormSchema[]` | `[]` |
| `model` | 搜索参数对象（双向绑定） | `Recordable` | `{}` |
| `inline` | 是否行内表单 | `boolean` | `true` |
| `isCol` | 是否使用栅格布局 | `boolean` | `true` |
| `expand` | 是否支持展开/收起 | `boolean` | `false` |
| `showSearch` | 是否显示搜索按钮 | `boolean` | `true` |
| `showReset` | 是否显示重置按钮 | `boolean` | `true` |
| `labelWidth` | 标签宽度 | `string / number` | `120px` |
| `buttomPosition` | 按钮位置 | `string` | — |

### 常用 Slots

| Slot 名 | 说明 |
|---------|------|
| `actionMore` | 搜索/重置按钮右侧的额外操作区（如“新增”按钮） |

### 搜索项 Schema 示例

```ts
import type { FormSchema } from '@/types/form'

const searchSchema = reactive<FormSchema[]>([
  {
    field: 'name',
    label: '名称',
    component: 'Input',
    componentProps: { placeholder: '请输入名称', clearable: true }
  },
  {
    field: 'status',
    label: '状态',
    component: 'Select',
    componentProps: {
      placeholder: '请选择状态',
      clearable: true,
      class: '!w-160px',
      options: getIntDictOptions(DICT_TYPE.COMMON_STATUS)
    }
  },
  {
    field: 'createTime',
    label: '创建时间',
    component: 'DatePicker',
    componentProps: {
      type: 'daterange',
      valueFormat: 'YYYY-MM-DD',
      startPlaceholder: '开始日期',
      endPlaceholder: '结束日期',
      class: '!w-240px'
    }
  }
])
```

### 使用注意事项

1. **搜索参数初始值**：`queryParams` 的每个字段初始值统一为 `undefined`，重置时手动将各字段重置为 `undefined`（而非调用 `resetFields`）。
2. **事件绑定**：必须绑定 `@search="handleQuery"` 和 `@reset="handleReset"`，分别触发查询和重置逻辑。
3. **查询逻辑**：`handleQuery` 内部调用 `methods.setSearchParams({ ...queryParams })`，由 `useTable` 驱动重新拉取列表。
4. **新增按钮位置**：新增/导出等操作按钮放在 `<template #actionMore>` 插槽中，与搜索/重置按钮保持在同一行。
5. **组件宽度**：`Select`、`DatePicker` 等表单组件在搜索区必须设置最小宽度，避免伸缩变形。通过 `componentProps: { class: '!w-xxxpx' }` 控制，例如 `!w-160px`（下拉框）、`!w-240px`（日期范围）。
6. **回退条件**：仅当搜索条件只有 **1 个** 或自定义样式（如特殊布局、联动交互）`<Search>` 无法满足时，才允许使用原生 `el-form`。

---

## 字典使用规范

### 字典组件与工具

| 场景 | 推荐用法 | 来源 |
|------|----------|------|
| 表格中显示字典标签 | `<DictTag :type="DICT_TYPE.XXX" :value="row.status" />` | `@lingman/yd` |
| 表单下拉选项（整数型） | `getIntDictOptions(DICT_TYPE.XXX)` | `@lingman/yd` |
| 表单下拉选项（字符串型） | `getStrDictOptions(DICT_TYPE.XXX)` | `@lingman/yd` |
| 表单下拉选项（布尔型） | `getBoolDictOptions(DICT_TYPE.XXX)` | `@lingman/yd` |
| 根据值取标签 | `getDictLabel(DICT_TYPE.XXX, value)` | `@lingman/yd` |

### 使用示例

**表格列渲染字典标签：**

```vue
<template #status="{ row }">
  <DictTag :type="DICT_TYPE.COMMON_STATUS" :value="row.status" />
</template>
```

**搜索表单下拉框绑定字典：**

```vue
<el-select v-model="queryParams.status" clearable placeholder="请选择状态">
  <el-option
    v-for="dict in getIntDictOptions(DICT_TYPE.COMMON_STATUS)"
    :key="dict.value"
    :label="dict.label"
    :value="dict.value"
  />
</el-select>
```

**表单弹窗中使用字典（Form 组件）：**

```ts
const schema = reactive<FormSchema[]>([
  {
    field: 'status',
    label: '状态',
    component: 'Select',
    componentProps: {
      options: getIntDictOptions(DICT_TYPE.COMMON_STATUS)
    }
  }
])
```

### 关于 DICT_TYPE 的注意事项

- `@lingman/yd` 插件内置了部分通用字典类型（如 `DICT_TYPE.COMMON_STATUS`）。
- **每个项目的业务字典类型不一致**，项目中通常会在 `src/utils/dict.ts` 中定义自己的 `DICT_TYPE` 常量或枚举。
- 生成代码时，**优先使用项目自身定义的 `DICT_TYPE`**；若项目未定义，再使用插件内置的通用字典。
- 不要在生成的代码中硬编码字典值（如 `0` / `1`），应始终通过 `DICT_TYPE` 引用。

---

## 公共方法使用规范

以下公共方法来自 `@lingman/yd`，CRUD 页面中**禁止重复封装**，必须直接使用。

### 日期时间方法

| 方法 | 用途 | 典型使用场景 |
|------|------|-------------|
| `formatDate(date)` | 格式化为 `YYYY-MM-DD HH:mm:ss` | 表格列 formatter、表单回显 |
| `dateFormatter(row, col, val)` | 表格日期列 formatter（含时分秒） | `TableColumn.formatter` |
| `dateFormatter2(row, col, val)` | 表格日期列 formatter（仅日期） | `TableColumn.formatter` |
| `formatToDate(date)` | 格式化为 `YYYY-MM-DD` | 日期展示 |
| `formatToDateTime(date)` | 格式化为 `YYYY-MM-DD HH:mm:ss` | 日期时间展示 |
| `dateUtil(date)` | dayjs 实例 | 复杂日期计算 |
| `beginOfDay(date)` | 当天开始时间 | 日期范围查询参数处理 |
| `endOfDay(date)` | 当天结束时间 | 日期范围查询参数处理 |
| `defaultShortcuts` | el-date-picker 快捷选项 | 日期选择器快捷配置 |

```ts
// 表格列日期格式化示例
const columns: TableColumn[] = [
  { field: 'createTime', label: '创建时间', width: 170, formatter: (_row, _col, val) => val ? formatDate(val) : '' }
]
```

### 消息提示方法（`useMessage()`）

| 方法 | 用途 |
|------|------|
| `message.success('操作成功')` | 成功提示 |
| `message.error('操作失败')` | 错误提示 |
| `message.warning('警告信息')` | 警告提示 |
| `message.delConfirm('确认删除该记录吗？')` | 删除二次确认（返回 Promise） |
| `message.confirm('确认要执行该操作吗?', '提示')` | 通用确认弹窗 |
| `message.exportConfirm()` | 导出二次确认 |
| `message.prompt('请输入内容', '提示')` | 输入框弹窗 |

```ts
// 删除操作示例
const handleDelete = async (row: any) => {
  await message.delConfirm('确认删除该记录吗？')
  await ApiAppXxxAppAdminApiAuto.deleteXxxAppAdminApi({ id: row.id })
  message.success('删除成功')
  methods.getList()
}
```

### 表单校验方法（`useValidator()`）

| 方法 | 用途 |
|------|------|
| `required()` | 必填校验 |
| `lengthRange(min, max)` | 长度范围校验 |
| `notSpace()` | 不允许空格 |

```ts
import { useValidator } from '@lingman/yd'
const { required } = useValidator()

const rules = reactive<FormRules>({
  name: [required()],
  code: [required(), lengthRange(2, 30)]
})
```

### 下载方法

| 方法 | 用途 |
|------|------|
| `download.excel(data, filename)` | 下载 Excel |
| `download.word(data, filename)` | 下载 Word |
| `download.zip(data, filename)` | 下载 Zip |
| `download.json(data, filename)` | 下载 JSON |

```ts
import { download } from '@lingman/yd'

// 导出按钮示例
const handleExport = async () => {
  await message.exportConfirm()
  const res = await ApiAppXxxAppAdminApiAuto.exportXxxAppAdminApi(queryParams)
  download.excel(res, 'xxx_list.xlsx')
}
```

### 通用工具方法

| 方法 | 用途 | 使用场景 |
|------|------|----------|
| `generateUUID()` | 生成 UUID | 需要临时唯一标识时 |
| `generateRandomStr(length)` | 生成随机字符串 | 随机码、临时密钥 |
| `copyValueToTarget(target, source)` | 按目标对象属性复制 | 表单数据拷贝 |
| `buildSortingField({ prop, order })` | 构建排序字段 | 表格排序参数处理 |
| `trim(str)` | 去除首尾空格 | 输入预处理 |
| `jsonParse(str)` | 安全 JSON 解析 | 解析后端返回的 JSON 字符串 |

### 树形数据方法（如涉及树形 CRUD）

| 方法 | 用途 |
|------|------|
| `eachTree(tree, callback)` | 遍历树 |
| `treeToList(tree)` | 树转列表 |
| `findNode(tree, func)` | 查找节点 |
| `filter(tree, func)` | 过滤树 |
| `findPath(tree, func)` | 查找路径 |

### 导入说明

- **自动导入**：`useMessage()`、`useValidator()`、`formatDate`、`generateUUID` 等大部分 Hooks 和工具函数已通过 `unplugin-auto-import` 自动导入，**无需手动 import**。
- **手动导入**：极少数组件或工具需要从 `@lingman/yd` 显式 import：

```ts
import { download, dateUtil } from '@lingman/yd'
import { UploadFile, Echart } from '@lingman/yd/components'
```
