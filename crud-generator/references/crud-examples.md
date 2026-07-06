# CRUD 代码示例

## 完整业务模块示例：检测任务

以下是一个完整的检测任务模块 CRUD 代码示例

### DO（由 CLI 工具从数据库生成）

```java
package com.lm.app.models.entity;

import com.baomidou.mybatisplus.annotation.*;
import com.lm.starter.framework.mybatis.core.dataobject.BaseDO;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;

@TableName("t_task")
@KeySequence("task_seq")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
public class TaskDO extends BaseDO {

    /** 主键ID */
@TableId(value = "id")
private Long id;

/** 任务名称 */
@TableField("task_name")
private String taskName;

/** 关联模型ID */
@TableField("model_id")
private Long modelId;

/** 告警模板（告警消息内容模板） */
@TableField("alarm_template")
private String alarmTemplate;

/** 告警级别（如：1-提示，2-警告，3-严重） */
@TableField("alarm_level")
private String alarmLevel;

/** 状态（0-禁用，1-启用） */
@TableField("status")
private Short status;

/** 备注说明 */
@TableField("remark")
private String remark;
}
```

### Mapper（由 CLI 工具从数据库生成）

```java
package com.lm.app.models.mapper;

import com.lm.app.models.entity.TaskDO;
import com.lm.starter.framework.mybatis.core.mapper.BaseMapperX;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface DetectionTaskMapper extends BaseMapperX<TaskDO> {
}
```

### Convert

```java
package com.lm.app.convert.detectiontask;

import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskRespVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.app.models.entity.TaskDO;
import com.lm.starter.framework.common.pojo.PageResult;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

import java.util.Collections;
import java.util.List;

@Mapper
public interface DetectionTaskConvert {

    DetectionTaskConvert INSTANCE = Mappers.getMapper(DetectionTaskConvert.class);

    TaskDO convert(DetectionTaskSaveReqVO bean);

    DetectionTaskRespVO convert(TaskDO bean);

    default PageResult<DetectionTaskRespVO> convertPage(PageResult<TaskDO> page) {
        if (page == null) {
            return null;
        }
        List<DetectionTaskRespVO> list = page.getList() == null
            ? Collections.emptyList()
            : page.getList().stream().map(this::convert).toList();
        return new PageResult<>(list, page.getTotal());
    }
}
```

### VO

#### DetectionTaskSaveReqVO.java

```java
package com.lm.app.controller.admin.detectiontask.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

@Data
@Schema(description = "管理后台 - 检测任务保存 Request VO")
public class DetectionTaskSaveReqVO {

    @Schema(description = "编号（更新时必填）")
    private Long id;

    @Schema(description = "任务名称", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "任务名称不能为空")
    private String taskName;

    @Schema(description = "模型ID", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "模型ID不能为空")
    private Long modelId;

    @Schema(description = "告警模板", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "告警模板不能为空")
    private String alarmTemplate;

    @Schema(description = "告警级别", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "告警级别不能为空")
    private String alarmLevel;

    @Schema(description = "状态（0-禁用，1-启用）", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "状态不能为空")
    private Short status;

    @Schema(description = "备注")
    private String remark;
}
```

#### DetectionTaskDeleteReqVO.java

```java
package com.lm.app.controller.admin.detectiontask.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

@Data
@Schema(description = "管理后台 - 检测任务删除 Request VO")
public class DetectionTaskDeleteReqVO {

    @Schema(description = "编号", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "编号不能为空")
    private Long id;
}
```

#### DetectionTaskRespVO.java

```java
package com.lm.app.controller.admin.detectiontask.vo;

import com.fasterxml.jackson.annotation.JsonFormat;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Schema(description = "管理后台 - 检测任务 Response VO")
public class DetectionTaskRespVO {

    @Schema(description = "编号")
    private Long id;

    @Schema(description = "任务名称")
    private String taskName;

    @Schema(description = "模型编号")
    private Long modelId;

    @Schema(description = "告警模板")
    private String alarmTemplate;

    @Schema(description = "告警级别")
    private String alarmLevel;

    @Schema(description = "状态")
    private Short status;

    @Schema(description = "备注")
    private String remark;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @Schema(description = "创建时间")
    private LocalDateTime createTime;
}
```

#### DetectionTaskPageReqVO.java

```java
package com.lm.app.controller.admin.detectiontask.vo;

import com.lm.starter.framework.common.pojo.PageParam;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = true)
@Schema(description = "管理后台 - 检测任务分页 Request VO")
public class DetectionTaskPageReqVO extends PageParam {

    @Schema(description = "任务名称")
    private String taskName;

    @Schema(description = "状态")
    private Short status;
}
```

### Service

#### DetectionTaskService.java

```java
package com.lm.app.service.detectiontask;

import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskPageReqVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskRespVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.starter.framework.common.pojo.PageResult;

public interface DetectionTaskService {

   /**
 * 创建检测任务
 *
 * @param createReqVO 创建请求参数
 * @return 新创建的检测任务ID
 */
Long create(DetectionTaskSaveReqVO createReqVO);

/**
 * 更新检测任务
 *
 * @param updateReqVO 更新请求参数（需包含任务ID）
 */
void update(DetectionTaskSaveReqVO updateReqVO);

/**
 * 删除检测任务
 *
 * @param id 检测任务ID
 */
void delete(Long id);

/**
 * 获取检测任务详情
 *
 * @param id 检测任务ID
 * @return 检测任务响应对象，不存在时返回 null
 */
DetectionTaskRespVO get(Long id);

/**
 * 分页查询检测任务列表
 *
 * @param pageReqVO 分页查询请求参数
 * @return 检测任务分页结果
 */
PageResult<DetectionTaskRespVO> page(DetectionTaskPageReqVO pageReqVO);
}
```

