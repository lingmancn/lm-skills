# 短信发送 API - SmsSendApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `sendSingleSmsToAdmin`](#1-sendsinglesmstoadmin)
  - [2. `sendSingleSmsToMember`](#2-sendsinglesmstomember)
- [参数说明](#参数说明)
  - [SmsSendSingleToUserReqDTO](#smssendsingletouserreqdto)
- [接口一览](#接口一览)

## 由来

`SmsSendApi` 定义在 **`com.lm.starter.module.system.api.sms`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可发送短信，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<Long>`）不同，本接口方法**直接返回** `Long`，通常表示一次发送记录/日志编号（具体含义以 Starter 实现为准）；参数不合法将触发 `jakarta.validation` 校验。

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
   import com.lm.starter.module.system.api.sms.SmsSendApi;
   import com.lm.starter.module.system.api.sms.dto.send.SmsSendSingleToUserReqDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.Collections;
   import java.util.Map;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private SmsSendApi smsSendApi;
   
       /** 发送短信给 Admin（示例：以模板发送） */
       public Long sendToAdmin(Long adminUserId, String mobile) {
           SmsSendSingleToUserReqDTO reqDTO = new SmsSendSingleToUserReqDTO();
           reqDTO.setUserId(adminUserId);
           // mobile 可选：为空时通常由实现端基于 userId 补全（以实现端为准）
           reqDTO.setMobile(mobile);
           reqDTO.setTemplateCode("USER_SEND");
           reqDTO.setTemplateParams(Map.of("name", "管理员"));
           return smsSendApi.sendSingleSmsToAdmin(reqDTO);
       }
   }
   ```

## 方法

### 1. `sendSingleSmsToAdmin`

**发送单条短信给 Admin 用户**。当 `mobile` 为空时，通常会使用 `userId` 加载对应 Admin 的手机号并作为收件人（以实现端为准）。

```java
Long sendSingleSmsToAdmin(@Valid SmsSendSingleToUserReqDTO reqDTO);
```

### 2. `sendSingleSmsToMember`

**发送单条短信给 Member 用户**。当 `mobile` 为空时，通常会使用 `userId` 加载对应 Member 的手机号并作为收件人（以实现端为准）。

```java
Long sendSingleSmsToMember(@Valid SmsSendSingleToUserReqDTO reqDTO);
```

## 参数说明

### SmsSendSingleToUserReqDTO

- **userId**：用户编号（可选；当 `mobile` 为空时通常需要提供，用于实现端补全手机号）
- **mobile**：手机号（可选；为空时通常由实现端基于 `userId` 补全）
- **templateCode**：短信模板编号（必填）
- **templateParams**：模板参数（可选，`Map<String, Object>`）

## 接口一览

```java
public interface SmsSendApi {
    Long sendSingleSmsToAdmin(@Valid SmsSendSingleToUserReqDTO reqDTO);

    Long sendSingleSmsToMember(@Valid SmsSendSingleToUserReqDTO reqDTO);
}
```