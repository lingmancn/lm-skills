# 权限配置示例

## 完整模块权限配置：公告管理

### 1. 菜单权限 SQL

```sql
-- 父菜单
INSERT INTO system_menu (id, name, permission, type, sort, parent_id, path, icon, component, status, create_time, update_time, deleted)
VALUES (nextval('system_menu_seq'), '公告管理', '', 1, 1, 0, 'announcement', 'el-icon-document', NULL, 0, now(), now(), 0);

-- 子菜单（列表页）
INSERT INTO system_menu (id, name, permission, type, sort, parent_id, path, icon, component, status, create_time, update_time, deleted)
VALUES (nextval('system_menu_seq'), '公告列表', 'app:announcement:query', 2, 1, 
    (SELECT id FROM system_menu WHERE name = '公告管理' AND parent_id = 0), 
    'list', '', 'app/announcement/index', 0, now(), now(), 0);

-- 按钮权限
INSERT INTO system_menu (id, name, permission, type, sort, parent_id, path, icon, component, status, create_time, update_time, deleted)
VALUES 
    (nextval('system_menu_seq'), '公告创建', 'app:announcement:create', 3, 1, 
        (SELECT id FROM system_menu WHERE name = '公告列表'), '', '', '', 0, now(), now(), 0),
    (nextval('system_menu_seq'), '公告更新', 'app:announcement:update', 3, 2, 
        (SELECT id FROM system_menu WHERE name = '公告列表'), '', '', '', 0, now(), now(), 0),
    (nextval('system_menu_seq'), '公告删除', 'app:announcement:delete', 3, 3, 
        (SELECT id FROM system_menu WHERE name = '公告列表'), '', '', '', 0, now(), now(), 0);
```

**字段说明**：
- `type`: 1=目录 2=菜单 3=按钮
- `permission`: 权限标识
- `component`: 前端组件路径

### 2. Controller 权限注解

```java
@Tag(name = "管理后台 - 公告管理")
@RestController
@RequestMapping("/admin/announcement")
@Validated
public class AnnouncementAdminController {

    @Resource
    private AnnouncementService announcementService;

    @PostMapping("/create")
    @Operation(summary = "创建公告")
    @PreAuthorize("@ss.hasPermission('app:announcement:create')")
    public CommonResult<Long> create(@Valid @RequestBody AnnouncementCreateReqVO reqVO) {
        return CommonResult.success(announcementService.create(reqVO));
    }

    @PostMapping("/update")
    @Operation(summary = "更新公告")
    @PreAuthorize("@ss.hasPermission('app:announcement:update')")
    public CommonResult<Boolean> update(@Valid @RequestBody AnnouncementUpdateReqVO reqVO) {
        announcementService.update(reqVO);
        return CommonResult.success(true);
    }

    @GetMapping("/delete")
    @Operation(summary = "删除公告")
    @PreAuthorize("@ss.hasPermission('app:announcement:delete')")
    public CommonResult<Boolean> delete(@RequestParam("id") Long id) {
        announcementService.delete(id);
        return CommonResult.success(true);
    }

    @GetMapping("/get")
    @Operation(summary = "获得公告详情")
    @PreAuthorize("@ss.hasPermission('app:announcement:query')")
    public CommonResult<AnnouncementRespVO> get(@RequestParam("id") Long id) {
        return CommonResult.success(announcementService.get(id));
    }

    @PostMapping("/page")
    @Operation(summary = "获得公告分页")
    @PreAuthorize("@ss.hasPermission('app:announcement:query')")
    public CommonResult<PageResult<AnnouncementRespVO>> page(@RequestBody AnnouncementPageReqVO reqVO) {
        return CommonResult.success(announcementService.page(reqVO));
    }
}
```

### 3. 数据权限配置

```java
@Configuration(proxyBeanMethods = false)
public class AppDataPermissionConfiguration {

    @Bean
    public DeptDataPermissionRuleCustomizer appDeptDataPermissionRuleCustomizer() {
        return rule -> {
            // 公告数据权限（按创建人/部门）
            rule.addUserColumn(AnnouncementDO.class, "creator");
            rule.addDeptColumn(AnnouncementDO.class, "dept_id");
        };
    }
}
```

### 4. 角色菜单绑定 SQL

```sql
-- 给管理员角色绑定公告菜单权限
INSERT INTO system_role_menu (role_id, menu_id)
SELECT 1, id FROM system_menu WHERE permission LIKE 'app:announcement:%';
```

## 权限标识命名对照表

| 操作 | 权限标识 | 对应按钮 |
|------|---------|---------|
| 查询 | `app:{biz}:query` | 列表查看、详情查看 |
| 创建 | `app:{biz}:create` | 新增按钮 |
| 更新 | `app:{biz}:update` | 编辑按钮 |
| 删除 | `app:{biz}:delete` | 删除按钮 |
| 导出 | `app:{biz}:export` | 导出按钮 |
| 导入 | `app:{biz}:import` | 导入按钮 |
