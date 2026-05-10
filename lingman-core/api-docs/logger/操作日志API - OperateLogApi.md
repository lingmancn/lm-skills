# 操作日志 API - OperateLogApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `createOperateLog`](#1-createoperatelog)
  - [2. `createOperateLogAsync`](#2-createoperatelogasync)
  - [3. `getOperateLogPage`](#3-getoperatelogpage)
- [参数说明](#参数说明)
  - [OperateLogCreateReqDTO](#operatelogcreatereqdto)
  - [OperateLogPageReqDTO](#operatelogpagereqdto)
  - [OperateLogRespDTO](#operatelogrespdto)
- [接口一览](#接口一览)

## 由来

系统模块对外提供的操作日志入口是 `OperateLogApi`（**`com.lm.starter.module.system.api.logger`** 包）。它 **extends** `OperateLogCommonApi`（**`com.lm.starter.framework.common.biz.system.logger`** 包），因此：

- **写入能力（通用）**：`createOperateLog` / `createOperateLogAsync` 定义在 `OperateLogCommonApi` 中，并由 `OperateLogApi` 通过继承对业务侧暴露
- **查询能力（扩展）**：`getOperateLogPage` 定义在 `OperateLogApi` 中，用于按条件分页查询操作日志

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，Starter 侧更偏向于作为**本地契约**使用：方法直接返回业务需要的数据结构（例如 `PageResult<OperateLogRespDTO>`）或 `void`，避免 Feign/HTTP 语义渗透到业务代码。

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
   import com.lm.starter.framework.common.biz.system.logger.dto.OperateLogCreateReqDTO;
   import com.lm.starter.framework.common.pojo.PageResult;
   import com.lm.starter.module.system.api.logger.OperateLogApi;
   import com.lm.starter.module.system.api.logger.dto.OperateLogPageReqDTO;
   import com.lm.starter.module.system.api.logger.dto.OperateLogRespDTO;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private OperateLogApi operateLogApi;
   
       /** 写入操作日志（同步） */
       public void createOperateLog(Long userId, Integer userType) {
           OperateLogCreateReqDTO reqDTO = new OperateLogCreateReqDTO();
           reqDTO.setUserId(userId);
           reqDTO.setUserType(userType);
           reqDTO.setType("订单");
           reqDTO.setSubType("创建订单");
           reqDTO.setBizId(188L);
           reqDTO.setAction("创建编号为 188 的订单");
           reqDTO.setRequestMethod("POST");
           reqDTO.setRequestUrl("/order/create");
           reqDTO.setUserIp("127.0.0.1");
           reqDTO.setUserAgent("Mozilla/5.0");
           operateLogApi.createOperateLog(reqDTO);
       }
   
       /** 写入操作日志（异步，默认实现会调用 createOperateLog） */
       public void createOperateLogAsync(Long userId, Integer userType) {
           OperateLogCreateReqDTO reqDTO = new OperateLogCreateReqDTO();
           reqDTO.setUserId(userId);
           reqDTO.setUserType(userType);
           reqDTO.setType("订单");
           reqDTO.setSubType("创建订单");
           reqDTO.setBizId(188L);
           reqDTO.setAction("创建编号为 188 的订单");
           reqDTO.setRequestMethod("POST");
           reqDTO.setRequestUrl("/order/create");
           reqDTO.setUserIp("127.0.0.1");
           reqDTO.setUserAgent("Mozilla/5.0");
           operateLogApi.createOperateLogAsync(reqDTO);
       }
   
       /** 分页查询操作日志 */
       public PageResult<OperateLogRespDTO> pageOperateLog(String type, Long bizId) {
           OperateLogPageReqDTO pageReqDTO = new OperateLogPageReqDTO();
           pageReqDTO.setType(type);
           pageReqDTO.setBizId(bizId);
           pageReqDTO.setPageNo(1);
           pageReqDTO.setPageSize(10);
           return operateLogApi.getOperateLogPage(pageReqDTO);
       }
   }
   ```

## 方法

### 1. `createOperateLog`

**创建操作日志（同步）**，传入 `OperateLogCreateReqDTO` 写入操作日志。若参数不合法将触发 `jakarta.validation` 的校验。

```java
void createOperateLog(@Valid OperateLogCreateReqDTO createReqDTO);
```

### 2. `createOperateLogAsync`

**创建操作日志（异步）**，在 `OperateLogCommonApi` 的默认实现中通过 `@Async` 异步执行，内部调用 `createOperateLog`。

```java
@Async
default void createOperateLogAsync(OperateLogCreateReqDTO createReqDTO) {
    this.createOperateLog(createReqDTO);
}
```

### 3. `getOperateLogPage`

**分页查询操作日志**，按条件（例如模块类型、业务编号、用户编号等）返回操作日志分页数据。

```java
PageResult<OperateLogRespDTO> getOperateLogPage(OperateLogPageReqDTO pageReqDTO);
```

## 参数说明

### OperateLogCreateReqDTO

- **traceId**：链路追踪编号（`String`）
- **userId**：用户编号（`Long`，必填）
- **userType**：用户类型（`Integer`，必填），枚举 `UserTypeEnum`
- **type**：操作模块类型（`String`，必填）
- **subType**：操作名（`String`，必填）
- **bizId**：操作模块业务编号（`Long`，必填）
- **action**：操作内容，记录整个操作的明细（`String`，必填）
- **extra**：拓展字段，JSON 格式（`String`）
- **requestMethod**：请求方法名（`String`，必填）
- **requestUrl**：请求地址（`String`，必填）
- **userIp**：用户 IP（`String`，必填）
- **userAgent**：浏览器 UA（`String`，必填）

### OperateLogPageReqDTO

继承 `PageParam`，额外包含以下字段：

- **type**：模块类型（`String`）
- **bizId**：模块数据编号（`Long`）
- **userId**：用户编号（`Long`）

### OperateLogRespDTO

- **id**：日志编号（`Long`）
- **traceId**：链路追踪编号（`String`）
- **userId**：用户编号（`Long`）
- **userName**：用户名称（`String`），通过 `@Trans` 翻译
- **userType**：用户类型（`Integer`）
- **type**：操作模块类型（`String`）
- **subType**：操作名（`String`）
- **bizId**：操作模块业务编号（`Long`）
- **action**：操作内容（`String`）
- **extra**：拓展字段（`String`）
- **requestMethod**：请求方法名（`String`）
- **requestUrl**：请求地址（`String`）
- **userIp**：用户 IP（`String`）
- **userAgent**：浏览器 UA（`String`）
- **createTime**：创建时间（`LocalDateTime`）

## 接口一览

```java
public interface OperateLogCommonApi {
    void createOperateLog(@Valid OperateLogCreateReqDTO createReqDTO);

    @Async
    default void createOperateLogAsync(OperateLogCreateReqDTO createReqDTO) {
        this.createOperateLog(createReqDTO);
    }
}
```

```java
public interface OperateLogApi extends OperateLogCommonApi {
    PageResult<OperateLogRespDTO> getOperateLogPage(OperateLogPageReqDTO pageReqDTO);
}
```
