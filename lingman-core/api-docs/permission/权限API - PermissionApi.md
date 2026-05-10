# 权限 API - PermissionApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `hasAnyPermissions`](#1-hasanypermissions)
  - [2. `hasAnyRoles`](#2-hasanyroles)
  - [3. `getDeptDataPermission`](#3-getdeptdatapermission)
  - [4. `getUserRoleIdListByRoleIds`](#4-getuserroleidlistbyroleids)
- [参数 / 返回说明](#参数--返回说明)
  - [DeptDataPermissionRespDTO](#deptdatapermissionrespdto)
- [接口一览](#接口一览)

## 由来

系统模块对外提供的权限入口是 `PermissionApi`（**`com.lm.starter.module.system.api.permission`** 包）。它 **extends** `PermissionCommonApi`（**`com.lm.starter.framework.common.biz.system.permission`** 包），因此：

- **通用校验能力（继承）**：`hasAnyPermissions` / `hasAnyRoles` / `getDeptDataPermission` 定义在 `PermissionCommonApi` 中，并由 `PermissionApi` 通过继承对业务侧暴露
- **查询能力（扩展）**：`getUserRoleIdListByRoleIds` 定义在 `PermissionApi` 中，用于按角色编号集合查询相关用户的角色数据（见方法说明）

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，Starter 侧更偏向于作为**本地契约**使用：方法直接返回业务需要的数据结构（例如 `Boolean`、`Set<Long>`、`DeptDataPermissionRespDTO>`），避免 Feign/HTTP 语义渗透到业务代码。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-system</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：

   ```java
   import com.lm.starter.framework.common.biz.system.permission.dto.DeptDataPermissionRespDTO;
   import com.lm.starter.module.system.api.permission.PermissionApi;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.List;
   import java.util.Set;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private PermissionApi permissionApi;
   
       /** 判断是否具备任一权限 */
       public boolean canReadOrWrite(Long userId) {
           return permissionApi.hasAnyPermissions(userId, "read", "write");
       }
   
       /** 判断是否具备任一角色 */
       public boolean isAdminOrManager(Long userId) {
           return permissionApi.hasAnyRoles(userId, "admin", "manager");
       }
   
       /** 获取用户的部门数据权限 */
       public DeptDataPermissionRespDTO deptPermission(Long userId) {
           return permissionApi.getDeptDataPermission(userId);
       }
   
       /** 根据角色编号集合查询相关用户的角色数据 */
       public Set<Long> userRoleIdListByRoleIds(List<Long> roleIds) {
           return permissionApi.getUserRoleIdListByRoleIds(roleIds);
       }
   }
   ```

## 方法

### 1. `hasAnyPermissions`

**判断是否有权限（任一一个即可）**。当 `permissions` 中任一权限点满足时返回 `true`。

```java
Boolean hasAnyPermissions(Long userId, String... permissions);
```

### 2. `hasAnyRoles`

**判断是否有角色（任一一个即可）**。当 `roles` 中任一角色满足时返回 `true`。

```java
Boolean hasAnyRoles(Long userId, String... roles);
```

### 3. `getDeptDataPermission`

**获得登录用户的部门数据权限**，返回 `DeptDataPermissionRespDTO`。

```java
DeptDataPermissionRespDTO getDeptDataPermission(Long userId);
```

### 4. `getUserRoleIdListByRoleIds`

**获得拥有多个角色的用户角色编号集合**，入参为角色编号集合 `roleIds`。

> 说明：芋道云原版方法返回 `CommonResult<Set<Long>>`，Starter 本地契约版本直接返回 `Set<Long>`。

```java
Set<Long> getUserRoleIdListByRoleIds(Collection<Long> roleIds);
```

## 参数 / 返回说明

### DeptDataPermissionRespDTO

- **all**：是否可查看全部数据
- **self**：是否可查看自己的数据
- **deptIds**：可查看的部门编号集合

## 接口一览

```java
public interface PermissionApi extends PermissionCommonApi {
    Set<Long> getUserRoleIdListByRoleIds(Collection<Long> roleIds);
}
```