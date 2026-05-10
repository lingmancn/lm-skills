# 管理员用户 API - AdminUserApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getUser`](#1-getuser)
  - [2. `getUserListBySubordinate`](#2-getuserlistbysubordinate)
  - [3. `getUserList`](#3-getuserlist)
  - [4. `getUserListByDeptIds`](#4-getuserlistbydeptids)
  - [5. `getUserListByPostIds`](#5-getuserlistbypostids)
  - [6. `getUserMap`](#6-getusermap)
  - [7. `validateUser`](#7-validateuser)
  - [8. `validateUserList`](#8-validateuserlist)
- [参数 / 返回说明](#参数--返回说明)
  - [AdminUserRespDTO](#adminuserrespdto)
- [接口一览](#接口一览)

## 由来

`AdminUserApi` 定义在 **`com.lm.starter.module.system.api.user`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可查询管理员用户信息、按部门/岗位/上下级关系获取用户列表、并校验用户有效性，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，Starter 侧更偏向于作为**本地契约**使用：

- **查询方法**：直接返回 `AdminUserRespDTO` / `List<AdminUserRespDTO>` / `Map<Long, AdminUserRespDTO>`
- **校验方法**：以 `void + 异常` 表达校验结果（用户不存在/被禁用等情况由实现端抛异常；异常类型/错误码以 Starter 实现为准）

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
   import com.lm.starter.module.system.api.user.AdminUserApi;
   import com.lm.starter.module.system.api.user.dto.AdminUserRespDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.List;
   import java.util.Map;
   import java.util.Set;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private AdminUserApi adminUserApi;
   
       public AdminUserRespDTO getUser(Long id) {
           return adminUserApi.getUser(id);
       }
   
       public List<AdminUserRespDTO> listByDeptIds(Set<Long> deptIds) {
           return adminUserApi.getUserListByDeptIds(deptIds);
       }
   
       public Map<Long, AdminUserRespDTO> userMap(Set<Long> ids) {
           return adminUserApi.getUserMap(ids);
       }
   
       public void ensureEnabled(Long id) {
           // 不抛异常即校验通过
           adminUserApi.validateUser(id);
       }
   }
   ```

## 方法

### 1. `getUser`

**通过用户 ID 查询用户**。

```java
AdminUserRespDTO getUser(Long id);
```

### 2. `getUserListBySubordinate`

**通过用户 ID 查询用户下属**。

```java
List<AdminUserRespDTO> getUserListBySubordinate(Long id);
```

### 3. `getUserList`

**通过用户 ID 查询用户们**。

```java
List<AdminUserRespDTO> getUserList(Collection<Long> ids);
```

### 4. `getUserListByDeptIds`

**获得指定部门的用户数组**。

```java
List<AdminUserRespDTO> getUserListByDeptIds(Collection<Long> deptIds);
```

### 5. `getUserListByPostIds`

**获得指定岗位的用户数组**。

```java
List<AdminUserRespDTO> getUserListByPostIds(Collection<Long> postIds);
```

### 6. `getUserMap`

**获得用户 Map**（key 为 `userId`，value 为 `AdminUserRespDTO`），便于业务侧按用户编号快速索引用户信息。

> 说明：这是接口内置的 `default` 方法，内部调用 `getUserList(ids)` 并转换为 Map。

```java
default Map<Long, AdminUserRespDTO> getUserMap(Collection<Long> ids);
```

### 7. `validateUser`

**校验用户是否有效**。如下情况通常视为无效：用户编号不存在、用户被禁用。

> 说明：这是接口内置的 `default` 方法，内部调用 `validateUserList(Collections.singleton(id))`。

```java
default void validateUser(Long id);
```

### 8. `validateUserList`

**校验用户们是否有效**。校验失败时由实现端抛出异常，校验通过则正常返回。

```java
void validateUserList(Collection<Long> ids);
```

## 参数 / 返回说明

### AdminUserRespDTO

- **id**：用户 ID
- **nickname**：用户昵称
- **status**：账号状态（通常参考 `CommonStatusEnum`）
- **deptId**：部门编号
- **postIds**：岗位编号集合
- **mobile**：手机号码
- **avatar**：用户头像

## 接口一览

```java
public interface AdminUserApi {
    AdminUserRespDTO getUser(Long id);

    List<AdminUserRespDTO> getUserListBySubordinate(Long id);

    List<AdminUserRespDTO> getUserList(Collection<Long> ids);

    List<AdminUserRespDTO> getUserListByDeptIds(Collection<Long> deptIds);

    List<AdminUserRespDTO> getUserListByPostIds(Collection<Long> postIds);

    default Map<Long, AdminUserRespDTO> getUserMap(Collection<Long> ids) {
        List<AdminUserRespDTO> users = this.getUserList(ids);
        return CollectionUtils.convertMap(users, AdminUserRespDTO::getId);
    }

    default void validateUser(Long id) {
        this.validateUserList(Collections.singleton(id));
    }

    void validateUserList(Collection<Long> ids);
}
```