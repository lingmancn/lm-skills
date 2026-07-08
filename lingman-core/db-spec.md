# Lingman-Starter 数据库建表规范

## 建表必备字段

所有业务表必须包含以下公共字段，字段类型、默认值和注释保持一致：

```sql
"creator" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"create_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"updater" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"update_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"deleted" int2 NOT NULL DEFAULT 0,
```

## 表名与序列规范

- 表名统一使用 `t_` 前缀，如 `t_course`、`t_video_device`。
- 表名使用小写蛇形命名（snake_case），含义明确，避免缩写。
- 序列名不要带 `t_` 前缀，例如表 `t_course` 对应 `course_seq`。
- 主键约束命名使用 `pk_t_业务表名`，例如 `pk_t_course`。

## 字段规范

- **主键**：PostgreSQL 使用 `int8 NOT NULL`，字段名为 `id`。
- **字段命名**：小写蛇形（snake_case），与 Java 属性名一致。
- **状态字段**：使用 `int2`，用数字表示状态，配合枚举类。
- **时间字段**：使用 `timestamp(6)`，默认 `CURRENT_TIMESTAMP`。
- **逻辑删除**：`deleted` 使用 `int2`，默认 `0`。
- **字符串长度**：根据业务实际长度设定，避免过度使用 `varchar(255)`。
- **金额字段**：优先根据业务单位选择 `int8`（单位：分）或 `numeric`，避免 `float` / `double`。
- **字段注释**：必须填写。

## 索引与约束规范

- 建表语句中不要添加任何普通索引。
- 建表语句中不要添加外键约束。
- 建表语句中不要添加业务唯一约束。
- 关联字段只保留字段本身，例如 `college_id int8`。
- 如存在高频查询字段、关联字段或唯一性要求，可以在建表 SQL 之后给出索引或约束建议，并明确说明需要开发者确认后才生成对应 SQL。

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
OWNER TO "p706";

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

CREATE SEQUENCE course_seq START 10000;
```

## 改表示例

```sql
-- 添加字段
ALTER TABLE "public"."t_task"
ADD COLUMN "exec_interval" int4 DEFAULT 5;

COMMENT ON COLUMN "public"."t_task"."exec_interval" IS '执行间隔（秒）';

-- 修改字段类型
ALTER TABLE "public"."t_task"
ALTER COLUMN "remark" TYPE varchar(1000) COLLATE "pg_catalog"."default";
```

## 索引建议示例

建表时不直接生成索引 SQL。只有开发者确认需要添加索引后，才生成类似以下 SQL：

```sql
CREATE INDEX "idx_task_status" ON "public"."t_task" USING btree ("status");
CREATE INDEX "idx_task_model_id" ON "public"."t_task" USING btree ("model_id");
```

## 与 DO 的映射关系

| PostgreSQL 类型 | Java 类型 | DO 注解 |
|---------------|----------|---------|
| `int8` / `bigint` | `Long` | `@TableField` |
| `varchar` | `String` | `@TableField` |
| `text` | `String` | `@TableField` |
| `int2` / `smallint` | `Short` / `Integer` | `@TableField` |
| `int4` / `int` / `integer` | `Integer` | `@TableField` |
| `timestamp` | `LocalDateTime` | `@TableField` |
| `numeric` / `decimal` | `BigDecimal` | `@TableField` |
| `json` / `jsonb` | `String` / 对象 | `@TableField(typeHandler = ...)` |

## 注意事项

- DO 基类选择：未启用多租户用 `BaseDO`，启用多租户用 `TenantBaseDO`
- 主键使用 PostgreSQL 序列：`@KeySequence("xxx_seq")` + `@TableId(type = IdType.INPUT)`，序列名不要带 `t_` 前缀。
- JSON 字段需加 `autoResultMap = true` + 自定义 TypeHandler
- 修改字段时，需同步更新对应的 DO 类
