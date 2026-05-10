# 登入日志 API - LoginLogApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `createLoginLog`](#1-createloginlog)
- [参数说明](#参数说明)
  - [LoginLogCreateReqDTO](#loginlogcreatereqdto)
- [接口一览](#接口一览)

## 由来

`LoginLogApi` 定义在 **`com.lm.starter.module.system.api.logger`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可创建登录日志，**无需**经过 Feign 与 `CommonResult` 包装，便于在业务代码里像普通 Spring Bean 一样使用。

与芋道云原版（`@FeignClient`、`CommonResult<Boolean>`）不同，本接口方法为 `void`，由 Starter 内的实现类负责落库/写入日志等具体行为；若参数不合法将触发 `jakarta.validation` 的校验。

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
   import com.lm.starter.module.system.api.logger.LoginLogApi;
   import com.lm.starter.module.system.api.logger.dto.LoginLogCreateReqDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private LoginLogApi loginLogApi;
   
       public void recordLoginSuccess(Long userId, Integer userType, String username, String userIp) {
           // LoginLogCreateReqDTO 支持链式 setter（由 Lombok 生成 / 反编译后可见）
           LoginLogCreateReqDTO reqDTO = new LoginLogCreateReqDTO()
                   .setLogType(1)
                   .setUserId(userId)
                   .setUserType(userType)
                   .setUsername(username)
                   .setResult(1)
                   .setUserIp(userIp);
           loginLogApi.createLoginLog(reqDTO);
       }
   }
   ```

## 方法

### 1. `createLoginLog`

**创建登录日志**，传入 `LoginLogCreateReqDTO` 写入登录日志。

```java
void createLoginLog(@Valid LoginLogCreateReqDTO reqDTO);
```

## 参数说明

### LoginLogCreateReqDTO

- **logType**：日志类型（`Integer`，必填）
- **traceId**：链路追踪编号（`String`）
- **userId**：用户编号（`Long`）
- **userType**：用户类型（`Integer`，必填）
- **username**：用户账号（`String`）
- **result**：登录结果（`Integer`，必填）
- **userIp**：用户 IP（`String`，必填）
- **userAgent**：浏览器 UserAgent（`String`）

## 接口一览

```java
public interface LoginLogApi {
    void createLoginLog(@Valid LoginLogCreateReqDTO reqDTO);
}
```
