# 站内信发送 API - NotifyMessageSendApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `sendSingleMessageToAdmin`](#1-sendsinglemessagetoadmin)
  - [2. `sendSingleMessageToMember`](#2-sendsinglemessagetomember)
- [参数说明](#参数说明)
  - [NotifySendSingleToUserReqDTO](#notifysendsingletouserreqdto)
- [接口一览](#接口一览)

## 由来

`NotifyMessageSendApi` 定义在 **`com.lm.starter.module.system.api.notify`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可发送站内信，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<Long>`）不同，本接口方法**直接返回** `Long`，通常表示一次发送记录/消息编号（具体含义以 Starter 实现为准）；参数不合法将触发 `jakarta.validation` 校验。

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
   import com.lm.starter.module.system.api.notify.NotifyMessageSendApi;
   import com.lm.starter.module.system.api.notify.dto.NotifySendSingleToUserReqDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.Map;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private NotifyMessageSendApi notifyMessageSendApi;
   
       /** 发送站内信给 Admin（示例：以模板发送） */
       public Long sendToAdmin(Long adminUserId) {
           NotifySendSingleToUserReqDTO reqDTO = new NotifySendSingleToUserReqDTO();
           reqDTO.setUserId(adminUserId);
           reqDTO.setTemplateCode("USER_SEND");
           reqDTO.setTemplateParams(Map.of("name", "管理员"));
           return notifyMessageSendApi.sendSingleMessageToAdmin(reqDTO);
       }
   }
   ```

## 方法

### 1. `sendSingleMessageToAdmin`

**发送单条站内信给 Admin 用户**。

```java
Long sendSingleMessageToAdmin(@Valid NotifySendSingleToUserReqDTO reqDTO);
```

### 2. `sendSingleMessageToMember`

**发送单条站内信给 Member 用户**。

```java
Long sendSingleMessageToMember(@Valid NotifySendSingleToUserReqDTO reqDTO);
```

## 参数说明

### NotifySendSingleToUserReqDTO

- **userId**：用户编号（必填）
- **templateCode**：站内信模板编号（必填）
- **templateParams**：模板参数（可选，`Map<String, Object>`）

## 接口一览

```java
public interface NotifyMessageSendApi {
    Long sendSingleMessageToAdmin(@Valid NotifySendSingleToUserReqDTO reqDTO);

    Long sendSingleMessageToMember(@Valid NotifySendSingleToUserReqDTO reqDTO);
}
```