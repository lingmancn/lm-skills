# 常见 SQL 示例

## 建表 SQL

### 基础业务表

```sql
CREATE TABLE `t_announcement` (
  `id` bigint unsigned NOT NULL COMMENT '主键 ID',
  `title` varchar(128) NOT NULL DEFAULT '' COMMENT '公告标题',
  `content` text NOT NULL COMMENT '公告内容',
  `type` tinyint NOT NULL DEFAULT '0' COMMENT '公告类型：1通知 2公告',
  `status` tinyint NOT NULL DEFAULT '0' COMMENT '状态：0禁用 1启用',
  `creator` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updater` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `deleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否删除',
  `tenant_id` bigint NOT NULL DEFAULT '0' COMMENT '租户编号',
  PRIMARY KEY (`id`),
  KEY `idx_announcement_status` (`status`),
  KEY `idx_announcement_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='公告表';
```

### 带业务外键的表

```sql
CREATE TABLE `t_approval_record` (
  `id` bigint unsigned NOT NULL COMMENT '主键 ID',
  `biz_id` bigint NOT NULL DEFAULT '0' COMMENT '业务记录 ID',
  `process_instance_id` varchar(64) NOT NULL DEFAULT '' COMMENT '流程实例 ID',
  `process_definition_key` varchar(64) NOT NULL DEFAULT '' COMMENT '流程定义 Key',
  `status` tinyint NOT NULL DEFAULT '0' COMMENT '状态：0待审批 1已通过 2已驳回',
  `reason` varchar(512) DEFAULT '' COMMENT '审批意见',
  `creator` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updater` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `deleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否删除',
  `tenant_id` bigint NOT NULL DEFAULT '0' COMMENT '租户编号',
  PRIMARY KEY (`id`),
  KEY `idx_approval_biz_id` (`biz_id`),
  KEY `idx_approval_process_instance_id` (`process_instance_id`),
  KEY `idx_approval_status` (`status`),
  KEY `idx_approval_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='审批记录表';
```

## 改表 SQL

### 添加字段

```sql
ALTER TABLE `t_announcement`
ADD COLUMN `publish_time` datetime DEFAULT NULL COMMENT '发布时间';
```

### 修改字段

```sql
ALTER TABLE `t_announcement`
MODIFY COLUMN `title` varchar(256) NOT NULL DEFAULT '' COMMENT '公告标题';
```

### 添加索引

```sql
ALTER TABLE `t_announcement`
ADD INDEX `idx_announcement_publish_time` (`publish_time`);
```

### 删除字段

```sql
ALTER TABLE `t_announcement`
DROP COLUMN `publish_time`;
```

## 查询 SQL

### 基础分页查询

```sql
SELECT id, title, content, type, status, create_time
FROM t_announcement
WHERE deleted = 0
  AND tenant_id = #{tenantId}
  AND status = #{status}
  AND title LIKE CONCAT('%', #{title}, '%')
ORDER BY create_time DESC
LIMIT #{offset}, #{pageSize};
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
  AND a.tenant_id = #{tenantId}
ORDER BY a.create_time DESC;
```

### 统计查询

```sql
-- 按状态统计数量
SELECT status, COUNT(*) as count
FROM t_announcement
WHERE deleted = 0
  AND tenant_id = #{tenantId}
GROUP BY status;

-- 按日期统计创建数量
SELECT DATE(create_time) as date, COUNT(*) as count
FROM t_announcement
WHERE deleted = 0
  AND tenant_id = #{tenantId}
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
    AND tenant_id = #{tenantId}
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
    AND a.tenant_id = #{tenantId}
    <if test="title != null and title != ''">
      AND a.title LIKE CONCAT('%', #{title}, '%')
    </if>
  ORDER BY a.create_time DESC
</select>
```
