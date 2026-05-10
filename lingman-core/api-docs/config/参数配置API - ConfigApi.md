# 参数配置 API - ConfigApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getConfigValueByKey`](#1-getconfigvaluebykey)
- [接口一览](#接口一览)

## 由来

`ConfigApi` 定义在 **`com.lm.starter.module.infra.api.config`** 包下，由基础设施模块（`lingman-module-infra`）提供，用于读取系统参数配置。业务工程引入 Starter 后，直接注入该接口即可查询配置值，**无需**经过 Feign 与 `CommonResult` 包装。

该接口适用于读取 `infra_config` 表中维护的全局参数（如开关、阈值、第三方 Key 等），避免在业务代码中直接依赖配置表。

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
   import com.lm.starter.module.infra.api.config.ConfigApi;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private ConfigApi configApi;

       /** 根据 key 获取配置值 */
       public String getConfig(String key) {
           return configApi.getConfigValueByKey(key);
       }
   }
   ```

## 方法

### 1. `getConfigValueByKey`

**根据参数键查询参数值**。从系统参数配置表中获取指定 key 对应的配置值。

```java
String getConfigValueByKey(String key);
```

- **key**：参数键（`String`）
- **返回**：参数值（`String`）

## 接口一览

```java
public interface ConfigApi {
    String getConfigValueByKey(String key);
}
```
