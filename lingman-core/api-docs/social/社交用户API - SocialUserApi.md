# 社交用户 API - SocialUserApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `bindSocialUser`](#1-bindsocialuser)
  - [2. `unbindSocialUser`](#2-unbindsocialuser)
  - [3. `getSocialUserByUserId`](#3-getsocialuserbyuserid)
  - [4. `getSocialUserByCode`](#4-getsocialuserbycode)
- [参数 / 返回说明](#参数--返回说明)
  - [SocialUserBindReqDTO](#socialuserbindreqdto)
  - [SocialUserUnbindReqDTO](#socialuserunbindreqdto)
  - [SocialUserRespDTO](#socialuserrespdto)
- [接口一览](#接口一览)

## 由来

`SocialUserApi` 定义在 **`com.lm.starter.module.system.api.social`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可完成社交用户（以微信生态为主）的绑定、解绑与查询，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，本接口方法**直接返回**所需的数据结构（例如 `String`、`SocialUserRespDTO`），或在成功时返回 `void`；失败时以异常表达（例如授权信息不正确、参数不合法等；异常类型/错误码以 Starter 实现为准）。入参上标注的 `@Valid` 会触发 `jakarta.validation` 校验。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-system</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**（示例：绑定 + 查询）：

   ```java
   import com.lm.starter.module.system.api.social.SocialUserApi;
   import com.lm.starter.module.system.api.social.dto.SocialUserBindReqDTO;
   import com.lm.starter.module.system.api.social.dto.SocialUserRespDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private SocialUserApi socialUserApi;
   
       /** 绑定社交用户，返回 openid（以实现端为准） */
       public String bind(Long userId, Integer userType, Integer socialType, String code, String state) {
           SocialUserBindReqDTO reqDTO = new SocialUserBindReqDTO(userId, userType, socialType, code, state);
           return socialUserApi.bindSocialUser(reqDTO);
       }
   
       /** 根据 userId 查询已绑定的社交用户信息 */
       public SocialUserRespDTO getByUserId(Integer userType, Long userId, Integer socialType) {
           return socialUserApi.getSocialUserByUserId(userType, userId, socialType);
       }
   }
   ```

## 方法

### 1. `bindSocialUser`

**绑定社交用户**。通常用于 OAuth 登录/绑定流程：用 `code/state` 换取社交用户身份，并与业务侧用户进行关联。

```java
String bindSocialUser(@Valid SocialUserBindReqDTO reqDTO);
```

### 2. `unbindSocialUser`

**取消绑定社交用户**。取消用户与某个社交账号（例如微信 openid）的关联关系。

```java
void unbindSocialUser(@Valid SocialUserUnbindReqDTO reqDTO);
```

### 3. `getSocialUserByUserId`

**获得社交用户（基于 userId）**。

```java
SocialUserRespDTO getSocialUserByUserId(Integer userType, Long userId, Integer socialType);
```

### 4. `getSocialUserByCode`

**获得社交用户（基于授权码 code/state）**。在授权信息不正确等情况下，通常会抛出业务异常（以 Starter 实现为准）。

```java
SocialUserRespDTO getSocialUserByCode(Integer userType, Integer socialType, String code, String state);
```

## 参数 / 返回说明

### SocialUserBindReqDTO

- **userId**：用户编号（必填）
- **userType**：用户类型（必填）
- **socialType**：社交平台类型（必填）
- **code**：授权码（必填）
- **state**：state（必填）

### SocialUserUnbindReqDTO

- **userId**：用户编号（必填）
- **userType**：用户类型（必填）
- **socialType**：社交平台类型（必填）
- **openid**：社交平台 openid（必填）

### SocialUserRespDTO

- **openid**：社交用户 openid
- **nickname**：社交用户昵称
- **avatar**：社交用户头像
- **userId**：关联的用户编号

## 接口一览

```java
public interface SocialUserApi {
    String bindSocialUser(@Valid SocialUserBindReqDTO reqDTO);

    void unbindSocialUser(@Valid SocialUserUnbindReqDTO reqDTO);

    SocialUserRespDTO getSocialUserByUserId(Integer userType, Long userId, Integer socialType);

    SocialUserRespDTO getSocialUserByCode(Integer userType, Integer socialType, String code, String state);
}
```