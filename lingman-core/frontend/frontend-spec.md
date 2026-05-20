
# 芋道前端开发规范
> 基于 `yudao-ui-admin-vue3` 项目（Vue 3 + Vite + Element Plus + TypeScript + UnoCSS + Pinia）总结的前端开发规范。  
> 所有新功能、新页面必须严格遵守本规范。

---

## 目录

- [1. 技术栈概览](#1-技术栈概览)
- [2. 目录结构规范](#2-目录结构规范)
- [3. 命名规范](#3-命名规范)
- [4. API 层规范](#4-api-层规范)
- [5. 列表页规范（index.vue）](#5-列表页规范indexvue)
- [6. 表单弹窗规范（XxxForm.vue）](#6-表单弹窗规范xxxformvue)
- [7. 权限控制规范](#7-权限控制规范)
- [8. 字典使用规范](#8-字典使用规范)
- [9. 消息提示规范](#9-消息提示规范)
- [10. 状态管理规范（Pinia）](#10-状态管理规范pinia)
- [11. 样式规范](#11-样式规范)
- [12. 组件使用规范（Table / Dialog / Icon 等）](#12-组件使用规范)
- [13. `@lingman/yd` 插件规范](#13-lingmanyd-插件规范)
- [14. TypeScript 规范](#14-typescript-规范)
- [15. 常量与枚举规范](#15-常量与枚举规范)
- [16. 日期时间规范](#16-日期时间规范)
- [17. 代码质量规范](#17-代码质量规范)

---

## 1. 技术栈概览

| 分类 | 技术 | 版本 |
|------|------|------|
| 框架 | Vue 3 | 3.5.x |
| 构建工具 | Vite | 4.x / 5.x |
| UI 组件库 | Element Plus | 2.11.x |
| 业务组件/工具 | `@lingman/yd` | 0.0.39+ |
| 类型检查 | TypeScript | - |
| 原子化 CSS | UnoCSS | - |
| 状态管理 | Pinia | 2.x |
| 路由 | Vue Router | 4.x |
| HTTP 请求 | Axios（封装于 `@lingman/yd`） | 1.9.x |
| 国际化 | vue-i18n | 9.x |
| 工具库 | lodash-es、dayjs、VueUse | - |
| 图标 | @iconify/iconify + Element Plus Icons | - |

---

## 2. 目录结构规范

```
src/
├── api/              # API 层（自动生成 + 手写扩展统一存放）
│   ├── app/          # app 模块 API
│   │   └── detection-task.ts
│   ├── infra/        # infra 模块 API
│   ├── system/       # system 模块 API
│   │   └── user/
│   │       └── index.ts
│   └── types/        # API 类型定义（.d.ts）
├── assets/           # 静态资源（图片、图标等）
├── components/       # 全局公共组件（项目内自定义）
│   └── XxxComponent/
│       ├── index.ts  # 组件导出入口
│       └── src/      # 组件实现
├── config/           # 全局配置（axios 封装等）
├── constants/        # 常量/枚举定义
├── directives/       # 全局自定义指令
├── hooks/            # 组合式函数（Composables）
│   └── web/
│       └── useXxx.ts
├── layout/           # 布局组件
├── locales/          # 国际化语言包
├── plugins/          # Vue 插件注册
├── router/           # 路由配置
├── store/            # Pinia 状态管理
│   └── modules/
│       └── xxx.ts
├── styles/           # 全局样式
├── types/            # TypeScript 全局类型声明
├── utils/            # 工具函数
└── views/            # 页面视图，按业务模块分目录
    └── detection-task/
        ├── index.vue            # 列表页
        └── DetectionTaskForm.vue # 新增/修改表单弹窗
```

**规则：**
- `views/` 下按业务模块（`system`、`infra`、`crm` 等）创建子目录，模块名使用**小写**。
- 每个业务模块对应 `api/` 下同名目录。
- 组件文件必须放在 `components/` 下单独目录，包含 `index.ts` 导出。
- `hooks/web/` 只放与 Web 浏览器 API 交互的 Composables。

---

## 3. 命名规范

### 3.1 文件命名

| 类型 | 命名风格 | 示例 |
|------|----------|------|
| Vue 页面组件 | PascalCase | `UserForm.vue`, `index.vue` |
| Hooks | camelCase，`use` 前缀 | `useMessage.ts`, `useTable.ts` |
| Store 模块 | camelCase | `user.ts`, `permission.ts` |
| 工具函数 | camelCase | `formatTime.ts`, `dict.ts` |
| 类型定义 | camelCase | `global.d.ts`, `router.d.ts` |

### 3.2 组件命名

- 页面组件使用 `defineOptions({ name: 'ModulePageName' })` 显式声明组件名。
- 命名格式：`模块名` + `页面类型`，例如 `SystemUser`、`SystemUserForm`。

```ts
// ✅ 正确
defineOptions({ name: 'SystemUser' })
defineOptions({ name: 'SystemUserForm' })

// ❌ 错误
defineOptions({ name: 'userList' })
```

### 3.3 变量命名

```ts
// 列表加载状态
const loading = ref(true)           // 加载中
const total = ref(0)                // 总条数
const list = ref([])                // 列表数据
const queryParams = reactive({...}) // 搜索参数
const queryFormRef = ref()          // 搜索表单 ref

// 表单弹窗
const dialogVisible = ref(false)    // 弹窗是否显示
const dialogTitle = ref('')         // 弹窗标题
const formLoading = ref(false)      // 表单加载中
const formType = ref('')            // 表单类型：create / update
const formData = ref({...})         // 表单数据
const formRef = ref()               // 表单 ref

// 事件处理函数
const handleQuery = () => {}        // 搜索
const resetQuery = () => {}         // 重置
const openForm = () => {}           // 打开表单
const submitForm = () => {}         // 提交表单
const handleDelete = () => {}       // 删除
const handleExport = () => {}       // 导出
const handleStatusChange = () => {} // 状态变更
```

---

## 4. API 层规范

### 4.1 API 目录结构

所有 API 统一存放在 `src/api/` 下，按**是否自动生成**分为 `auto/` 和手写扩展两类：

```
api/
├── auto/                        # 【自动生成】由 `lm api` 命令从后端 Swagger 同步，禁止手动修改
│   ├── app/
│   │   └── detection-task.ts
│   ├── infra/
│   └── system/
├── app/                         # 手写扩展（同模块可在此补充）
├── infra/
├── system/                      # 手写扩展
│   └── user/
│       └── index.ts
└── types/                       # API 类型定义（.d.ts）
    └── app/
        └── detection-task.d.ts
```

| 类型 | 目录 | 维护方式 | 规则 |
|------|------|----------|------|
| **自动生成** | `src/api/auto/` | `lm api` 命令生成并覆盖 | **禁止手动修改**，每次执行命令会全量覆盖 |
| **手写扩展** | `src/api/` 下除 `auto/` 外的其他目录 | 开发者自行维护 | 按需编写，不会被命令覆盖 |

> ⚠️ **强制规则**：`src/api/auto/` 下的所有文件由 `lm api` 命令自动生成和覆盖，**不允许手动调整其中的接口**。如需扩展或修正，请在 `src/api/` 下的对应模块目录中手写补充文件。

### 4.2 API 对象导入方式

从 `api/auto/{module}/xxx.ts` 导入自动生成的 API 对象：

```ts
import { ApiAppDetectionTaskAppAdminApiAuto } from '@/api/auto/app/detection-task'
```

**调用方式**：通过 API 对象上的方法直接调用，方法内部已封装 `Get/Post/Put/Delete`：

```ts
// 分页查询
const res = await ApiAppDetectionTaskAppAdminApiAuto.pageDetectionTaskAppAdminApi({
  taskName: '测试',
  pageNo: 1,
  pageSize: 10
})

// 创建
await ApiAppDetectionTaskAppAdminApiAuto.createDetectionTaskAppAdminApi(data)

// 更新
await ApiAppDetectionTaskAppAdminApiAuto.updateDetectionTaskAppAdminApi(data)

// 删除
await ApiAppDetectionTaskAppAdminApiAuto.deleteDetectionTaskAppAdminApi({ id })

// 详情查询
const detail = await ApiAppDetectionTaskAppAdminApiAuto.getDetectionTaskAppAdminApi({ id })
```

### 4.3 HTTP 方法来源

`Get`、`Post`、`Put`、`Delete` 等 HTTP 方法由 `@lingman/yd` 提供，无需额外导入。API 自动生成文件内部已使用这些方法封装。

### 4.4 类型定义（可选）

如需提取 API 参数或响应类型，使用 `api/types/` 下的 `.d.ts` 文件：

```ts
import type { TriggerDetectionTaskAppAdminApiApiParams } from '@/api/types/app/detection-task'
```

## 5. 列表页规范（index.vue）

标准列表页使用 `useTable` hook 管理列表状态，模板结构如下：

### 5.1 Template 结构

```vue
<template>
  <!-- 搜索区域 -->
  <ContentWrap>
    <el-form ref="searchFormRef" :inline="true" :model="queryParams" class="-mb-15px">
      <el-form-item label="任务名称" prop="taskName">
        <el-input v-model="queryParams.taskName" clearable placeholder="请输入任务名称" />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" clearable placeholder="请选择状态">
          <el-option label="启用" :value="0" />
          <el-option label="禁用" :value="1" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="handleReset">
          <Icon class="mr-5px" icon="ep:refresh" />重置
        </el-button>
        <el-button type="primary" plain @click="handleCreate">
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
        <el-switch :model-value="row.status === 0" @change="(val: boolean) => handleStatusChange(row, val)" />
      </template>
      <!-- 操作列 -->
      <template #action="{ row }">
        <el-button link type="primary" @click="handleEdit(row)">编辑</el-button>
        <el-button link type="danger" @click="handleDelete(row.id)">删除</el-button>
      </template>
    </Table>
  </ContentWrap>

  <!-- 表单弹窗 -->
  <XxxForm ref="formRef" @success="methods.getList" />
</template>
```

### 5.2 Script 结构（固定顺序）

```ts
<script lang="ts" setup>
import { reactive, ref, onMounted, watch } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import type { TableColumn } from '@/types/table'
import XxxForm from './XxxForm.vue'

// 1. API 导入（从 auto 目录的自动生成文件）
import { ApiAppXxxAppAdminApiAuto } from '@/api/auto/app/xxx'

// 2. @lingman/yd 工具导入
import { formatDate } from '@lingman/yd'

// 3. 组件名声明
defineOptions({ name: 'Xxx' })

// 4. 表格列定义（静态，置于响应式状态之前）
const columns: TableColumn[] = [
  { field: 'id', label: 'ID', width: 80 },
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

// 7. 搜索（自动防抖）
const handleQuery = () => {
  methods.setSearchParams({ ...queryParams })
}

let searchTimer: ReturnType<typeof setTimeout> | null = null
watch(queryParams, () => {
  if (searchTimer) clearTimeout(searchTimer)
  searchTimer = setTimeout(handleQuery, 300)
})

// 8. 重置
const handleReset = () => {
  searchFormRef.value?.resetFields()
  methods.setSearchParams({})
}

// 9. 新增/编辑
const handleCreate = () => {
  formRef.value?.open('create')
}
const handleEdit = (row: any) => {
  formRef.value?.open('update', row)
}

// 10. 删除
const handleDelete = async (id: number) => {
  await ElMessageBox.confirm('确认删除吗？', '提示', { type: 'warning' })
  await ApiAppXxxAppAdminApiAuto.deleteXxxAppAdminApi({ id })
  ElMessage.success('删除成功')
  methods.getList()
}

// 11. 生命周期
onMounted(() => {
  // 加载字典、选项等
})
</script>
```

---

## 6. 表单弹窗规范（XxxForm.vue）

### 6.1 完整模板

> **强制规则**：新增/编辑表单**必须使用 `<Form>` 组件**（基于 Schema 的动态表单），禁止直接使用 `<el-form>`。列表页搜索表单仍使用原生 `el-form`。

```vue
<template>
  <Dialog v-model="visible" :title="title" width="800px">
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
import { ApiAppXxxAppAdminApiAuto } from '@/api/app/xxx'

defineOptions({ name: 'XxxForm' })

const emits = defineEmits(['success'])

const visible = ref(false)
const title = ref('')
const formType = ref<'create' | 'update'>('create')
const submitting = ref(false)
const formRef = ref<FormExpose>()

const schema = reactive<FormSchema[]>([
  { field: 'name', label: '名称', component: 'Input' },
  { field: 'status', label: '状态', component: 'Select', componentProps: { options: [{ label: '启用', value: 0 }, { label: '禁用', value: 1 }] } }
])

const rules = reactive<FormRules>({
  name: [{ required: true, message: '名称不能为空', trigger: 'blur' }]
})

/** 打开弹窗 */
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

/** 提交 */
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
        ElMessage.success('新增成功')
      } else {
        await ApiAppXxxAppAdminApiAuto.updateXxxAppAdminApi(data)
        ElMessage.success('修改成功')
      }
      visible.value = false
      emits('success')
    } finally {
      submitting.value = false
    }
  })
}

/** 重置 */
const resetForm = () => {
  formRef.value?.setValues({ id: undefined, name: '', status: 0 })
}
</script>
```

### 6.2 关键规则

- 表单弹窗通过 `defineExpose({ open })` 暴露 `open` 方法，由父组件通过 `ref` 调用。
- 操作成功后通过 `emits('success')` 通知父组件刷新列表（父组件监听后调用 `methods.getList()`）。
- 提交前必须调用 `formRef.value?.getElFormRef()?.validate()` 校验。
- `submitting` 在整个提交过程中保持 `true`，在 `finally` 中重置。
- 编辑时通过 `formRef.value?.setValues(row)` 回填数据，避免直接引用。
- 弹窗统一使用封装的 `<Dialog>` 组件，禁止直接使用 `<el-dialog>`。
- 表单统一使用封装的 `<Form>` 组件（基于 `FormSchema`），禁止直接使用 `<el-form>`。

---

## 7. 权限控制规范

### 7.1 按钮级权限指令

使用 `v-hasPermi` 指令控制按钮显示（**推荐方式**）：

```vue
<!-- 单个权限 -->
<el-button v-hasPermi="['system:user:create']">新增</el-button>

<!-- 多个权限（满足其一即显示） -->
<el-dropdown v-hasPermi="['system:user:delete', 'system:user:update-password']">
```

### 7.2 JS 中权限判断

在 JS 逻辑中使用 `checkPermi` 函数：

```ts
import { checkPermi } from '@/utils/permission'

// 条件禁用按钮
:disabled="!checkPermi(['system:user:update'])"

// v-if 控制
v-if="checkPermi(['system:user:delete'])"
```

### 7.3 权限标识命名规范

格式：`模块:业务:操作`

```
system:user:create     // 用户新增
system:user:update     // 用户修改
system:user:delete     // 用户删除
system:user:export     // 用户导出
system:user:import     // 用户导入
system:user:query      // 用户查询（列表页通常不加，默认可查）
```

---

## 8. 字典使用规范

### 8.1 获取字典选项

```ts
import { DICT_TYPE, getIntDictOptions, getStrDictOptions, getBoolDictOptions } from '@/utils/dict'

// 整数型字典（最常用）
getIntDictOptions(DICT_TYPE.COMMON_STATUS)

// 字符串型字典
getStrDictOptions(DICT_TYPE.SYSTEM_USER_SEX)

// 布尔型字典
getBoolDictOptions(DICT_TYPE.XXX)
```

### 8.2 在下拉框中使用

```vue
<el-select v-model="queryParams.status" placeholder="请选择状态" clearable>
  <el-option
    v-for="dict in getIntDictOptions(DICT_TYPE.COMMON_STATUS)"
    :key="dict.value"
    :label="dict.label"
    :value="dict.value"
  />
</el-select>
```

### 8.3 在表格中显示字典标签

```vue
<el-table-column label="状态">
  <template #default="scope">
    <dict-tag :type="DICT_TYPE.COMMON_STATUS" :value="scope.row.status" />
  </template>
</el-table-column>
```

### 8.4 常用字典类型

| 字典类型 | 说明 |
|----------|------|
| `DICT_TYPE.COMMON_STATUS` | 通用状态（启用/禁用） |
| `DICT_TYPE.SYSTEM_USER_SEX` | 用户性别 |
| `DICT_TYPE.SYSTEM_MENU_TYPE` | 菜单类型 |

---

## 9. 消息提示规范

优先使用 `@lingman/yd` 提供的 `useMessage()` Hook：

```ts
const message = useMessage()

// 操作成功/错误提示
message.success('操作成功')
message.error('操作失败')
message.warning('警告信息')

// 删除二次确认
await message.delConfirm('确认删除该记录吗？')

// 导出二次确认
await message.exportConfirm()

// 自定义确认弹窗
await message.confirm('确认要执行该操作吗?', '提示')

// 输入框弹窗
const result = await message.prompt('请输入内容', '提示')
```

**特殊场景**：当需要在非 setup 上下文或模板中直接使用时，可直接使用 Element Plus 的 `ElMessage` / `ElMessageBox`：

```ts
import { ElMessage, ElMessageBox } from 'element-plus'

ElMessage.success('删除成功')
await ElMessageBox.confirm('确认删除吗？', '提示', { type: 'warning' })
```

---

## 10. 状态管理规范（Pinia）

### 10.1 Store 文件结构

```ts
// src/store/modules/xxx.ts
import { store } from '@/store'
import { defineStore } from 'pinia'

export const useXxxStore = defineStore('admin-xxx', {
  state: () => ({
    // 状态定义
  }),
  getters: {
    getXxx(): XxxType {
      return this.xxx
    }
  },
  actions: {
    async setXxxAction(data: any) {
      // 异步操作
    }
  }
})

// 不使用 setup() 时的外部调用方式（在非 setup 上下文中）
export const useXxxStoreWithOut = () => {
  return useXxxStore(store)
}
```

### 10.2 使用规范

```ts
// 在 setup 中
const userStore = useUserStore()
const user = userStore.getUser

// 在非 setup 上下文（如路由守卫、工具函数）
const userStore = useUserStoreWithOut()
```

---

## 11. 样式规范

### 11.1 优先使用 UnoCSS 原子类

```vue
<!-- ✅ 优先使用原子类 -->
<div class="flex items-center justify-center">
<div class="flex items-center justify-between mb-4">
<el-input class="!w-240px" />

<!-- ❌ 避免写内联 style -->
<div style="display:flex; align-items:center;">
```

### 11.2 常用原子类

```
flex               → display: flex
items-center       → align-items: center
justify-center     → justify-content: center
justify-between    → justify-content: space-between
mb-4 / mt-4        → margin-bottom/top: 1rem
p-4                → padding: 1rem
w-full             → width: 100%
h-1/1              → height: 100%
!w-240px           → width: 240px（!important）
-mb-15px           → margin-bottom: -15px
```

### 11.3 组件级样式

- 组件内样式必须添加 `scoped`。
- 需要覆盖 Element Plus 样式时，使用 `:deep()` 选择器。

```vue
<style lang="scss" scoped>
.my-component {
  :deep(.el-table__header) {
    background-color: #f5f5f5;
  }
}
</style>
```

### 11.4 全局样式变量

使用 `var.css` 中定义的 CSS 变量，不要硬编码颜色值：

```css
/* ✅ 正确 */
background-color: var(--el-bg-color);
border-color: var(--el-border-color);
transition: background var(--transition-time-02);

/* ❌ 错误 */
background-color: #ffffff;
```

---

## 12. 组件使用规范

### 12.1 ContentWrap

所有列表页、详情页的内容区域必须包裹在 `<ContentWrap>` 中：

```vue
<ContentWrap>
  <!-- 搜索表单 -->
</ContentWrap>
<ContentWrap>
  <!-- 数据表格 -->
</ContentWrap>
```

### 12.2 Dialog

弹窗统一使用封装的 `<Dialog>` 组件，不直接使用 `<el-dialog>`：

```vue
<Dialog v-model="dialogVisible" :title="dialogTitle">
  <!-- 表单内容 -->
  <template #footer>
    <el-button type="primary" @click="submitForm">确 定</el-button>
    <el-button @click="dialogVisible = false">取 消</el-button>
  </template>
</Dialog>
```

### 12.3 Table（列表表格）

列表页表格统一使用二次封装的 `<Table>` 组件，**禁止直接使用 `<el-table>`**。

`Table` 组件内置分页（`ElPagination`），无需额外引入 `<Pagination>`。

#### 基本用法

```vue
<Table
  :columns="tableColumns"
  :data="list"
  :loading="loading"
  :pagination="{ total }"
  v-model:currentPage="queryParams.pageNo"
  v-model:pageSize="queryParams.pageSize"
>
  <!-- 自定义列：slot name 为列的 field 值 -->
  <template #status="{ row }">
    <dict-tag :type="DICT_TYPE.COMMON_STATUS" :value="row.status" />
  </template>
  <!-- 操作列 -->
  <template #action="{ row }">
    <el-button type="primary" link @click="openForm('update', row.id)">修改</el-button>
    <el-button type="danger" link @click="handleDelete(row.id)">删除</el-button>
  </template>
</Table>
```

#### 列定义规范（`TableColumn[]`）

```ts
import type { TableColumn } from '@/types/table'

const tableColumns: TableColumn[] = [
  { field: 'id', label: '编号', width: 80 },
  { field: 'name', label: '名称' },
  // 带字典的列：template 中用 #status 插槽渲染
  { field: 'status', label: '状态', width: 100 },
  // 日期列：在 formatter 中直接格式化，无需插槽
  { field: 'createTime', label: '创建时间', width: 180, formatter: dateFormatter },
  // 操作列：fixed 固定在右侧
  { field: 'action', label: '操作', width: 160, fixed: 'right' }
]
```

#### 常用 Props

| Prop | 说明 | 类型 |
|------|------|------|
| `columns` | 列配置数组 | `TableColumn[]` |
| `data` | 表格数据 | `Recordable[]` |
| `loading` | 加载状态 | `boolean` |
| `pagination` | 分页配置（含 `total`） | `Pagination` |
| `v-model:currentPage` | 当前页码（双向绑定） | `number` |
| `v-model:pageSize` | 每页条数（双向绑定） | `number` |
| `selection` | 是否显示多选列 | `boolean` |
| `reserveIndex` | 是否叠加序号 | `boolean` |

#### 分页联动

`Table` 通过 `v-model:currentPage` / `v-model:pageSize` 驱动分页，需 `watch` 触发数据重载：

```ts
watch([() => queryParams.pageNo, () => queryParams.pageSize], () => {
  getList()
})
```

### 12.4 Pagination（独立分页）

> 当使用 `<Table>` 组件时，已内置分页，**无需单独使用 `<Pagination>`**。  
> 仅在自定义布局（非 Table 场景）时使用：

```vue
<Pagination
  :total="total"
  v-model:page="queryParams.pageNo"
  v-model:limit="queryParams.pageSize"
  @pagination="getList"
/>
```

### 12.5 Icon

图标统一使用 `<Icon>` 组件，前缀 `ep:` 表示 Element Plus Icons：

```vue
<Icon icon="ep:search" />
<Icon icon="ep:plus" />
<Icon icon="ep:edit" />
<Icon icon="ep:delete" />
<Icon icon="ep:download" />
<Icon icon="ep:upload" />
<Icon icon="ep:refresh" />
```

### 12.6 DictTag

字典值显示统一使用 `<DictTag>` 组件：

```vue
<dict-tag :type="DICT_TYPE.COMMON_STATUS" :value="scope.row.status" />
```

### 12.7 DocAlert

页面顶部文档说明使用 `<doc-alert>`：

```vue
<doc-alert title="功能说明" url="https://doc.iocoder.cn/xxx/" />
```

---

## 13. `@lingman/yd` 插件规范

`@lingman/yd` 是封装芋道基础能力的插件，提供组件、Hooks、工具函数。项目通过 `app.use(LingManYd)` 全局注册。

**核心原则：`@lingman/yd` 里已有的功能，业务代码中禁止重复封装，必须优先使用。**

### 13.1 组件清单（47+）

| 分类 | 组件名 | 用途 | 备注 |
|------|--------|------|------|
| **布局** | `ContentWrap` | 内容区域包裹卡片 | 列表页/详情页必用 |
| | `ContentDetailWrap` | 详情页内容包裹 | 详情展示场景 |
| | `Sticky` | 粘性定位 | 吸顶/吸底 |
| **表格** | `Table` | 二次封装表格（内置分页） | 列表页必用 |
| | `TableSelectForm` | 表格弹窗选择 | 关联数据选择 |
| | `Pagination` | 独立分页 | 非 Table 场景使用 |
| **表单** | `Form` | 动态 Schema 表单 | 复杂表单场景 |
| | `Dialog` | 二次封装弹窗 | 表单弹窗 |
| | `InputPassword` | 密码输入框（带可见切换） | |
| | `InputWithColor` | 带颜色选择的输入框 | |
| | `ColorInput` | 颜色选择器 | |
| | `JsonEditor` | JSON 编辑器 | |
| | `Editor` | 富文本编辑器（wangEditor） | |
| | `MarkdownView` | Markdown 渲染 | |
| | `MagicCubeEditor` | 魔方编辑器 | |
| | `Crontab` | Cron 表达式选择器 | |
| | `ShortcutDateRangePicker` | 快捷日期范围选择 | |
| | `Verify` | 验证码组件（滑块/点选/文字） | |
| **数据展示** | `Descriptions` | 详情描述列表 | |
| | `DescriptionsItemLabel` | 描述项标签 | |
| | `Echart` | ECharts 图表封装 | 需从 `@lingman/yd/components` 导入 |
| | `CountTo` | 数字滚动动画 | |
| | `DictTag` | 字典标签 | 表格字典列必用 |
| | `Highlight` | 代码高亮 | |
| | `ImageViewer` | 图片预览 | |
| | `Qrcode` | 二维码生成 | |
| | `Infotip` | 信息提示气泡 | |
| | `OperateLogV2` | 操作日志 | |
| **上传** | `UploadFile` | 文件上传 | 需从 `@lingman/yd/components` 导入 |
| | `UploadImg` | 单图上传 | 需从 `@lingman/yd/components` 导入 |
| | `UploadImgs` | 多图上传 | 需从 `@lingman/yd/components` 导入 |
| | `CropperAvatar` | 头像裁剪上传 | 需从 `@lingman/yd/components` 导入 |
| | `CropperImage` | 图片裁剪 | 需从 `@lingman/yd/components` 导入 |
| **选择器** | `DeptSelectForm` | 部门选择弹窗 | |
| | `UserSelectForm` | 用户选择弹窗 | |
| | `TableSelectForm` | 表格数据选择弹窗 | |
| | `AppLinkInput` | 应用链接输入 | |
| | `AppLinkSelectDialog` | 应用链接选择弹窗 | |
| | `IconSelect` | 图标选择器 | |
| | `MapDialog` | 地图选择弹窗 | |
| **导航/反馈** | `Backtop` | 回到顶部 | |
| | `DocAlert` | 文档说明提示 | 页面顶部说明 |
| | `Error` | 错误页面（403/404/500） | |
| | `RouterSearch` | 路由搜索 | |
| | `Draggable` | 拖拽排序 | |
| | `VerticalButtonGroup` | 垂直按钮组 | |
| | `XButton` / `XTextButton` | 扩展按钮 | |
| | `CardTitle` | 卡片标题 | |
| | `IFrame` | iframe 嵌入 | |
| | `SummaryCard` | 汇总卡片 | |
| **图标** | `Icon` | 图标组件（支持 ep: 前缀） | `<Icon icon="ep:search" />` |

### 13.2 Hooks 清单（23+）

| Hook | 用途 | 使用场景 |
|------|------|----------|
| `useTable` | 列表页状态管理 | **所有列表页必用**，替代手动维护 loading/total/list |
| `useMessage` | 消息提示封装 | 替代直接使用 `ElMessage` / `ElMessageBox` |
| `useForm` | 动态表单注册 | 配合 `<Form>` 组件使用 |
| `useCrudSchemas` | CRUD 四联 Schema 生成 | 搜索+表格+表单+详情一体化配置 |
| `useValidator` | 表单校验规则生成 | `required()` / `lengthRange()` / `notSpace()` |
| `useI18n` | 国际化 | 多语言场景 |
| `useCache` | 本地缓存操作 | `wsCache.get/set/delete` |
| `useTagsView` | Tab 页签操作 | `closeCurrent` / `refreshPage` / `setTitle` |
| `useConfigGlobal` | 全局配置读取 | 项目级配置 |
| `useDesign` | 设计变量/CSS | 主题/变量相关 |
| `useEmitt` | 事件总线 | 组件间通信 |
| `useGuide` | 新手引导（driver.js） | 页面引导 |
| `useIcon` | 图标渲染（VNode） | 动态图标 |
| `useLocale` | 语言切换 | 国际化 |
| `useNetwork` | 网络状态监听 | 在线/离线检测 |
| `useNProgress` | 顶部进度条 | 路由切换等 |
| `usePageLoading` | 页面加载状态 | |
| `useScrollTo` | 滚动到指定位置 | 锚点跳转 |
| `useTimeAgo` | 相对时间（xx 分钟前） | |
| `useTitle` | 页面标题设置 | |
| `useWatermark` | 页面水印 | |
| `useNow` | 实时时钟 | |
| `usePageLoading` | 页面加载控制 | |

### 13.3 工具函数清单（按分类）

#### 日期时间（优先使用，禁止手写日期转换）

| 函数 | 用途 |
|------|------|
| `formatDate(date)` | 格式化为 `YYYY-MM-DD HH:mm:ss` |
| `formatToDate(date)` | 格式化为 `YYYY-MM-DD` |
| `formatToDateTime(date)` | 格式化为 `YYYY-MM-DD HH:mm:ss` |
| `dateFormatter(row, col, val)` | 表格日期列 formatter |
| `dateFormatter2(row, col, val)` | 表格日期列 formatter（仅日期） |
| `dateUtil` | dayjs 实例 |
| `formatPast(date)` | 相对时间（几秒前/几分钟前） |
| `formatPast2(ms)` | 毫秒转天时分秒 |
| `formatAxis(date)` | 时间问候语（上午好/下午好） |
| `beginOfDay(date)` | 当天开始时间 |
| `endOfDay(date)` | 当天结束时间 |
| `betweenDay(d1, d2)` | 计算日期间隔天数 |
| `addTime(date, ms)` | 日期加时间 |
| `getDateRange(start, end)` | 获取日期范围字符串 |
| `getDayRange(date, days)` | 获取指定天数范围 |
| `getLast7Days()` | 最近7天范围 |
| `getLast30Days()` | 最近30天范围 |
| `defaultShortcuts` | el-date-picker 快捷选项 |

#### 字典（优先使用，禁止自己写字典查询）

| 函数 | 用途 |
|------|------|
| `getDictOptions(dictType)` | 获取字典选项数组 |
| `getIntDictOptions(dictType)` | 获取整数型字典选项 |
| `getStrDictOptions(dictType)` | 获取字符串型字典选项 |
| `getBoolDictOptions(dictType)` | 获取布尔型字典选项 |
| `getDictLabel(dictType, value)` | 根据值获取字典标签 |
| `getDictObj(dictType, value)` | 根据值获取字典对象 |
| `DICT_TYPE` | 系统内置字典类型枚举 |

#### 下载（优先使用，禁止封装新的下载方法）

| 函数 | 用途 |
|------|------|
| `download.excel(data, filename)` | 下载 Excel |
| `download.word(data, filename)` | 下载 Word |
| `download.zip(data, filename)` | 下载 Zip |
| `download.html(data, filename)` | 下载 HTML |
| `download.markdown(data, filename)` | 下载 Markdown |
| `download.json(data, filename)` | 下载 JSON |
| `download.image({ url })` | 下载图片 |
| `download.base64ToFile(base64, filename)` | base64 转 File |

#### 通用工具（优先使用）

| 函数 | 用途 |
|------|------|
| `generateUUID()` | 生成 UUID |
| `generateRandomStr(length)` | 生成指定长度随机字符串 |
| `copyValueToTarget(target, source)` | 按目标对象属性复制 |
| `buildSortingField({ prop, order })` | 构建排序字段 |
| `getUrlValue(key, url)` | 获取 URL 参数 |
| `trim(str)` | 去除首尾空格 |
| `firstUpperCase(str)` | 首字母大写 |
| `jsonParse(str)` | 安全 JSON 解析 |
| `fileSizeFormatter(row, col, val)` | 文件大小格式化 |
| `generateAcceptedFileTypes(types)` | 生成 accept 属性值 |

#### 树形数据（优先使用）

| 函数 | 用途 |
|------|------|
| `eachTree(tree, callback)` | 遍历树 |
| `treeToList(tree)` | 树转列表 |
| `treeMap(tree, config)` | 树映射 |
| `findNode(tree, func)` | 查找节点 |
| `findNodeAll(tree, func)` | 查找所有匹配节点 |
| `findPath(tree, func)` | 查找路径 |
| `findPathAll(tree, func)` | 查找所有路径 |
| `filter(tree, func)` | 过滤树 |
| `forEach(tree, func)` | 遍历树 |

#### 认证/Token（优先使用）

| 函数 | 用途 |
|------|------|
| `getAccessToken()` | 获取 accessToken |
| `getRefreshToken()` | 获取 refreshToken |
| `setToken(token)` | 设置 token |
| `removeToken()` | 移除 token |
| `getTenantId()` | 获取租户 ID |
| `setTenantId(id)` | 设置租户 ID |

#### 加解密（优先使用）

| 函数 | 用途 |
|------|------|
| `encrypt(txt)` / `decrypt(txt)` | RSA 加解密 |
| `AES.encrypt(data, key)` / `AES.decrypt(data, key)` | AES 加解密 |
| `ApiEncrypt.encryptRequest(data)` | API 请求加密 |
| `ApiEncrypt.decryptResponse(data)` | API 响应解密 |

#### 金额/数字（ERP 场景）

| 函数 | 用途 |
|------|------|
| `fenToYuan(price)` | 分转元 |
| `yuanToFen(amount)` | 元转分 |
| `floatToFixed2(num)` | 保留两位小数 |
| `formatToFraction(num)` | 分数保留两位小数 |
| `erpNumberFormatter(num, digit)` | ERP 数字格式化 |
| `erpCountInputFormatter(num)` | ERP 数量格式化 |
| `erpPriceInputFormatter(num)` | ERP 金额格式化 |
| `erpCalculatePercentage(value, total)` | ERP 百分比计算 |

### 13.4 HTTP 方法

| 方法 | 用途 |
|------|------|
| `Get(url, data, options)` | GET 请求 |
| `Post(url, data, options)` | POST 请求 |
| `Put(url, data, options)` | PUT 请求 |
| `Delete(url, data, options)` | DELETE 请求 |
| `Request(config)` | 通用请求 |

> 一般情况下不需要直接使用这些方法，API 自动生成文件内部已封装。

### 13.5 优先使用 `@lingman/yd` 的典型场景

以下场景 `@lingman/yd` 已有实现，业务代码中**优先使用**，避免重复封装：

| 场景 | 优先使用 `@lingman/yd` |
|------|----------------------|
| 列表页状态管理（`loading`/`total`/`list`） | `useTable({ getListApi })` |
| 消息提示（`message`/`confirm`/`alert`） | `useMessage()` |
| 日期格式化 | `formatDate` / `dateFormatter` / `dateUtil` |
| 字典查询/转换 | `getDictOptions` / `getDictLabel` / `<DictTag>` |
| 文件下载 | `download.excel` / `download.word` 等 |
| 表单校验规则（必填/长度/空格） | `useValidator()` 的 `required`/`lengthRange`/`notSpace` |
| 表格组件 | `<Table>` 组件 |
| 弹窗组件 | `<Dialog>` 组件 |
| UUID/随机字符串生成 | `generateUUID()` / `generateRandomStr()` |
| Token 读写 | `getAccessToken()` / `setToken()` / `removeToken()` |
| 树形数据遍历/查找 | `eachTree` / `findNode` / `treeToList` 等 |
| 金额分转元/元转分 | `fenToYuan` / `yuanToFen` |

### 13.6 导入方式

**自动导入**（通过 `unplugin-auto-import`，无需手动 import）：
- 所有 Hooks：`useTable`、`useMessage`、`useForm`、`useI18n` 等
- 大部分工具函数：`formatDate`、`generateUUID`、`getDictOptions` 等

**手动导入**（需要从 `@lingman/yd` 或 `@lingman/yd/components` 显式 import）：
```ts
// 部分组件
import { Echart, CropperAvatar, UploadFile } from '@lingman/yd/components'

// 部分工具
import { formatDate, download } from '@lingman/yd'
```

---

## 14. TypeScript 规范

### 13.1 VO 类型定义

每个 API 文件中必须定义对应的 VO 接口，与后端保持一致：

```ts
// ✅ 正确：在 API 文件中定义并导出
export interface UserVO {
  id: number
  username: string
  nickname: string
  status: number
  createTime: Date
}
```

### 13.2 类型使用

```ts
// ✅ 正确：使用具体类型
const list = ref<UserApi.UserVO[]>([])
const formData = ref<UserApi.UserVO>({...})

// ❌ 避免：滥用 any
const list = ref<any[]>([])
```

### 13.3 分页参数类型

分页参数使用全局类型 `PageParam`（已在全局类型中声明）：

```ts
export const getUserPage = (params: PageParam) => {
  return request.get({ url: '/system/user/page', params })
}
```

### 13.4 表单校验类型

```ts
import { FormRules } from 'element-plus'

const formRules = reactive<FormRules>({
  username: [{ required: true, message: '用户名不能为空', trigger: 'blur' }],
  mobile: [
    { required: true, message: '手机号不能为空', trigger: 'blur' },
    { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
  ]
})
```

---

## 14. 常量与枚举规范

### 14.1 存放位置

业务常量统一放在 `src/utils/constants.ts`，按模块分组注释：

```ts
// ========== COMMON 模块 ==========
export const CommonStatusEnum = {
  ENABLE: 0,
  DISABLE: 1
}

// ========== SYSTEM 模块 ==========
export const SystemMenuTypeEnum = {
  DIR: 1,
  MENU: 2,
  BUTTON: 3
}
```

### 14.2 使用规范

```ts
import { CommonStatusEnum } from '@/utils/constants'

// ✅ 使用常量，语义清晰
if (row.status === CommonStatusEnum.ENABLE) { ... }

// ❌ 使用魔法数字
if (row.status === 0) { ... }
```

---

## 15. 日期时间规范

### 15.1 表格中格式化

优先使用 `@lingman/yd` 的 `formatDate` 函数：

```ts
import { formatDate } from '@lingman/yd'

const columns: TableColumn[] = [
  { field: 'createTime', label: '创建时间', width: 170, formatter: (_row, _col, val) => val ? formatDate(val) : '' }
]
```

也可使用 `dateFormatter`（表格通用 formatter）：

```ts
import { dateFormatter } from '@lingman/yd'

const columns: TableColumn[] = [
  { field: 'createTime', label: '创建时间', width: 180, formatter: dateFormatter }
]
```

### 15.2 日期选择器

```vue
<el-date-picker
  v-model="queryParams.createTime"
  value-format="YYYY-MM-DD HH:mm:ss"
  type="datetimerange"
  start-placeholder="开始日期"
  end-placeholder="结束日期"
  class="!w-240px"
/>
```

### 15.3 日期处理

使用 `@lingman/yd` 提供的 `dateUtil`（即 dayjs）处理日期：

```ts
import { dateUtil, formatDate, formatToDateTime } from '@lingman/yd'

formatDate(date)                    // 2024-01-01 12:00:00
formatToDateTime(date)              // 同上
formatToDate(date)                  // 2024-01-01
dateUtil(date).format('YYYY-MM-DD') // 灵活格式化
```

---

## 16. 代码质量规范

### 16.1 异步操作必须捕获异常

```ts
// ✅ 正确：删除操作，catch 为空（用户取消确认弹窗时 reject）
const handleDelete = async (id: number) => {
  try {
    await message.delConfirm()
    await XxxApi.deleteXxx(id)
    message.success(t('common.delSuccess'))
    await getList()
  } catch {}
}

// ✅ 正确：加载操作，finally 中重置 loading
const getList = async () => {
  loading.value = true
  try {
    const data = await XxxApi.getXxxPage(queryParams)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}
```

### 16.2 注释规范

- 方法注释使用 `/** 方法说明 */` 格式。
- 单行注释使用 `//`，与代码同行时加空格。
- 变量声明后用 `//` 注释说明用途。

```ts
/** 查询列表 */
const getList = async () => { ... }

const loading = ref(true)  // 列表的加载中
const total = ref(0)       // 列表的总页数
```

### 16.3 自动导入

以下 API 已通过 `unplugin-auto-import` 自动导入，**无需手动 import**：

- Vue 响应式 API：`ref`、`reactive`、`computed`、`watch`、`onMounted` 等
- Vue Router：`useRouter`、`useRoute`
- 全局 Composables：`useMessage()`、`useI18n()`、`useAttrs()`、`useSlots()`

### 16.4 Git 提交规范

提交信息格式：`type(scope): subject`

| type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | 修复 Bug |
| `style` | 样式调整（不影响功能） |
| `refactor` | 重构 |
| `chore` | 构建/依赖/工具配置更新 |
| `docs` | 文档更新 |

示例：
```
feat(system): 新增用户批量删除功能
fix(crm): 修复客户列表分页不正确的问题
style(layout): 调整侧边栏样式
```

### 16.5 代码检查命令

```bash
# ESLint 检查并修复
pnpm lint:eslint

# Prettier 格式化
pnpm lint:format

# StyleLint 检查样式
pnpm lint:style

# TypeScript 类型检查
pnpm ts:check
```
