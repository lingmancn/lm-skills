# WebSocket 发送 API - WebSocketSenderApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `send`](#1-send)
  - [2. `sendObject`](#2-sendobject)
- [接口一览](#接口一览)

## 由来

`WebSocketSenderApi` 定义在 **`com.lm.starter.module.infra.api.websocket`** 包下，由基础设施模块（`lingman-module-infra`）提供，作为 WebSocket 消息推送的统一入口。它是对内部 `WebSocketMessageSender` 的封装，提供给其它模块使用。

业务工程引入 Starter 后，直接注入该接口即可向指定用户、用户类型或 Session 推送消息，**无需**经过 Feign 与 `CommonResult` 包装。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-infra</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：

   ```java
   import com.lm.starter.module.infra.api.websocket.WebSocketSenderApi;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private WebSocketSenderApi webSocketSenderApi;

       /** 推送消息给指定用户 */
       public void sendToUser(Integer userType, Long userId, String messageType, String content) {
           webSocketSenderApi.send(userType, userId, messageType, content);
       }

       /** 推送对象消息（自动序列化为 JSON） */
       public void sendObjectToUser(Integer userType, Long userId, String messageType, Object data) {
           webSocketSenderApi.sendObject(userType, userId, messageType, data);
       }
   }
   ```

## 方法

### 1. `send`

**发送消息**。提供三个重载，分别针对不同目标：

```java
// 发送给指定用户
void send(Integer userType, Long userId, String messageType, String messageContent);

// 发送给指定用户类型的所有用户
void send(Integer userType, String messageType, String messageContent);

// 发送给指定 Session
void send(String sessionId, String messageType, String messageContent);
```

- **userType**：用户类型（`Integer`）
- **userId**：用户编号（`Long`）
- **sessionId**：Session 编号（`String`）
- **messageType**：消息类型（`String`），由业务自定义，用于前端区分消息类别
- **messageContent**：消息内容（`String`），JSON 格式

### 2. `sendObject`

**发送对象消息**（默认方法）。将 Java 对象自动序列化为 JSON 后发送，是对 `send` 的便捷封装。

```java
default void sendObject(Integer userType, Long userId, String messageType, Object messageContent) {
    send(userType, userId, messageType, JsonUtils.toJsonString(messageContent));
}

default void sendObject(Integer userType, String messageType, Object messageContent) {
    send(userType, messageType, JsonUtils.toJsonString(messageContent));
}

default void sendObject(String sessionId, String messageType, Object messageContent) {
    send(sessionId, messageType, JsonUtils.toJsonString(messageContent));
}
```

## 接口一览

```java
public interface WebSocketSenderApi {
    void send(Integer userType, Long userId, String messageType, String messageContent);

    void send(Integer userType, String messageType, String messageContent);

    void send(String sessionId, String messageType, String messageContent);

    default void sendObject(Integer userType, Long userId, String messageType, Object messageContent) {
        send(userType, userId, messageType, JsonUtils.toJsonString(messageContent));
    }

    default void sendObject(Integer userType, String messageType, Object messageContent) {
        send(userType, messageType, JsonUtils.toJsonString(messageContent));
    }

    default void sendObject(String sessionId, String messageType, Object messageContent) {
        send(sessionId, messageType, JsonUtils.toJsonString(messageContent));
    }
}
```
