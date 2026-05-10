# 邮件发送 API - MailSendApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `sendSingleMailToAdmin`](#1-sendsinglemailtoadmin)
  - [2. `sendSingleMailToMember`](#2-sendsinglemailtomember)
- [参数说明](#参数说明)
  - [MailSendSingleToUserReqDTO](#mailsendsingletouserreqdto)
- [接口一览](#接口一览)

## 由来

`MailSendApi` 定义在 **`com.lm.starter.module.system.api.mail`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可发送邮件，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<Long>`）不同，本接口方法**直接返回** `Long`，通常表示一次发送记录/日志的编号（具体含义以 Starter 实现为准）；参数不合法将触发 `jakarta.validation` 校验。

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
   import com.lm.starter.module.system.api.mail.MailSendApi;
   import com.lm.starter.module.system.api.mail.dto.MailSendSingleToUserReqDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.Collections;
   import java.util.Map;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private MailSendApi mailSendApi;
   
       /** 发送邮件给 Admin（示例：以模板发送） */
       public Long sendToAdmin(Long adminUserId) {
           MailSendSingleToUserReqDTO reqDTO = new MailSendSingleToUserReqDTO();
           reqDTO.setUserId(adminUserId);
           reqDTO.setTemplateCode("WELCOME");
           reqDTO.setTemplateParams(Map.of("name", "管理员"));
           // 也可以显式设置收件人邮箱；不设置时，通常由实现端基于 userId 补全
           reqDTO.setToMails(Collections.emptyList());
           return mailSendApi.sendSingleMailToAdmin(reqDTO);
       }
   }
   ```

## 方法

### 1. `sendSingleMailToAdmin`

**发送单条邮件给 Admin 用户**。当 `toMails` 为空时，通常会使用 `userId` 加载对应 Admin 的邮箱并作为收件人（以实现端为准）。

```java
Long sendSingleMailToAdmin(@Valid MailSendSingleToUserReqDTO reqDTO);
```

### 2. `sendSingleMailToMember`

**发送单条邮件给 Member 用户**。当 `toMails` 为空时，通常会使用 `userId` 加载对应 Member 的邮箱并作为收件人（以实现端为准）。

```java
Long sendSingleMailToMember(@Valid MailSendSingleToUserReqDTO reqDTO);
```

## 参数说明

### MailSendSingleToUserReqDTO

- **userId**：用户编号（`Long`）；非空时加载对应用户的邮箱，添加到 `toMails` 中
- **toMails**：收件邮箱（`List<String>`）
- **ccMails**：抄送邮箱（`List<String>`）
- **bccMails**：密送邮箱（`List<String>`）
- **templateCode**：邮件模板编号（`String`，必填）
- **templateParams**：邮件模板参数（`Map<String, Object>`）
- **attachments**：附件（`File[]`）

## 接口一览

```java
public interface MailSendApi {
    Long sendSingleMailToAdmin(@Valid MailSendSingleToUserReqDTO reqDTO);

    Long sendSingleMailToMember(@Valid MailSendSingleToUserReqDTO reqDTO);
}
```
