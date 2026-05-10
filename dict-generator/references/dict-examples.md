# 字典配置示例

## 完整字典配置：公告类型

### 1. 字典类型 SQL

```sql
INSERT INTO system_dict_type (id, name, type, status, remark, create_time, update_time, deleted)
VALUES (nextval('system_dict_type_seq'), '公告类型', 'announcement_type', 0, '', now(), now(), 0);
```

### 2. 字典数据 SQL

```sql
INSERT INTO system_dict_data (id, sort, label, value, dict_type, status, color_type, remark, create_time, update_time, deleted)
VALUES 
    (nextval('system_dict_data_seq'), 1, '通知', '1', 'announcement_type', 0, 'primary', '', now(), now(), 0),
    (nextval('system_dict_data_seq'), 2, '公告', '2', 'announcement_type', 0, 'success', '', now(), now(), 0),
    (nextval('system_dict_data_seq'), 3, '政策', '3', 'announcement_type', 0, 'warning', '', now(), now(), 0);
```

### 3. Java 常量

```java
public interface DictTypeConstants {

    String OUTBOUND_TYPE = "out_type_enum";

    // 新增：公告类型字典
    String ANNOUNCEMENT_TYPE = "announcement_type";
}
```

### 4. 前端字典映射（Vue）

```typescript
// src/utils/dict.ts 或组件内
export const AnnouncementTypeDict = [
  { label: '通知', value: '1', color: 'primary' },
  { label: '公告', value: '2', color: 'success' },
  { label: '政策', value: '3', color: 'warning' }
];
```

### 5. 数据翻译使用

```java
@Data
@Schema(description = "公告 Response VO")
public class AnnouncementRespVO implements VO {

    @Schema(description = "公告类型")
    @Trans(type = TransType.SIMPLE, target = DictDataDO.class, fields = "label", ref = "typeName")
    private Integer type;

    @Schema(description = "公告类型名称（自动翻译）")
    private String typeName;
}
```

### 6. 代码中使用字典

```java
@Service
public class AnnouncementServiceImpl implements AnnouncementService {

    @Resource
    private DictDataApi dictDataApi;

    public List<DictDataRespDTO> getAnnouncementTypes() {
        return dictDataApi.getDictDataList(DictTypeConstants.ANNOUNCEMENT_TYPE);
    }
}
```

## 字典值设计规范

| 字段 | 示例值 | 说明 |
|------|--------|------|
| `label` | 通知 | 前端下拉框显示文本 |
| `value` | 1 | 数据库存储值（String） |
| `sort` | 1 | 显示顺序 |
| `color_type` | primary | Element Plus 标签颜色 |
| `status` | 0 | 0=启用 1=禁用 |

### 颜色对照

| color_type | 显示效果 |
|-----------|---------|
| primary | 蓝色 |
| success | 绿色 |
| warning | 黄色 |
| danger | 红色 |
| info | 灰色 |

## 项目现有字典参考

| 字典类型 | 用途 | 值 |
|---------|------|-----|
| `out_type_enum` | 外出类型 | 1=公差 2=休假 3=事假 4=其他 |
| `common_status` | 通用状态 | 0=禁用 1=启用 |
