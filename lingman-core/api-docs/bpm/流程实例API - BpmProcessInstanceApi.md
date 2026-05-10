# 流程实例 API - BpmProcessInstanceApi

## 目录

- [由来](#由来)
- [导入该 API](#导入该-api)
- [方法](#方法)
  - [1. `createProcessInstance`](#1-createprocessinstance)
- [参数 / 返回说明](#参数--返回说明)
  - [BpmProcessInstanceCreateReqDTO](#bpmprocessinstancecreatereqdto)
- [相关](#相关)
  - [流程实例状态事件](#流程实例状态事件)
- [接口一览](#接口一览)

## 由来

`BpmProcessInstanceApi` 定义在 **`com.lm.starter.module.bpm.api.task`** 包下，由 BPM 模块（`lingman-module-bpm`）提供，用于从业务模块创建流程实例（发起审批）。

业务工程引入 BPM Starter 后，直接注入该接口即可启动工作流，**无需**经过 Feign 与 `CommonResult` 包装。

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
   import com.lm.starter.module.bpm.api.task.BpmProcessInstanceApi;
   import com.lm.starter.module.bpm.api.task.dto.BpmProcessInstanceCreateReqDTO;
   import jakarta.annotation.Resource;
   import org.springframework.stereotype.Service;

   import java.util.Map;

   @Service
   public class YourBizServiceImpl implements YourBizService {

       @Resource
       private BpmProcessInstanceApi bpmProcessInstanceApi;

       /** 发起一个审批流程 */
       public String startApproval(Long userId, String businessKey, Map<String, Object> formData) {
           BpmProcessInstanceCreateReqDTO reqDTO = new BpmProcessInstanceCreateReqDTO();
           reqDTO.setProcessDefinitionKey("leave-apply");
           reqDTO.setBusinessKey(businessKey);
           reqDTO.setVariables(formData);
           return bpmProcessInstanceApi.createProcessInstance(userId, reqDTO);
       }
   }
   ```

## 方法

### 1. `createProcessInstance`

**创建流程实例（提供给内部）**。根据流程定义标识和业务数据发起一个新的流程实例。

```java
String createProcessInstance(Long userId, @Valid BpmProcessInstanceCreateReqDTO reqDTO);
```

- **userId**：发起用户编号（`Long`）
- **reqDTO**：创建信息
- **返回**：流程实例编号（`String`）

## 参数 / 返回说明

### BpmProcessInstanceCreateReqDTO

- **processDefinitionKey**：流程定义的标识（`String`，必填）
- **variables**：变量实例，即动态表单数据（`Map<String, Object>`）
- **businessKey**：业务的唯一标识，如请假申请的编号（`String`，必填）；通过它可以查询到对应的实例
- **startUserSelectAssignees**：发起人自选审批人 Map（`Map<String, List<Long>>`），key 为 taskKey 任务编码，value 为审批人 ID 数组。例如 `{ "taskKey1": [1, 2] }` 表示 taskKey1 这个任务提前设定了由 userId 为 1、2 的用户进行审批

## 相关

### 流程实例状态事件

BPM 模块还提供了流程实例状态变化的事件机制：

- **`BpmProcessInstanceStatusEvent`**：流程实例状态（结果）发生变化时发布的 Spring 事件，包含 `id`（实例编号）、`processDefinitionKey`（流程定义 Key）、`status`（结果状态）、`reason`（结束原因）、`businessKey`（业务标识）
- **`BpmProcessInstanceStatusEventListener`**：事件监听器抽象基类，业务模块继承后实现 `getProcessDefinitionKey()` 和 `onEvent()` 即可监听特定流程的完成/取消事件

```java
// 监听请假流程的完成事件
@Component
public class LeaveProcessListener extends BpmProcessInstanceStatusEventListener {

    @Override
    protected String getProcessDefinitionKey() {
        return "leave-apply";
    }

    @Override
    protected void onEvent(BpmProcessInstanceStatusEvent event) {
        // 根据 event.getStatus() 和 event.getBusinessKey() 处理业务逻辑
    }
}
```

## 接口一览

```java
public interface BpmProcessInstanceApi {
    String createProcessInstance(Long userId, @Valid BpmProcessInstanceCreateReqDTO reqDTO);
}
```
