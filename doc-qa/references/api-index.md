# API 文档索引

## 目录

### 系统管理 (system)

| API | 路径 | 用途 |
|-----|------|------|
| AdminUserApi | [user/管理员用户API - AdminUserApi.md](../../lingman-core/api-docs/user/管理员用户API - AdminUserApi.md) | 查询管理员用户信息、按部门/岗位获取用户列表、校验用户有效性 |
| PermissionApi | [permission/权限API - PermissionApi.md](../../lingman-core/api-docs/permission/权限API - PermissionApi.md) | 权限校验、角色查询、部门数据权限 |
| RoleApi | [permission/角色API - RoleApi.md](../../lingman-core/api-docs/permission/角色API - RoleApi.md) | 角色校验、查询 |
| DeptApi | [dept/部门 API - DeptApi.md](../../lingman-core/api-docs/dept/部门 API - DeptApi.md) | 部门查询、校验、获取子部门 |
| PostApi | [dept/岗位 API - PostApi.md](../../lingman-core/api-docs/dept/岗位 API - PostApi.md) | 岗位查询、校验 |
| DictDataCommonApi | [dict/字典数据 API - DictDataCommonApi.md](../../lingman-core/api-docs/dict/字典数据 API - DictDataCommonApi.md) | 字典数据查询、校验 |

### 基础设施 (infra)

| API | 路径 | 用途 |
|-----|------|------|
| OperateLogApi | [logger/操作日志API - OperateLogApi.md](../../lingman-core/api-docs/logger/操作日志API - OperateLogApi.md) | 操作日志记录 |
| LoginLogApi | [logger/登入日志API - LoginLogApi.md](../../lingman-core/api-docs/logger/登入日志API - LoginLogApi.md) | 登录日志记录 |
| NotifyMessageSendApi | [notify/站内信发送API - NotifyMessageSendApi.md](../../lingman-core/api-docs/notify/站内信发送API - NotifyMessageSendApi.md) | 站内信发送 |
| SmsSendApi | [sms/短信发送API - SmsSendApi.md](../../lingman-core/api-docs/sms/短信发送API - SmsSendApi.md) | 短信发送 |
| SmsCodeApi | [sms/短信验证码API - SmsCodeApi.md](../../lingman-core/api-docs/sms/短信验证码API - SmsCodeApi.md) | 短信验证码 |
| MailSendApi | [mail/邮件发送API - MailSendApi.md](../../lingman-core/api-docs/mail/邮件发送API - MailSendApi.md) | 邮件发送 |

### 会员/社交 (member)

| API | 路径 | 用途 |
|-----|------|------|
| SocialClientApi | [social/社交应用API - SocialClientApi.md](../../lingman-core/api-docs/social/社交应用API - SocialClientApi.md) | 社交应用管理 |
| SocialUserApi | [social/社交用户API - SocialUserApi.md](../../lingman-core/api-docs/social/社交用户API - SocialUserApi.md) | 社交用户管理 |

## 常用 API 速查

### 获取当前登录用户

```java
Long userId = SecurityFrameworkUtils.getLoginUserId();
```

### 使用 AdminUserApi

```java
@Resource
private AdminUserApi adminUserApi;

// 获取单个用户
AdminUserRespDTO user = adminUserApi.getUser(userId);

// 获取用户 Map
Map<Long, AdminUserRespDTO> userMap = adminUserApi.getUserMap(userIds);

// 获取部门下的用户
List<AdminUserRespDTO> users = adminUserApi.getUserListByDeptIds(deptIds);
```

### 使用 PermissionApi

```java
@Resource
private PermissionApi permissionApi;

// 判断是否有任一权限
Boolean hasPerm = permissionApi.hasAnyPermissions(userId, "system:user:query");

// 判断是否有任一角色
Boolean hasRole = permissionApi.hasAnyRoles(userId, "admin");

// 获取部门数据权限
DeptDataPermissionRespDTO deptPerm = permissionApi.getDeptDataPermission(userId);
```

### 使用 DictDataApi

```java
@Resource
private DictDataApi dictDataApi;

// 获取字典数据列表
List<DictDataRespDTO> dictList = dictDataApi.getDictDataList("dict_type_code");

// 校验字典值
List<String> values = Arrays.asList("1", "2");
dictDataApi.validateDictDataList("dict_type_code", values);
```

### 使用 DeptApi

```java
@Resource
private DeptApi deptApi;

// 获取部门详情
DeptRespDTO dept = deptApi.getDept(deptId);

// 获取子部门列表
List<DeptRespDTO> childDepts = deptApi.getChildDeptList(deptId);
```

### 使用 PostApi

```java
@Resource
private PostApi postApi;

// 获取岗位详情
PostRespDTO post = postApi.getPost(postId);
```
