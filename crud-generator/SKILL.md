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

本 Skill 生成前端页面时，必须严格遵守以下规范。所有规范来源于 `lingman-core/frontend/frontend-spec.md`，此处内联完整模板以保证生效。

### 强制规则

1. **列表页必须使用 `useTable` hook 管理状态**，禁止手动维护 `loading`/`total`/`tableList`。
2. **列表页必须使用 `<Table>` 组件**，禁止直接使用 `<el-table>`。
3. **列表页搜索区和表格区必须分别包裹在 `<ContentWrap>` 中**。
4. **表单弹窗必须使用 `<Dialog>` 组件**，禁止直接使用 `<el-dialog>`。
5. **优先使用 `@lingman/yd` 提供的组件和 Hooks**，已有功能禁止重复封装。
6. **API 文件优先引用 `src/api/auto/` 下的自动生成对象**；仅当手写扩展时才使用 `src/api/` 下的自定义路径。

### 列表页标准模板（index.vue）

```vue
<template>
  <!-- 搜索区域 -->
  <ContentWrap>
    <el-form ref="searchFormRef" :inline="true" :model="queryParams" class="-mb-15px">
      <!-- 搜索项由业务字段决定，使用 el-input / el-select / el-date-picker 等 -->
      <el-form-item>
        <el-button @click="handleReset">
          <Icon class="mr-5px" icon="ep:refresh" />重置
        </el-button>
        <el-button type="primary" @click="handleCreate">
          <Icon class="mr-5px" icon="ep:plus" />新增
        </el-button>
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表区域 -->
  <ContentWrap>
    <Table
      :columns="columns"
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
import XxxForm from './XxxForm.vue'

// 1. API 导入（优先使用 auto 目录的自动生成文件）
import { ApiAppXxxAppAdminApiAuto } from '@/api/auto/app/xxx'

// 2. @lingman/yd 工具导入
import { formatDate, DICT_TYPE } from '@lingman/yd'

// 3. 组件名声明
defineOptions({ name: 'Xxx' })

// 4. 表格列定义（静态，置于响应式状态之前）
const columns: TableColumn[] = [
  { field: 'name', label: '名称', minWidth: 150 },
  { field: 'status', label: '状态', width: 100 },
  { field: 'createTime', label: '创建时间', width: 170, formatter: (_row, _col, val) => val ? formatDate(val) : '' },
  { field: 'action', label: '操作', width: 200, fixed: 'right' }
]

// 5. 搜索参数
const queryParams = reactive({
  name: undefined as string | undefined,
  status: undefined as number | undefined
})

const searchFormRef = ref()
const formRef = ref()

// 6. useTable hook：自动管理 loading、total、list、分页
const { register, tableObject, methods } = useTable({
  getListApi: (params) =>
    ApiAppXxxAppAdminApiAuto.pageXxxAppAdminApi({
      ...queryParams,
      ...params
    })
})
methods.getList()

// 7. 搜索与重置
const handleQuery = () => {
  methods.setSearchParams({ ...queryParams })
}

const handleReset = () => {
  searchFormRef.value?.resetFields()
  methods.setSearchParams({})
}

// 8. 新增/编辑
const handleCreate = () => {
  formRef.value?.open('create')
}
const handleEdit = (row: any) => {
  formRef.value?.open('update', row)
}

// 9. 删除
const handleDelete = async (row: any) => {
  await message.delConfirm('确认删除该记录吗？')
  await ApiAppXxxAppAdminApiAuto.deleteXxxAppAdminApi({ id: row.id })
  message.success('删除成功')
  methods.getList()
}

onMounted(() => {
  // 加载字典、选项等初始化逻辑
})
</script>
```

### 表单弹窗标准模板（XxxForm.vue）

> **说明**：新增/编辑表单**必须使用 `<Form>` 组件**（基于 Schema 的动态表单）。列表页的搜索表单仍使用原生 `el-form`。

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

const submitForm = async () => {
  const elForm = formRef.value?.getElFormRef()
  if (!elForm) return
  await elForm.validate(async (valid) => {
    if (!valid) return
    submitting.value = true
    try {
      const data = formRef.value?.formModel
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
| 内容区域包裹 | `<ContentWrap>` |
| 弹窗 | `<Dialog>` |
| 表单（新增/编辑） | `<Form :schema :rules>` |
| 消息提示 | `useMessage()`（`message.success` / `message.delConfirm` / `message.confirm`） |
| 日期格式化 | `formatDate` / `dateFormatter` |
| 字典标签 | `<DictTag>` |
| 图标 | `<Icon icon="ep:xxx" />` |
| 搜索防抖 | `watch(queryParams, ...)` + `setTimeout` |

### 前端生成规则

1. **模板优先**：生成前端代码时，必须以上述"列表页标准模板"和"表单弹窗标准模板"为基础骨架，只修改业务相关的字段和逻辑。
2. **表格列动态生成**：根据 `RespVO` 字段生成 `TableColumn[]`；日期列使用 `formatter: (_row, _col, val) => val ? formatDate(val) : ''`；状态/字典列预留 `#status` 插槽使用 `<DictTag>`；操作列固定为 `{ field: 'action', label: '操作', width: 200, fixed: 'right' }`。
3. **搜索表单动态生成**：列表页搜索区使用原生 `el-form`，根据 `PageReqVO` 字段生成搜索项，初始值统一为 `undefined`。
4. **表单弹窗动态生成**：新增/编辑弹窗使用 `<Form>` 组件，根据 `SaveReqVO` 字段生成 `FormSchema[]` 和 `FormRules`。
5. **API 路径**：优先使用 `@/api/auto/app/xxx` 的自动生成 API 对象；仅当用户明确要求手写 API 时才使用 `@/api/xxx` 自定义路径。
