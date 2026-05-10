# 通知分发 API - NotificationDispatchApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `send`](#1-send)
- [参数 / 返回说明](#参数--返回说明)
  - [NotificationDispatchReqDTO](#notificationdispatchreqdto)
  - [NotificationDispatchRespDTO](#notificationdispatchrespdto)
  - [NotificationDispatchResultDTO](#notificationdispatchresultdto)
- [接口一览](#接口一览)

## 由来

`NotificationDispatchApi` 定义在 **`com.lm.starter.module.system.api.notify`** 包下，作为系统模块对业务侧的**统一通知分发入口**。业务工程引入 Starter 后，直接注入该接口即可发送通知，**无需**经过 Feign 与 `CommonResult` 包装。

该接口的核心设计理念是：**调用方不需要关心通知具体走短信、邮件、站内信还是业务自定义渠道**。调用方只需提供接收人、模板标识、业务语义负载，通知编排层会根据租户启用的策略列表，结合负载解析器，自动转换成各个渠道真正需要的模板参数。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，本接口方法**直接返回**业务数据结构，避免 Feign/HTTP 语义渗透到业务代码。

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
   import com.lm.starter.module.system.api.notify.NotificationDispatchApi;
   import com.lm.starter.module.system.api.notify.dto.NotificationDispatchReqDTO;
   import com.lm.starter.module.system.api.notify.dto.NotificationDispatchRespDTO;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private NotificationDispatchApi notificationDispatchApi;

       /** 统一分发通知（示例：按模板发送给指定用户） */
       public NotificationDispatchRespDTO sendNotification(Long userId, Integer userType, String templateCode) {
           NotificationDispatchReqDTO reqDTO = new NotificationDispatchReqDTO();
           reqDTO.setUserId(userId);
           reqDTO.setUserType(userType);
           reqDTO.setTemplateCode("ORDER_CREATED");
           // payload 和 strategyCodes 按业务需求设置
           return notificationDispatchApi.send(reqDTO);
       }
   }
   ```

## 方法

### 1. `send`

**按租户启用的通知策略统一分发通知**。根据传入的用户、模板和负载，自动匹配当前租户的启用策略（短信、邮件、站内信等），并将通知分发到各个渠道。

> 可以通过 `strategyCodes` 限制本次只使用部分策略；为空时使用租户全部已启用策略。

```java
NotificationDispatchRespDTO send(@Valid NotificationDispatchReqDTO reqDTO);
```

## 参数 / 返回说明

### NotificationDispatchReqDTO

- **userId**：通知接收用户编号（`Long`，必填）
- **userType**：通知接收用户类型（`Integer`，必填）
- **templateCode**：业务模板编码（`String`，必填）
- **payload**：业务通知负载（`NotificationPayload`），推荐由业务模块定义强类型 payload，各渠道分别实现对应的 `NotificationPayloadResolver`
- **strategyCodes**：本次允许执行的策略编码集合（`List<String>`）；为空时按当前租户全部已启用策略发送；不为空时仅在租户已启用策略中取交集

### NotificationDispatchRespDTO

- **success**：是否整体成功（`Boolean`）
- **results**：策略执行结果列表（`List<NotificationDispatchResultDTO>`）

### NotificationDispatchResultDTO

- **strategyCode**：策略编码（`String`）
- **success**：是否成功（`Boolean`）
- **resultId**：业务结果编号（`Long`），如短信/邮件发送记录 ID
- **errorMessage**：错误信息（`String`）

## 接口一览

```java
public interface NotificationDispatchApi {
    NotificationDispatchRespDTO send(@Valid NotificationDispatchReqDTO reqDTO);
}
```
