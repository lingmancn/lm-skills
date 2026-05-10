# 流程任务 API - BpmProcessTaskApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `triggerTask`](#1-triggertask)
- [接口一览](#接口一览)

## 由来

`BpmProcessTaskApi` 定义在 **`com.lm.starter.module.bpm.api.task`** 包下，由 BPM 模块（`lingman-module-bpm`）提供，用于在业务模块中触发流程任务的执行。

当业务流程中的某个节点需要在外部条件满足后推进（例如回调、定时触发等场景），可以注入该接口按任务定义 Key 触发任务执行。

## 导入该 API

1. **在业务工程中引入 Starter 依赖**：

   **Maven**

   ```xml
   <dependency>
   	<groupId>com.lm.bpm</groupId>
   	<artifactId>lingman-module-bpm</artifactId>
   </dependency>
   ```

2. **在业务代码中注入并调用**：

   ```java
   import com.lm.starter.module.bpm.api.task.BpmProcessTaskApi;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private BpmProcessTaskApi bpmProcessTaskApi;

       /** 触发指定流程实例的指定任务 */
       public void triggerTask(String processInstanceId, String taskDefineKey) {
           bpmProcessTaskApi.triggerTask(processInstanceId, taskDefineKey);
       }
   }
   ```

## 方法

### 1. `triggerTask`

**触发流程任务的执行**。根据流程实例编号和任务定义 Key 推进流程到对应任务节点。

```java
void triggerTask(@NotEmpty(message = "流程实例的编号不能为空") String processInstanceId,
                 @NotEmpty(message = "任务 Key 不能为空") String taskDefineKey);
```

- **processInstanceId**：流程实例编号（`String`，必填）
- **taskDefineKey**：任务定义 Key（`String`，必填），对应 BPMN 中 userTask 的 `key` 属性

## 接口一览

```java
public interface BpmProcessTaskApi {
    void triggerTask(@NotEmpty(message = "流程实例的编号不能为空") String processInstanceId,
                     @NotEmpty(message = "任务 Key 不能为空") String taskDefineKey);
}
```
