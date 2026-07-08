# 常见 SQL 示例

## 建表 SQL

### 基础业务表

```sql
CREATE TABLE "public"."t_announcement" (
"id" int8 NOT NULL,
"title" varchar(128) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
"content" text COLLATE "pg_catalog"."default" NOT NULL,
"type" int2 NOT NULL DEFAULT 0,
"status" int2 NOT NULL DEFAULT 0,
"creator" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"create_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"updater" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"update_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"deleted" int2 NOT NULL DEFAULT 0,
CONSTRAINT "pk_t_announcement" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."t_announcement"
OWNER TO "p706";

COMMENT ON COLUMN "public"."t_announcement"."id" IS '主键ID';

COMMENT ON COLUMN "public"."t_announcement"."title" IS '公告标题';

COMMENT ON COLUMN "public"."t_announcement"."content" IS '公告内容';

COMMENT ON COLUMN "public"."t_announcement"."type" IS '公告类型：1通知 2公告';

COMMENT ON COLUMN "public"."t_announcement"."status" IS '状态：0禁用 1启用';

COMMENT ON COLUMN "public"."t_announcement"."creator" IS '创建者';

COMMENT ON COLUMN "public"."t_announcement"."create_time" IS '创建时间';

COMMENT ON COLUMN "public"."t_announcement"."updater" IS '更新者';

COMMENT ON COLUMN "public"."t_announcement"."update_time" IS '更新时间';

COMMENT ON COLUMN "public"."t_announcement"."deleted" IS '逻辑删除标识';

COMMENT ON TABLE "public"."t_announcement" IS '公告表';

CREATE SEQUENCE announcement_seq START 10000;
```

### 带业务关联字段的表

建表时只保留关联字段本身，不在 `CREATE TABLE` 中添加外键约束或普通索引。

```sql
CREATE TABLE "public"."t_approval_record" (
"id" int8 NOT NULL,
"biz_id" int8 NOT NULL DEFAULT 0,
"process_instance_id" varchar(64) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
"process_definition_key" varchar(64) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
"status" int2 NOT NULL DEFAULT 0,
"reason" varchar(512) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"creator" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"create_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"updater" varchar(64) COLLATE "pg_catalog"."default" DEFAULT ''::character varying,
"update_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
"deleted" int2 NOT NULL DEFAULT 0,
CONSTRAINT "pk_t_approval_record" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."t_approval_record"
OWNER TO "p706";

COMMENT ON COLUMN "public"."t_approval_record"."id" IS '主键ID';

COMMENT ON COLUMN "public"."t_approval_record"."biz_id" IS '业务记录ID';

COMMENT ON COLUMN "public"."t_approval_record"."process_instance_id" IS '流程实例ID';

COMMENT ON COLUMN "public"."t_approval_record"."process_definition_key" IS '流程定义Key';

COMMENT ON COLUMN "public"."t_approval_record"."status" IS '状态：0待审批 1已通过 2已驳回';

COMMENT ON COLUMN "public"."t_approval_record"."reason" IS '审批意见';

COMMENT ON COLUMN "public"."t_approval_record"."creator" IS '创建者';

COMMENT ON COLUMN "public"."t_approval_record"."create_time" IS '创建时间';

COMMENT ON COLUMN "public"."t_approval_record"."updater" IS '更新者';

COMMENT ON COLUMN "public"."t_approval_record"."update_time" IS '更新时间';

COMMENT ON COLUMN "public"."t_approval_record"."deleted" IS '逻辑删除标识';

COMMENT ON TABLE "public"."t_approval_record" IS '审批记录表';

CREATE SEQUENCE approval_record_seq START 10000;
```

## 改表 SQL

### 添加字段

```sql
ALTER TABLE "public"."t_announcement"
ADD COLUMN "publish_time" timestamp(6) DEFAULT NULL;

COMMENT ON COLUMN "public"."t_announcement"."publish_time" IS '发布时间';
```

### 修改字段

```sql
ALTER TABLE "public"."t_announcement"
ALTER COLUMN "title" TYPE varchar(256) COLLATE "pg_catalog"."default";
```

### 索引建议

建表时不要直接添加索引。只有开发者确认需要索引后，才生成索引 SQL，例如：

```sql
CREATE INDEX "idx_announcement_publish_time" ON "public"."t_announcement" USING btree ("publish_time");
```

### 删除字段

```sql
ALTER TABLE "public"."t_announcement"
DROP COLUMN "publish_time";
```

## 查询 SQL

### 基础分页查询

```sql
SELECT id, title, content, type, status, create_time
FROM t_announcement
WHERE deleted = 0
  AND status = #{status}
  AND title LIKE CONCAT('%', #{title}, '%')
ORDER BY create_time DESC
LIMIT #{pageSize} OFFSET #{offset};
```

### 联表查询

```sql
SELECT
  a.id,
  a.title,
  a.content,
  a.status,
  u.nickname as creator_name,
  a.create_time
FROM t_announcement a
LEFT JOIN system_user u ON a.creator = u.id
WHERE a.deleted = 0
ORDER BY a.create_time DESC;
```

### 统计查询

```sql
-- 按状态统计数量
SELECT status, COUNT(*) as count
FROM t_announcement
WHERE deleted = 0
GROUP BY status;

-- 按日期统计创建数量
SELECT DATE(create_time) as date, COUNT(*) as count
FROM t_announcement
WHERE deleted = 0
  AND create_time BETWEEN #{startTime} AND #{endTime}
GROUP BY DATE(create_time)
ORDER BY date;
```

## Mapper XML 示例

### 基础查询

```xml
<select id="selectPage" resultType="com.lm.app.models.entity.AnnouncementDO">
  SELECT * FROM t_announcement
  WHERE deleted = 0
    <if test="title != null and title != ''">
      AND title LIKE CONCAT('%', #{title}, '%')
    </if>
    <if test="status != null">
      AND status = #{status}
    </if>
  ORDER BY create_time DESC
</select>
```

### 联表分页查询

```xml
<select id="selectPageWithJoin" resultType="com.lm.app.controller.admin.announcement.vo.AnnouncementRespVO">
  SELECT
    a.id,
    a.title,
    a.content,
    a.type,
    a.status,
    u.nickname as creator_name,
    a.create_time
  FROM t_announcement a
  LEFT JOIN system_user u ON a.creator = u.id
  WHERE a.deleted = 0
    <if test="title != null and title != ''">
      AND a.title LIKE CONCAT('%', #{title}, '%')
    </if>
  ORDER BY a.create_time DESC
</select>
```
