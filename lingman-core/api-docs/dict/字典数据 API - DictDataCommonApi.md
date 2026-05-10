# 字典数据 API - DictDataCommonApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `getDictDataList`](#1-getdictdatalist)
  - [2. `validateDictDataList`](#2-validatedictdatalist)
- [接口一览](#接口一览)

## 由来

系统模块对外提供的字典数据入口是 `DictDataApi`（**`com.lm.starter.module.system.api.dict`** 包）。它 **extends** `DictDataCommonApi`（**`com.lm.starter.framework.common.biz.system.dict`** 包）

- **查询能力**：`getDictDataList` 定义在 `DictDataCommonApi` 中，并由 `DictDataApi` 通过继承对业务侧暴露
- **校验能力**：`validateDictDataList` 定义在 `DictDataApi` 中，用于扩展对字典值集合的合法性校验

与芋道云原版（`@FeignClient`、`CommonResult<T>`）不同，Starter 侧更偏向于作为**本地契约**使用：方法直接返回业务需要的数据结构（例如 `List<DictDataRespDTO>`），避免 Feign/HTTP 语义渗透到业务代码。

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
   import com.lm.starter.framework.common.biz.system.dict.dto.DictDataRespDTO;
   import com.lm.starter.module.system.api.dict.DictDataApi;
   import com.lm.stuff.service.admin.YourBizService;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;
   
   import java.util.Arrays;
   import java.util.List;
   import java.util.stream.Collectors;
   
   @Service
   public class YourBizServiceImpl implements YourBizService {
   
       @Resource
       private DictDataApi dictDataApi;
   
       /** 按字典类型获取字典数据列表（示例：业务里按需封装即可） */
       public List<DictDataRespDTO> listSexDict() {
           return dictDataApi.getDictDataList("SEX");
       }
   
       /** 校验一组字典值是否有效（示例：不通过通常抛业务异常） */
       public void validateSexValues(String... values) {
           dictDataApi.validateDictDataList("SEX",
                   Arrays.stream(values).collect(Collectors.toList()));
       }
   }
   ```

## 方法

### 1. `getDictDataList`

**获得指定字典类型的字典数据列表**，根据 `dictType` 返回该类型下的字典数据数组（例如用于下拉选项）。

```java
List<DictDataRespDTO> getDictDataList(String dictType);
```

### 2. `validateDictDataList`

**校验字典数据值是否有效**，校验指定 `dictType` 下的一组 `values` 是否全部合法（是否存在、是否启用等规则以实现端为准）。不通过时通常抛业务异常。

```java
void validateDictDataList(String dictType, Collection<String> values);
```

## 接口一览

```java
public interface DictDataCommonApi {
    List<DictDataRespDTO> getDictDataList(String dictType);
}
```

```java
public interface DictDataApi extends DictDataCommonApi {
    void validateDictDataList(String dictType, Collection<String> values);
}
```
