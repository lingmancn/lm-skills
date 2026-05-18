---
name: dict-generator
description: Lingman-Starter 框架字典配置生成助手。当用户需要：(1) 生成字典类型和字典值 SQL (2) 生成字典对应的 Java 枚举类 (3) 生成前端字典映射配置 (4) 在代码中使用字典翻译注解 时触发此技能。不要在以下场景触发：生成业务 CRUD 代码（由 crud-generator 处理）、生成权限（由 permission-generator 处理）、文档查询（由 doc-qa 处理）。
---

# 字典配置生成指南

## 字典体系概述

lingman-starter 使用 yudao 字典系统：

| 层级 | 表 | 说明 |
|------|-----|------|
| 字典类型 | `system_dict_type` | 定义字典分类，如 `out_type_enum` |
| 字典数据 | `system_dict_data` | 定义具体字典值，如 `1=公差`、`2=休假` |
| 枚举类 | `DictTypeConstants` | Java 代码中引用字典类型的常量 |
| 数据翻译 | `@Trans` 注解 | 自动将字典值翻译为显示文本 |

## 生成范围

1. **字典类型 SQL** — `system_dict_type` 插入语句
2. **字典数据 SQL** — `system_dict_data` 插入语句
3. **Java 枚举常量** — `DictTypeConstants` 中添加常量
4. **前端映射** — Vue 前端字典映射配置
5. **数据翻译注解** — `@Trans` 使用示例

## 字典命名规范

```
{业务}_{含义}_enum

示例：
- out_type_enum         # 外出类型
- announcement_type     # 公告类型
- common_status         # 通用状态
```

## 字典值设计规范

| 字段 | 说明 |
|------|------|
| `label` | 显示文本（前端展示） |
| `value` | 存储值（数据库存储） |
| `sort` | 排序号 |
| `status` | 状态：0 禁用 1 启用 |
| `color_type` | 标签颜色（success/warning/danger/info） |

## 数据翻译使用

```java
@Data
@Schema(description = "外出记录 Response VO")
public class OutboundRecordRespVO implements VO {

    @Schema(description = "外出类型")
    @Trans(type = TransType.SIMPLE, target = DictDataDO.class, fields = "label", ref = "outTypeName")
    private Integer outType;

    @Schema(description = "外出类型名称（自动翻译）")
    private String outTypeName;
}
```

## Java 枚举常量模板

```java
public interface DictTypeConstants {

    String OUTBOUND_TYPE = "out_type_enum";

    // 新增字典类型
    String ANNOUNCEMENT_TYPE = "announcement_type";
}
```

## 前端字典使用

前端优先使用 `@lingman/yd` 提供的字典工具：

### 下拉框选项

```ts
import { getIntDictOptions, getStrDictOptions, getBoolDictOptions } from '@lingman/yd'

// 整数型字典（最常用）
const options = getIntDictOptions('out_type_enum')

// 字符串型字典
const options = getStrDictOptions('system_user_sex')

// 布尔型字典
const options = getBoolDictOptions('xxx_bool_enum')
```

```vue
<el-select v-model="queryParams.status" clearable placeholder="请选择状态">
  <el-option
    v-for="dict in getIntDictOptions('out_type_enum')"
    :key="dict.value"
    :label="dict.label"
    :value="dict.value"
  />
</el-select>
```

### 表格中显示字典标签

```vue
<template #status="{ row }">
  <DictTag :type="DICT_TYPE.COMMON_STATUS" :value="row.status" />
</template>
```

### 获取字典标签文本

```ts
import { getDictLabel } from '@lingman/yd'

const label = getDictLabel('out_type_enum', row.outType)
```

### 常量定义（项目内）

在 `src/constants/dict.ts` 中定义字典类型常量：

```ts
export const DICT_TYPE_ALARM_LEVEL = 'alarm_level_enum'
export const DICT_TYPE_COMMON_STATUS = 'common_status'
```

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 字典配置示例 | [dict-examples.md](references/dict-examples.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
| 前端规范 | [frontend-spec.md](../lingman-core/frontend/frontend-spec.md) |
