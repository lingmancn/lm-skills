# 角色 API - RoleApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `validRoleList`](#1-validrolelist)
- [接口一览](#接口一览)

## 由来

系统模块对外提供的角色校验入口是 `RoleApi`（**`com.lm.starter.module.system.api.permission`** 包）。

与芋道云原版（`@FeignClient`、`CommonResult<Boolean>`）不同，Starter 侧更偏向于作为**本地契约**使用：方法不再返回 `CommonResult`，而是以 **`void` + 异常** 的方式表达校验结果——当角色不合法时由实现端抛出业务异常；当不抛异常即代表校验通过。

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
   import com.lm.starter.module.system.api.permission.RoleApi;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.List;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private RoleApi roleApi;
   
       /** 校验角色编号集合是否合法 */
       public void validateRoleIds(List<Long> roleIds) {
           // 不抛异常即校验通过；异常类型/错误码以 Starter 实现为准
           roleApi.validRoleList(roleIds);
       }
   }
   ```

## 方法

### 1. `validRoleList`

**校验角色是否合法**。入参为角色编号集合 `ids`；校验失败时由实现端抛出异常，校验通过则正常返回。

```java
void validRoleList(Collection<Long> ids);
```

## 接口一览

```java
public interface RoleApi {
    void validRoleList(Collection<Long> ids);
}
```