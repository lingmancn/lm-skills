# Lingman-Starter 数据库建表规范

## 建表必备字段

所有业务表必须包含以下字段：

```sql
`id` bigint NOT NULL COMMENT '主键 ID',
`creator` varchar(64) DEFAULT '' COMMENT '创建者',
`create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`updater` varchar(64) DEFAULT '' COMMENT '更新者',
`update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
`deleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否删除',
PRIMARY KEY (`id`)
```

**多租户场景**（如 p705 工程）：如果项目启用了多租户，需额外添加：

```sql
`tenant_id` bigint NOT NULL DEFAULT '0' COMMENT '租户编号',
```

## 表名规范

- 统一使用 `t_` 前缀，如 `t_task`、`t_video_device`
- 小写蛇形命名（snake_case）
- 含义明确，避免缩写

## 字段规范

- **主键**：`bigint`，策略根据数据库和业务场景选择
- **字段命名**：小写蛇形（snake_case），与 Java 属性名一致
- **状态字段**：`tinyint` 或 `int`，用数字表示状态，配合枚举类
- **时间字段**：`datetime`，默认 `CURRENT_TIMESTAMP`
- **逻辑删除**：`deleted` bit(1)，默认 `b'0'`
- **租户字段**（如启用多租户）：`tenant_id` bigint
- **字符串长度**：根据业务实际长度设定，避免过度使用 `varchar(255)`
- **金额字段**：`decimal(18,2)`，避免 `float` / `double`
- **字段注释**：必须填写

## 索引规范

- 主键默认索引
- 外键字段必须建索引
- 经常查询的条件字段建索引
- 联合索引遵循最左匹配原则
- 索引命名：`idx_表名_字段名`

## 建表示例

```sql
CREATE TABLE "public"."t_course" (
  "id" int8 NOT NULL,
  "name" varchar(100) COLLATE "pg_catalog"."default" NOT NULL,
  "college_id" int8,
  "price" int8,
  "deposit" int8,
  "class_hours" int4,
  "video_url" varchar(500) COLLATE "pg_catalog"."default" DEFAULT NULL::character varying,
  "outline" text COLLATE "pg_catalog"."default",
  "detail" text COLLATE "pg_catalog"."default",
  "status" int2 NOT NULL DEFAULT 1,
  "creator" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
  "create_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "updater" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
  "update_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "deleted" int2 NOT NULL DEFAULT 0,
  CONSTRAINT "pk_t_course" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."t_course" 
  OWNER TO "p705";

COMMENT ON COLUMN "public"."t_course"."id" IS '主键ID';

COMMENT ON COLUMN "public"."t_course"."name" IS '课程名称';

COMMENT ON COLUMN "public"."t_course"."college_id" IS '学院ID';

COMMENT ON COLUMN "public"."t_course"."price" IS '课程价格（单位：分）';

COMMENT ON COLUMN "public"."t_course"."deposit" IS '定金（单位：分）';

COMMENT ON COLUMN "public"."t_course"."class_hours" IS '课时数';

COMMENT ON COLUMN "public"."t_course"."video_url" IS '视频URL';

COMMENT ON COLUMN "public"."t_course"."outline" IS '课程大纲';

COMMENT ON COLUMN "public"."t_course"."detail" IS '课程详情';

COMMENT ON COLUMN "public"."t_course"."status" IS '状态: 0-禁用, 1-启用';

COMMENT ON COLUMN "public"."t_course"."creator" IS '创建者';

COMMENT ON COLUMN "public"."t_course"."create_time" IS '创建时间';

COMMENT ON COLUMN "public"."t_course"."updater" IS '更新者';

COMMENT ON COLUMN "public"."t_course"."update_time" IS '更新时间';

COMMENT ON COLUMN "public"."t_course"."deleted" IS '逻辑删除标识';

COMMENT ON TABLE "public"."t_course" IS '课程主表';

CREATE SEQUENCE task_seq START 10000;
```

## 改表示例

```sql
-- 添加字段
ALTER TABLE t_task
ADD COLUMN exec_interval int DEFAULT 5 COMMENT '执行间隔（秒）';

-- 修改字段类型
ALTER TABLE t_task
ALTER COLUMN remark TYPE varchar(1000);

-- 添加索引
CREATE INDEX idx_task_status ON t_task(status);
CREATE INDEX idx_task_model_id ON t_task(model_id);
```

## 与 DO 的映射关系

| PostgreSQL 类型 | Java 类型 | DO 注解 |
|---------------|----------|---------|
| `bigint` | `Long` | `@TableField` |
| `varchar` | `String` | `@TableField` |
| `text` | `String` | `@TableField` |
| `smallint` | `Short` / `Integer` | `@TableField` |
| `int` / `integer` | `Integer` | `@TableField` |
| `timestamp` | `LocalDateTime` | `@TableField` |
| `decimal` | `BigDecimal` | `@TableField` |
| `bit(1)` | `Boolean` | `@TableField` |
| `json` / `jsonb` | `String` / 对象 | `@TableField(typeHandler = ...)` |

## 注意事项

- DO 基类选择：未启用多租户用 `BaseDO`，启用多租户用 `TenantBaseDO`
- 主键使用 PostgreSQL 序列：`@KeySequence("xxx_seq")` + `@TableId(type = IdType.INPUT)`
- JSON 字段需加 `autoResultMap = true` + 自定义 TypeHandler
- 修改字段时，需同步更新对应的 DO 类
