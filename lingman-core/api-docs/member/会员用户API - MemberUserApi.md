# 会员用户 API - MemberUserApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getUser`](#1-getuser)
  - [2. `getUserList`](#2-getuserlist)
  - [3. `getUserMap`](#3-getusermap)
  - [4. `getUserByMobile`](#4-getuserbymobile)
  - [5. `validateUser`](#5-validateuser)
- [参数 / 返回说明](#参数--返回说明)
  - [MemberUserRespDTO](#memberuserrespdto)
- [接口一览](#接口一览)

## 由来

`MemberUserApi` 定义在 **`com.lm.starter.module.member.api.user`** 包下，作为会员模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可查询会员用户信息、按手机号获取用户、并校验用户有效性，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，Starter 侧更偏向于作为**本地契约**使用：

- **查询方法**：直接返回 `MemberUserRespDTO` / `List<MemberUserRespDTO>` / `Map<Long, MemberUserRespDTO>`
- **校验方法**：以 `void + 异常` 表达校验结果（用户不存在/被禁用等情况由实现端抛异常；异常类型/错误码以 Starter 实现为准）

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-member</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：

   ```java
   import com.lm.starter.module.member.api.user.MemberUserApi;
   import com.lm.starter.module.member.api.user.dto.MemberUserRespDTO;
   import com.lm.stuff.service.member.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Map;
   import java.util.Set;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private MemberUserApi memberUserApi;

       public MemberUserRespDTO getUser(Long id) {
           return memberUserApi.getUser(id);
       }

       public MemberUserRespDTO getByMobile(String mobile) {
           return memberUserApi.getUserByMobile(mobile);
       }

       public Map<Long, MemberUserRespDTO> userMap(Set<Long> ids) {
           return memberUserApi.getUserMap(ids);
       }

       public void ensureValid(Long id) {
           // 不抛异常即校验通过
           memberUserApi.validateUser(id);
       }
   }
   ```

## 方法

### 1. `getUser`

**通过用户 ID 查询会员用户**。

```java
MemberUserRespDTO getUser(Long id);
```

### 2. `getUserList`

**通过用户 ID 集合查询会员用户列表**。

```java
List<MemberUserRespDTO> getUserList(Collection<Long> ids);
```

### 3. `getUserMap`

**获得用户 Map**（key 为 `userId`，value 为 `MemberUserRespDTO`），便于业务侧按用户编号快速索引用户信息。

> 说明：这是接口内置的 `default` 方法，内部调用 `getUserList(ids)` 并转换为 Map。

```java
default Map<Long, MemberUserRespDTO> getUserMap(Collection<Long> ids);
```

### 4. `getUserByMobile`

**通过手机号查询会员用户**。

```java
MemberUserRespDTO getUserByMobile(String mobile);
```

### 5. `validateUser`

**校验用户是否有效**。如下情况通常视为无效：用户编号不存在、用户被禁用。

> 说明：校验失败时由实现端抛出异常，校验通过则正常返回。

```java
void validateUser(Long id);
```

## 参数 / 返回说明

### MemberUserRespDTO

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 用户 ID |
| mobile | String | 手机号码 |
| nickname | String | 用户昵称 |
| avatar | String | 用户头像 |
| sex | Integer | 用户性别（0 未知 1 男 2 女） |
| status | Integer | 账号状态（0 正常 1 停用），参考 `CommonStatusEnum` |

## 接口一览

```java
public interface MemberUserApi {

    MemberUserRespDTO getUser(Long id);

    List<MemberUserRespDTO> getUserList(Collection<Long> ids);

    default Map<Long, MemberUserRespDTO> getUserMap(Collection<Long> ids) {
        List<MemberUserRespDTO> users = this.getUserList(ids);
        return CollectionUtils.convertMap(users, MemberUserRespDTO::getId);
    }

    MemberUserRespDTO getUserByMobile(String mobile);

    void validateUser(Long id);
}
```
