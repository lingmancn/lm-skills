# 部门 API - DeptApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getDept`](#1-getdept)
  - [2. `getDeptList`](#2-getdeptlist)
  - [3. `validateDeptList`](#3-validatedeptlist)
  - [4. `getDeptMap`](#4-getdeptmap)
  - [5. `getChildDeptList`](#5-getchilddeptlist)
- [接口一览](#接口一览)

## 由来

`DeptApi` 定义在 **`com.lm.starter.module.system.api.dept`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可调用部门查询与校验能力，**无需**经过 Feign 与 `CommonResult` 包装，便于在业务代码里像普通 Spring Bean 一样使用。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，本接口方法**直接返回** `DeptRespDTO`、`List<DeptRespDTO>` 或 `void`，由 Starter 内的实现类完成与 system 数据源的交互（或内聚实现）。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-system</artifactId>
   </dependency>
   ```
   
3. **在业务代码中注入并调用**：无需再声明 Feign，直接按接口类型注入即可。

   ```java
   import com.lm.starter.module.system.api.dept.DeptApi;
   import com.lm.starter.module.system.api.dept.dto.DeptRespDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private DeptApi deptApi;
   
       /** 按 id 查询部门（示例：业务里按需封装即可） */
       public DeptRespDTO getDept(Long id) {
           return deptApi.getDept(id);
       }
   }
   ```

## 方法

### 1. `getDept`

**获得部门信息**，根据**单个部门编号**获取部门详情。

```java
DeptRespDTO getDept(Long id);
```

### 2. `getDeptList`

**获得部门信息数组**，根据**一批部门编号**批量查询。适合列表展示、批量回填名称等，避免对同一批 id 多次调用 `getDept`。

```java
List<DeptRespDTO> getDeptList(Collection<Long> ids);
```

### 3. `validateDeptList`

**校验部门是否合法**，校验给定 id 集合是否**全部合法**（存在、启用等规则以实现端为准）。不通过时通常抛业务异常；返回类型为 `void`。

```java
void validateDeptList(Collection<Long> ids);
```

### 4. `getDeptMap`

**获得指定编号的部门 Map**，在接口上的**默认实现**：内部调用 `getDeptList`，再用 `CollectionUtils.convertMap` 按 `DeptRespDTO::getId` 转成 `Map<Long, DeptRespDTO>`，便于按 id 快速查找。

```java
default Map<Long, DeptRespDTO> getDeptMap(Collection<Long> ids) {
    List<DeptRespDTO> list = this.getDeptList(ids);
    return CollectionUtils.convertMap(list, DeptRespDTO::getId);
}
```

### 5. `getChildDeptList`

**获得指定部门的所有子部门**，查询**指定部门下的子部门列表**。是否仅直接子级、是否包含多级，以实现类为准。

```java
List<DeptRespDTO> getChildDeptList(Long id);
```

## 接口一览

```java
public interface DeptApi {
    DeptRespDTO getDept(Long id);

    List<DeptRespDTO> getDeptList(Collection<Long> ids);

    void validateDeptList(Collection<Long> ids);

    default Map<Long, DeptRespDTO> getDeptMap(Collection<Long> ids) {
        List<DeptRespDTO> list = this.getDeptList(ids);
        return CollectionUtils.convertMap(list, DeptRespDTO::getId);
    }

    List<DeptRespDTO> getChildDeptList(Long id);
}
```