#### DetectionTaskServiceImpl.java

```java
package com.lm.app.service.detectiontask;

import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskPageReqVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskRespVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.app.convert.detectiontask.DetectionTaskConvert;
import com.lm.app.enums.ErrorCodeConstants;
import com.lm.app.models.entity.TaskDO;
import com.lm.app.models.mapper.DetectionTaskMapper;
import com.lm.starter.framework.common.pojo.PageResult;
import com.lm.starter.framework.mybatis.core.query.LambdaQueryWrapperX;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import static com.lm.starter.framework.common.exception.util.ServiceExceptionUtil.exception;

@Service
public class DetectionTaskServiceImpl implements DetectionTaskService {

    @Resource
    private DetectionTaskMapper detectionTaskMapper;
    @Resource
    private DetectionTaskConvert detectionTaskConvert;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create(DetectionTaskSaveReqVO createReqVO) {
        TaskDO detectionTask = detectionTaskConvert.convert(createReqVO);
        detectionTaskMapper.insert(detectionTask);
        return detectionTask.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void update(DetectionTaskSaveReqVO updateReqVO) {
        validateDetectionTaskExists(updateReqVO.getId());
        TaskDO updateObj = detectionTaskConvert.convert(updateReqVO);
        updateObj.clean();
        detectionTaskMapper.updateById(updateObj);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void delete(Long id) {
        validateDetectionTaskExists(id);
        detectionTaskMapper.deleteById(id);
    }

    @Override
    public DetectionTaskRespVO get(Long id) {
        TaskDO task = detectionTaskMapper.selectById(id);
        if (task == null) {
            throw exception(ErrorCodeConstants.DETECTION_TASK_NOT_EXISTS);
        }
        return detectionTaskConvert.convert(task);
    }
    /**
 * 分页查询检测任务
 * <p>支持按任务名称模糊搜索、按状态精确过滤，默认按ID倒序排列</p>
 */
    @Override
    public PageResult<DetectionTaskRespVO> page(DetectionTaskPageReqVO pageReqVO) {
        LambdaQueryWrapperX<TaskDO> queryWrapper = new LambdaQueryWrapperX<>();
        queryWrapper.likeIfPresent(TaskDO::getTaskName, pageReqVO.getTaskName());
        queryWrapper.eqIfPresent(TaskDO::getStatus, pageReqVO.getStatus());
        queryWrapper.orderByDesc(TaskDO::getId);
        return detectionTaskMapper.selectPage(pageReqVO, queryWrapper)
            .convert(detectionTaskConvert::convert);
    }
    /**
 * 校验检测任务是否存在，不存在则抛出 DETECTION_TASK_NOT_EXISTS 异常
 */
    private void validateDetectionTaskExists(Long id) {
        if (id == null || detectionTaskMapper.selectById(id) == null) {
            throw exception(ErrorCodeConstants.DETECTION_TASK_NOT_EXISTS);
        }
    }
}
```

### Controller

#### DetectionTaskController.java

```java
package com.lm.app.controller.admin.detectiontask;

import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskDeleteReqVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskPageReqVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskRespVO;
import com.lm.app.controller.admin.detectiontask.vo.DetectionTaskSaveReqVO;
import com.lm.app.service.detectiontask.DetectionTaskService;
import com.lm.starter.framework.common.pojo.CommonResult;
import com.lm.starter.framework.common.pojo.PageResult;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;

import static com.lm.starter.framework.common.pojo.CommonResult.success;

@Tag(name = "管理后台 - 检测任务")
@RestController
@RequestMapping("/app/detection-task")
public class DetectionTaskController {

    @Resource
    private DetectionTaskService detectionTaskService;

    @PostMapping("/create")
    @Operation(summary = "创建检测任务")
    public CommonResult<Long> create(@Valid @RequestBody DetectionTaskSaveReqVO createReqVO) {
        return success(detectionTaskService.create(createReqVO));
    }

    @PutMapping("/update")
    @Operation(summary = "更新检测任务")
    public CommonResult<Boolean> update(@Valid @RequestBody DetectionTaskSaveReqVO updateReqVO) {
        detectionTaskService.update(updateReqVO);
        return success(true);
    }

    @DeleteMapping("/delete")
    @Operation(summary = "删除检测任务")
    public CommonResult<Boolean> delete(@Valid @RequestBody DetectionTaskDeleteReqVO deleteReqVO) {
        detectionTaskService.delete(deleteReqVO.getId());
        return success(true);
    }

    @GetMapping("/get")
    @Operation(summary = "获得检测任务详情")
    public CommonResult<DetectionTaskRespVO> get(@RequestParam("id") Long id) {
        return success(detectionTaskService.get(id));
    }

    @GetMapping("/page")
    @Operation(summary = "获得检测任务分页")
    public CommonResult<PageResult<DetectionTaskRespVO>> page(@Valid DetectionTaskPageReqVO pageReqVO) {
        return success(detectionTaskService.page(pageReqVO));
    }
}
```

### 错误码

```java
package com.lm.app.enums;

import com.lm.starter.framework.common.exception.ErrorCode;

public interface ErrorCodeConstants {

    ErrorCode DETECTION_TASK_NOT_EXISTS = new ErrorCode(1_100_000_001, "检测任务不存在");
}
```
