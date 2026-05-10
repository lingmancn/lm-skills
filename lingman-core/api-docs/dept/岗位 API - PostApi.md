# 岗位 API - PostApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `validPostList`](#1-validpostlist)
  - [2. `getPostList`](#2-getpostlist)
  - [3. `getPostMap`](#3-getpostmap)
- [接口一览](#接口一览)

## 由来

`PostApi` 定义在 **`com.lm.starter.module.system.api.dept`** 包下，作为系统模块对业务侧的**本地契约**：业务工程引入 Starter 后，直接注入该接口即可调用岗位查询与校验能力，**无需**经过 Feign 与 `CommonResult` 包装，便于在业务代码里像普通 Spring Bean 一样使用。

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，本接口方法**直接返回** `List<PostRespDTO>` / `Map<Long, PostRespDTO>` 或 `void`，由 Starter 内的实现类完成与 system 数据源的交互（或内聚实现）。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.starter</groupId>
   	<artifactId>lingman-module-system</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：无需再声明 Feign，直接按接口类型注入即可。

   ```java
   import com.lm.starter.module.system.api.dept.PostApi;
   import com.lm.starter.module.system.api.dept.dto.PostRespDTO;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.List;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private PostApi postApi;
   
       /** 按 id 批量查询岗位（示例：业务里按需封装即可） */
       public List<PostRespDTO> getPosts(List<Long> ids) {
           return postApi.getPostList(ids);
       }
   }
   ```

## 方法

### 1. `validPostList`

**校验岗位是否合法**，校验给定 id 集合是否**全部合法**（存在、启用等规则以实现端为准）。不通过时通常抛业务异常；返回类型为 `void`。

```java
void validPostList(Collection<Long> ids);
```

### 2. `getPostList`

**获得岗位信息数组**，根据**一批岗位编号**批量查询。适合列表展示、批量回填名称/编码等，避免对同一批 id 多次调用。

```java
List<PostRespDTO> getPostList(Collection<Long> ids);
```

### 3. `getPostMap`

**获得指定编号的岗位 Map**，在接口上的**默认实现**：

- 入参为空时返回空 Map（内部使用 `CollUtil.isEmpty` + `MapUtil.empty()`）
- 否则内部调用 `getPostList`，再用 `CollectionUtils.convertMap` 按 `PostRespDTO::getId` 转成 `Map<Long, PostRespDTO>`，便于按 id 快速查找

```java
default Map<Long, PostRespDTO> getPostMap(Collection<Long> ids) {
    if (CollUtil.isEmpty(ids)) {
        return MapUtil.empty();
    } else {
        List<PostRespDTO> list = this.getPostList(ids);
        return CollectionUtils.convertMap(list, PostRespDTO::getId);
    }
}
```

## 接口一览

```java
public interface PostApi {
    void validPostList(Collection<Long> ids);

    List<PostRespDTO> getPostList(Collection<Long> ids);

    default Map<Long, PostRespDTO> getPostMap(Collection<Long> ids) {
        if (CollUtil.isEmpty(ids)) {
            return MapUtil.empty();
        } else {
            List<PostRespDTO> list = this.getPostList(ids);
            return CollectionUtils.convertMap(list, PostRespDTO::getId);
        }
    }
}
```
