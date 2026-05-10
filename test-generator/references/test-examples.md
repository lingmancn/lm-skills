# 测试代码示例

## Controller 集成测试

### 检测任务 Controller 测试

```java
package com.lm.app.controller.admin.detectiontask;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.lm.app.AppApplication;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.app.models.entity.TaskDO;
import com.lm.app.models.mapper.DetectionTaskMapper;
import jakarta.annotation.Resource;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.transaction.annotation.Transactional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest(classes = AppApplication.class)
@Transactional
class DetectionTaskControllerTest {

    @Resource
    private DetectionTaskController detectionTaskController;

    @Resource
    private DetectionTaskMapper detectionTaskMapper;

    @Resource
    private ObjectMapper objectMapper;

    private MockMvc mockMvc;

    @org.junit.jupiter.api.BeforeEach
    void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(detectionTaskController).build();
    }

    @Test
    void create_shouldWriteToDatabase() throws Exception {
        DetectionTaskSaveReqVO reqVO = new DetectionTaskSaveReqVO();
        reqVO.setTaskName("火焰检测");
        reqVO.setModelId(1L);

        mockMvc.perform(post("/app/detection-task/create")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(reqVO)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data").isNumber());
    }

    @Test
    void get_shouldReturnResult_whenExists() throws Exception {
        TaskDO task = TaskDO.builder()
            .taskName("测试任务")
            .modelId(1L)
            .status((short) 0)
            .build();
        detectionTaskMapper.insert(task);

        mockMvc.perform(get("/app/detection-task/get")
                .param("id", String.valueOf(task.getId())))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data.taskName").value("测试任务"));
    }

    @Test
    void update_shouldModifyDatabase() throws Exception {
        TaskDO task = TaskDO.builder()
            .taskName("原名称")
            .modelId(1L)
            .status((short) 0)
            .build();
        detectionTaskMapper.insert(task);

        DetectionTaskSaveReqVO reqVO = new DetectionTaskSaveReqVO();
        reqVO.setId(task.getId());
        reqVO.setTaskName("新名称");
        reqVO.setModelId(1L);

        mockMvc.perform(put("/app/detection-task/update")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(reqVO)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0));

        TaskDO updated = detectionTaskMapper.selectById(task.getId());
        assertThat(updated.getTaskName()).isEqualTo("新名称");
    }

    @Test
    void delete_shouldRemoveFromDatabase() throws Exception {
        TaskDO task = TaskDO.builder()
            .taskName("待删除")
            .modelId(1L)
            .status((short) 0)
            .build();
        detectionTaskMapper.insert(task);

        mockMvc.perform(delete("/app/detection-task/delete")
                .param("id", String.valueOf(task.getId())))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0));

        assertThat(detectionTaskMapper.selectById(task.getId())).isNull();
    }

    @Test
    void page_shouldReturnPageResult() throws Exception {
        mockMvc.perform(get("/app/detection-task/page"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data.list").isArray());
    }
}
```

## Service 单元测试

```java
package com.lm.app.service.detectiontask;

import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.app.convert.detectiontask.DetectionTaskConvert;
import com.lm.app.enums.ErrorCodeConstants;
import com.lm.app.models.entity.TaskDO;
import com.lm.app.models.mapper.DetectionTaskMapper;
import com.lm.starter.framework.common.exception.ServiceException;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.argThat;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class DetectionTaskServiceImplTest {

    @Mock
    private DetectionTaskMapper detectionTaskMapper;

    @Mock
    private DetectionTaskConvert detectionTaskConvert;

    @InjectMocks
    private DetectionTaskServiceImpl detectionTaskService;

    @Test
    void create_shouldInsertAndReturnId() {
        // given
        DetectionTaskSaveReqVO reqVO = new DetectionTaskSaveReqVO();
        reqVO.setTaskName("火焰检测");
        reqVO.setModelId(1L);

        TaskDO entity = TaskDO.builder().taskName("火焰检测").modelId(1L).build();
        when(detectionTaskConvert.convert(reqVO)).thenReturn(entity);

        // when
        Long id = detectionTaskService.create(reqVO);

        // then
        verify(detectionTaskMapper).insert(argThat(t ->
            "火焰检测".equals(t.getTaskName()) && Long.valueOf(1L).equals(t.getModelId())
        ));
    }

    @Test
    void get_shouldThrow_whenNotExists() {
        // given
        when(detectionTaskMapper.selectById(1L)).thenReturn(null);

        // when/then
        assertThatThrownBy(() -> detectionTaskService.get(1L))
            .isInstanceOf(ServiceException.class)
            .hasMessageContaining(ErrorCodeConstants.DETECTION_TASK_NOT_EXISTS.getMsg());
    }

    @Test
    void get_shouldReturnVO_whenExists() {
        // given
        TaskDO entity = TaskDO.builder()
            .id(1L)
            .taskName("火焰检测")
            .build();
        when(detectionTaskMapper.selectById(1L)).thenReturn(entity);

        // when
        var result = detectionTaskService.get(1L);

        // then
        assertThat(result).isNotNull();
    }
}
```

## 纯单元测试（无 Spring 上下文）

```java
package com.lm.app.pipeline.result;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.lm.app.models.entity.AlarmRecordDO;
import com.lm.app.mq.message.detectiontask.DetectionTaskMessage;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

class AlarmRecordBuilderTest {

    private AlarmRecordBuilder builder;

    @BeforeEach
    void setUp() {
        builder = new AlarmRecordBuilder(new ObjectMapper());
    }

    @Test
    void build_shouldUseTaskNameAsAlarmName() {
        DetectionTaskMessage msg = DetectionTaskMessage.builder()
            .taskId(1L).deviceId(2L).modelId(3L)
            .taskName("火焰检测任务")
            .build();

        AlarmRecordDO alarm = builder.build(msg, null, "img");

        assertThat(alarm.getAlarmName()).isEqualTo("火焰检测任务");
    }

    @Test
    void build_shouldReturnNoAbnormal_whenInputIsNull() {
        DetectionTaskMessage msg = DetectionTaskMessage.builder()
            .taskId(1L).build();

        AlarmRecordDO alarm = builder.build(msg, null, null);

        assertThat(alarm.getHasAbnormal()).isFalse();
    }
}
```

## 测试规范速查

| 场景 | 测试类型 | 注解/方式 |
|------|---------|----------|
| Controller | 集成测试 | `@SpringBootTest` + `MockMvc` |
| Service | 单元测试 | `@ExtendWith(MockitoExtension.class)` |
| 工具类/Builder | 纯单元测试 | 无 Spring 注解 |
| 断言 | AssertJ | `assertThat(...).isEqualTo(...)` |
| 异常断言 | AssertJ | `assertThatThrownBy(() -> ...).isInstanceOf(...)` |

## 测试基类

根据测试类型选择：

- **Controller 测试**：`@SpringBootTest(classes = AppApplication.class)` + `@Transactional`
- **Service 测试**：`@ExtendWith(MockitoExtension.class)` + `@Mock` + `@InjectMocks`
- **纯单元测试**：无注解，直接 new 被测对象
