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
- [13. TypeScript 规范](#13-typescript-规范)
- [14. 常量与枚举规范](#14-常量与枚举规范)
- [15. 日期时间规范](#15-日期时间规范)
- [16. 代码质量规范](#16-代码质量规范)

---

## 1. 技术栈概览

| 分类 | 技术 | 版本 |
|------|------|------|
| 框架 | Vue 3 | 3.5.x |
| 构建工具 | Vite | 4.x |
| UI 组件库 | Element Plus | 2.11.x |
| 类型检查 | TypeScript | - |
| 原子化 CSS | UnoCSS | - |
| 状态管理 | Pinia | 2.x |
| 路由 | Vue Router | 4.x |
| HTTP 请求 | Axios（封装） | 1.9.x |
| 国际化 | vue-i18n | 9.x |
| 工具库 | lodash-es、dayjs、VueUse | - |
| 图标 | @iconify/iconify + Element Plus Icons | - |

---

## 2. 目录结构规范

```
src/
├── api/              # 接口层，按业务模块分目录
│   └── system/
│       └── user/
│           └── index.ts
├── assets/           # 静态资源（图片、图标等）
├── components/       # 全局公共组件
│   └── XxxComponent/
│       ├── index.ts  # 组件导出入口
│       └── src/      # 组件实现
├── config/           # 全局配置（axios 封装等）
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
    └── system/
        └── user/
            ├── index.vue       # 列表页
            ├── UserForm.vue    # 新增/修改表单弹窗
            └── UserImportForm.vue
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

### 4.1 文件结构 使用自动生成api即可 

### 4.2 在页面中使用 API （使用自定生成的api）

必须使用命名空间导入方式，方便追溯来源： 

const data = await UserApi.getUserPage(queryParams)
await UserApi.deleteUser(id)


## 5. 列表页规范（index.vue）

标准列表页模板结构如下：

### 5.1 Template 结构

```vue
<template>
  <!-- 文档提示（可选） -->
  <doc-alert title="页面说明" url="https://doc.iocoder.cn/xxx/" />

  <!-- 搜索区域 -->
  <ContentWrap>
    <el-form class="-mb-15px" :model="queryParams" ref="queryFormRef" :inline="true" label-width="68px">
      <!-- 搜索条件 -->
      <el-form-item label="字段名" prop="fieldName">
        <el-input
          v-model="queryParams.fieldName"
          placeholder="请输入字段名"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <!-- 操作按钮 -->
      <el-form-item>
        <el-button @click="handleQuery"><Icon icon="ep:search" />搜索</el-button>
        <el-button @click="resetQuery"><Icon icon="ep:refresh" />重置</el-button>
        <el-button type="primary" plain @click="openForm('create')" v-hasPermi="['module:business:create']">
          <Icon icon="ep:plus" /> 新增
        </el-button>
        <el-button type="success" plain @click="handleExport" :loading="exportLoading" v-hasPermi="['module:business:export']">
          <Icon icon="ep:download" />导出
        </el-button>
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表区域 -->
  <ContentWrap>
    <!-- Table 为二次封装组件，自带分页，通过 :columns 配置列，slot name 为 field 值 -->
    <Table
      :columns="tableColumns"
      :data="list"
      :loading="loading"
      :pagination="{ total }"
      v-model:currentPage="queryParams.pageNo"
      v-model:pageSize="queryParams.pageSize"
    >
      <!-- 字典类型列：用 #field 插槽自定义渲染 -->
      <template #status="{ row }">
        <dict-tag :type="DICT_TYPE.COMMON_STATUS" :value="row.status" />
      </template>
      <!-- 操作列 -->
      <template #action="{ row }">
        <div class="flex items-center justify-center">
          <el-button type="primary" link @click="openForm('update', row.id)" v-hasPermi="['module:business:update']">
            <Icon icon="ep:edit" />修改
          </el-button>
          <el-button type="danger" link @click="handleDelete(row.id)" v-hasPermi="['module:business:delete']">
            <Icon icon="ep:delete" />删除
          </el-button>
        </div>
      </template>
    </Table>
  </ContentWrap>

  <!-- 表单弹窗 -->
  <XxxForm ref="formRef" @success="getList" />
</template>
```

### 5.2 Script 结构（固定顺序）

```ts
<script lang="ts" setup>
// 1. 工具/常量导入
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'
import { checkPermi } from '@/utils/permission'
import { dateFormatter } from '@/utils/formatTime'
import download from '@/utils/download'
import { CommonStatusEnum } from '@/utils/constants'

// 2. API 导入（命名空间方式）
import * as XxxApi from '@/api/module/xxx'

// 3. 子组件导入
import XxxForm from './XxxForm.vue'

// 4. 组件名声明
defineOptions({ name: 'ModuleXxx' })

// 5. Composables
const message = useMessage()
const { t } = useI18n()

// 6. 表格列定义（静态，置于响应式状态之前）
import type { TableColumn } from '@/types/table'
const tableColumns: TableColumn[] = [
  { field: 'id', label: '编号', width: 80 },
  { field: 'name', label: '名称' },
  { field: 'status', label: '状态', width: 100 },
  // 日期列直接在列配置中使用 formatter，无需插槽
  { field: 'createTime', label: '创建时间', width: 180, formatter: dateFormatter },
  { field: 'action', label: '操作', width: 160, fixed: 'right' }
]

// 7. 响应式状态
const loading = ref(true)
const total = ref(0)
const list = ref([])
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  // ...业务字段，初始值统一为 undefined
})
const queryFormRef = ref()

// 8. 监听分页变化（Table 组件通过 v-model 驱动，需 watch 触发数据刷新）
watch([() => queryParams.pageNo, () => queryParams.pageSize], () => {
  getList()
})

// 9. 业务方法
/** 查询列表 */
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

/** 搜索按钮操作 */
const handleQuery = () => {
  queryParams.pageNo = 1
  getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value?.resetFields()
  handleQuery()
}

/** 添加/修改操作 */
const formRef = ref()
const openForm = (type: string, id?: number) => {
  formRef.value.open(type, id)
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    await message.delConfirm()
    await XxxApi.deleteXxx(id)
    message.success(t('common.delSuccess'))
    await getList()
  } catch {}
}

/** 导出按钮操作 */
const exportLoading = ref(false)
const handleExport = async () => {
  try {
    await message.exportConfirm()
    exportLoading.value = true
    const data = await XxxApi.exportXxx(queryParams)
    download.excel(data, 'xxx数据.xls')
  } catch {
  } finally {
    exportLoading.value = false
  }
}

// 10. 生命周期
onMounted(() => {
  getList()
})
</script>
```

---

## 6. 表单弹窗规范（XxxForm.vue）

### 6.1 完整模板

```vue
<template>
  <Dialog v-model="dialogVisible" :title="dialogTitle">
    <el-form
      ref="formRef"
      v-loading="formLoading"
      :model="formData"
      :rules="formRules"
      label-width="80px"
    >
      <el-form-item label="字段名" prop="fieldName">
        <el-input v-model="formData.fieldName" placeholder="请输入字段名" />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button :disabled="formLoading" type="primary" @click="submitForm">确 定</el-button>
      <el-button @click="dialogVisible = false">取 消</el-button>
    </template>
  </Dialog>
</template>

<script lang="ts" setup>
import * as XxxApi from '@/api/module/xxx'
import { FormRules } from 'element-plus'

defineOptions({ name: 'ModuleXxxForm' })

const { t } = useI18n()
const message = useMessage()

const dialogVisible = ref(false)
const dialogTitle = ref('')
const formLoading = ref(false)
const formType = ref('')   // 'create' | 'update'
const formData = ref({
  id: undefined,
  // ...业务字段初始值
})
const formRules = reactive<FormRules>({
  fieldName: [{ required: true, message: '字段名不能为空', trigger: 'blur' }]
})
const formRef = ref()

/** 打开弹窗 */
const open = async (type: string, id?: number) => {
  dialogVisible.value = true
  dialogTitle.value = type === 'create' ? '新增XXX' : '修改XXX'
  formType.value = type
  resetForm()
  if (id) {
    formLoading.value = true
    try {
      formData.value = await XxxApi.getXxx(id)
    } finally {
      formLoading.value = false
    }
  }
}
defineExpose({ open }) // 暴露给父组件调用

const emits = defineEmits(['success'])

/** 提交表单 */
const submitForm = async () => {
  await formRef.value.validate()
  formLoading.value = true
  try {
    const data = formData.value
    if (formType.value === 'create') {
      await XxxApi.createXxx(data)
      message.success(t('common.createSuccess'))
    } else {
      await XxxApi.updateXxx(data)
      message.success(t('common.updateSuccess'))
    }
    dialogVisible.value = false
    emits('success')
  } finally {
    formLoading.value = false
  }
}

/** 重置表单 */
const resetForm = () => {
  formData.value = {
    id: undefined,
    // ...字段初始值
  }
  formRef.value?.resetFields()
}
</script>
```

### 6.2 关键规则

- 表单弹窗通过 `defineExpose({ open })` 暴露 `open` 方法，由父组件通过 `ref` 调用。
- 操作成功后通过 `emits('success')` 通知父组件刷新列表。
- 提交前必须调用 `formRef.value.validate()` 校验。
- 弹窗关闭时通过 `resetForm()` 重置数据，避免脏数据。
- `formLoading` 在整个提交过程中保持 `true`，在 `finally` 中重置。

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

统一使用 `useMessage()` Hook，不直接调用 `ElMessage` 或 `ElMessageBox`：

```ts
const message = useMessage()

// 操作成功提示
message.success('操作成功')
message.success(t('common.delSuccess'))

// 错误提示
message.error('操作失败')

// 删除二次确认（固定用法）
await message.delConfirm()

// 导出二次确认（固定用法）
await message.exportConfirm()

// 自定义确认弹窗
await message.confirm('确认要启用该用户吗?')

// 输入框弹窗
const result = await message.prompt('请输入新密码', '提示')
const password = result.value
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

### 12.6 DocAlert

页面顶部文档说明使用 `<doc-alert>`：

```vue
<doc-alert title="功能说明" url="https://doc.iocoder.cn/xxx/" />
```

---

## 13. TypeScript 规范

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

```vue
<el-table-column
  label="创建时间"
  align="center"
  prop="createTime"
  :formatter="dateFormatter"
  width="180"
/>
```

```ts
import { dateFormatter } from '@/utils/formatTime'
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

使用 `dayjs` 处理日期，不使用原生 `Date` 操作：

```ts
import dayjs from 'dayjs'

dayjs(date).format('YYYY-MM-DD HH:mm:ss')
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
