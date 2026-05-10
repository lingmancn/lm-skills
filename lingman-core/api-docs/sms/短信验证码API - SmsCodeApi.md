# 短信验证码 API - SmsCodeApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `sendSmsCode`](#1-sendsmscode)
  - [2. `useSmsCode`](#2-usesmscode)
  - [3. `validateSmsCode`](#3-validatesmscode)
- [参数说明](#参数说明)
  - [SmsCodeSendReqDTO](#smscodesendreqdto)
  - [SmsCodeUseReqDTO](#smscodeusereqdto)
  - [SmsCodeValidateReqDTO](#smscodevalidatereqdto)
- [接口一览](#接口一览)

## 由来

`SmsCodeApi` 定义在 **`com.lm.starter.module.system.api.sms`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可完成短信验证码的发送、校验与使用，**无需**经过 Feign 与 `CommonResult` 包装。

与芋道云原版（`@FeignClient`、`CommonResult<Boolean>`）不同，Starter 侧更偏向于以 **`void + 异常`** 的方式表达处理结果：

- **正常返回**：表示执行成功（发送成功 / 校验通过 / 使用成功）
- **抛出异常**：表示执行失败（参数不合法、验证码错误/过期、场景不匹配、频率限制等；异常类型/错误码以 Starter 实现为准）

同时，入参上标注的 `@Valid` 会触发 `jakarta.validation` 校验。

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
   import com.lm.starter.module.system.api.sms.SmsCodeApi;
   import com.lm.starter.module.system.api.sms.dto.code.SmsCodeSendReqDTO;
   import com.lm.starter.module.system.api.sms.dto.code.SmsCodeUseReqDTO;
   import com.lm.starter.module.system.api.sms.dto.code.SmsCodeValidateReqDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private SmsCodeApi smsCodeApi;
   
       /** 发送验证码 */
       public void sendCode(String mobile, Integer scene, String ip) {
           SmsCodeSendReqDTO reqDTO = new SmsCodeSendReqDTO();
           reqDTO.setMobile(mobile);
           reqDTO.setScene(scene);
           reqDTO.setCreateIp(ip);
           smsCodeApi.sendSmsCode(reqDTO);
       }
   
       /** 校验验证码（不消费） */
       public void validateCode(String mobile, Integer scene, String code) {
           SmsCodeValidateReqDTO reqDTO = new SmsCodeValidateReqDTO();
           reqDTO.setMobile(mobile);
           reqDTO.setScene(scene);
           reqDTO.setCode(code);
           smsCodeApi.validateSmsCode(reqDTO);
       }
   
       /** 使用验证码（校验并消费） */
       public void useCode(String mobile, Integer scene, String code, String ip) {
           SmsCodeUseReqDTO reqDTO = new SmsCodeUseReqDTO();
           reqDTO.setMobile(mobile);
           reqDTO.setScene(scene);
           reqDTO.setCode(code);
           reqDTO.setUsedIp(ip);
           smsCodeApi.useSmsCode(reqDTO);
       }
   }
   ```

## 方法

### 1. `sendSmsCode`

**创建短信验证码并发送**。常用于登录、注册、找回密码等短信验证码场景。

```java
void sendSmsCode(@Valid SmsCodeSendReqDTO reqDTO);
```

### 2. `useSmsCode`

**验证短信验证码并进行使用（消费）**。通常表示“验证码校验通过并标记为已使用”，避免重复使用。

```java
void useSmsCode(@Valid SmsCodeUseReqDTO reqDTO);
```

### 3. `validateSmsCode`

**检查验证码是否有效（不消费）**。用于仅校验，不改变验证码使用状态（以实现端为准）。

```java
void validateSmsCode(@Valid SmsCodeValidateReqDTO reqDTO);
```

## 参数说明

### SmsCodeSendReqDTO

- **mobile**：手机号（必填）
- **scene**：发送场景（必填，枚举值来源于 `SmsSceneEnum`）
- **createIp**：发送 IP（必填）

### SmsCodeUseReqDTO

- **mobile**：手机号（必填）
- **scene**：发送场景（必填，枚举值来源于 `SmsSceneEnum`）
- **code**：验证码（必填）
- **usedIp**：使用 IP（必填）

### SmsCodeValidateReqDTO

- **mobile**：手机号（必填）
- **scene**：发送场景（必填，枚举值来源于 `SmsSceneEnum`）
- **code**：验证码（必填）

## 接口一览

```java
public interface SmsCodeApi {
    void sendSmsCode(@Valid SmsCodeSendReqDTO reqDTO);

    void useSmsCode(@Valid SmsCodeUseReqDTO reqDTO);

    void validateSmsCode(@Valid SmsCodeValidateReqDTO reqDTO);
}
```