# 各层代码模板

## Controller 模板

```java
package com.lm.app.controller.admin.{biz};

import com.lm.app.controller.admin.{biz}.vo.*;
import com.lm.app.service.{biz}.{Name}Service;
import com.lm.starter.framework.common.pojo.CommonResult;
import com.lm.starter.framework.common.pojo.PageResult;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;

import static com.lm.starter.framework.common.pojo.CommonResult.success;

@Tag(name = "管理后台 - {BizName}")
@RestController
@RequestMapping("/app/{biz}")
public class {Name}Controller {

    @Resource
    private {Name}Service {name}Service;

    @PostMapping("/create")
    @Operation(summary = "创建{BizName}")
    public CommonResult<Long> create(@Valid @RequestBody {Name}SaveReqVO createReqVO) {
        return success({name}Service.create(createReqVO));
    }

    @PutMapping("/update")
    @Operation(summary = "更新{BizName}")
    public CommonResult<Boolean> update(@Valid @RequestBody {Name}SaveReqVO updateReqVO) {
        {name}Service.update(updateReqVO);
        return success(true);
    }

    @DeleteMapping("/delete")
    @Operation(summary = "删除{BizName}")
    public CommonResult<Boolean> delete(@Valid @RequestBody {DeleteReqVO} deleteReqVO) {
        {name}Service.delete(deleteReqVO.getId());
        return success(true);
    }

    @GetMapping("/get")
    @Operation(summary = "获得{BizName}详情")
    public CommonResult<{Name}RespVO> get(@RequestParam("id") Long id) {
        return success({name}Service.get(id));
    }

    @GetMapping("/page")
    @Operation(summary = "获得{BizName}分页")
    public CommonResult<PageResult<{Name}RespVO>> page(@Valid {Name}PageReqVO pageReqVO) {
        return success({name}Service.page(pageReqVO));
    }
}
```

## Service 接口模板

```java
package com.lm.app.service.{biz};

import com.lm.starter.framework.common.pojo.PageResult;
import com.lm.app.controller.admin.{biz}.vo.*;

public interface {Name}Service {

    Long create({Name}SaveReqVO createReqVO);

    void update({Name}SaveReqVO updateReqVO);

    void delete(Long id);

    {Name}RespVO get(Long id);

    PageResult<{Name}RespVO> page({Name}PageReqVO pageReqVO);
}
```

## Service 实现模板

```java
package com.lm.app.service.{biz};

import com.lm.app.controller.admin.{biz}.vo.*;
import com.lm.app.convert.{biz}.{Name}Convert;
import com.lm.app.enums.ErrorCodeConstants;
import com.lm.app.models.entity.{Name}DO;
import com.lm.app.models.mapper.{Name}Mapper;
import com.lm.starter.framework.common.pojo.PageResult;
import com.lm.starter.framework.mybatis.core.query.LambdaQueryWrapperX;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Collections;
import java.util.List;

import static com.lm.starter.framework.common.exception.util.ServiceExceptionUtil.exception;

@Service
public class {Name}ServiceImpl implements {Name}Service {

    @Resource
    private {Name}Mapper {name}Mapper;
    @Resource
    private {Name}Convert {name}Convert;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create({Name}SaveReqVO createReqVO) {
        {Name}DO entity = {name}Convert.convert(createReqVO);
        {name}Mapper.insert(entity);
        return entity.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void update({Name}SaveReqVO updateReqVO) {
        validate{Name}Exists(updateReqVO.getId());
        {Name}DO updateObj = {name}Convert.convert(updateReqVO);
        updateObj.clean();
        {name}Mapper.updateById(updateObj);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void delete(Long id) {
        validate{Name}Exists(id);
        {name}Mapper.deleteById(id);
    }

    @Override
    public {Name}RespVO get(Long id) {
        {Name}DO entity = {name}Mapper.selectById(id);
        if (entity == null) {
            throw exception(ErrorCodeConstants.{NAME}_NOT_EXISTS);
        }
        return {name}Convert.convert(entity);
    }

    @Override
    public PageResult<{Name}RespVO> page({Name}PageReqVO pageReqVO) {
        LambdaQueryWrapperX<{Name}DO> queryWrapper = new LambdaQueryWrapperX<>();
        // queryWrapper.likeIfPresent({Name}DO::getName, pageReqVO.getName());
        queryWrapper.orderByDesc({Name}DO::getId);
        return {name}Mapper.selectPage(pageReqVO, queryWrapper)
            .convert({name}Convert::convert);
    }

    private void validate{Name}Exists(Long id) {
        if (id == null || {name}Mapper.selectById(id) == null) {
            throw exception(ErrorCodeConstants.{NAME}_NOT_EXISTS);
        }
    }
}
```

## Convert 模板

```java
package com.lm.app.convert.{biz};

import com.lm.app.controller.admin.{biz}.vo.*;
import com.lm.app.models.entity.{Name}DO;
import com.lm.starter.framework.common.pojo.PageResult;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

import java.util.Collections;
import java.util.List;

@Mapper
public interface {Name}Convert {

    {Name}Convert INSTANCE = Mappers.getMapper({Name}Convert.class);

    {Name}DO convert({Name}SaveReqVO bean);

    {Name}RespVO convert({Name}DO bean);

    default PageResult<{Name}RespVO> convertPage(PageResult<{Name}DO> page) {
        if (page == null) {
            return null;
        }
        List<{Name}RespVO> list = page.getList() == null
            ? Collections.emptyList()
            : page.getList().stream().map(this::convert).toList();
        return new PageResult<>(list, page.getTotal());
    }
}
```

## VO 模板

### SaveReqVO

```java
package com.lm.app.controller.admin.{biz}.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
@Schema(description = "管理后台 - {BizName}保存 Request VO")
public class {Name}SaveReqVO {

    @Schema(description = "编号（更新时必填）")
    private Long id;

    @Schema(description = "名称", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "名称不能为空")
    private String name;

    // 其他字段...
}
```

### DeleteReqVO（兜底模板）

> 单 ID 删除优先复用项目已有的公共删除/ID 请求 VO（如 `IdReqVO`、`DeleteReqVO`、`BaseIdReqVO` 等）。只有项目没有可复用公共 VO 时，才生成该业务专属 `{Name}DeleteReqVO`。

```java
package com.lm.app.controller.admin.{biz}.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

@Data
@Schema(description = "管理后台 - {BizName}删除 Request VO")
public class {Name}DeleteReqVO {

    @Schema(description = "编号", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "编号不能为空")
    private Long id;
}
```

### RespVO

```java
package com.lm.app.controller.admin.{biz}.vo;

import com.fasterxml.jackson.annotation.JsonFormat;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Schema(description = "管理后台 - {BizName} Response VO")
public class {Name}RespVO {

    @Schema(description = "编号")
    private Long id;

    @Schema(description = "名称")
    private String name;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @Schema(description = "创建时间")
    private LocalDateTime createTime;
}
```

### PageReqVO

```java
package com.lm.app.controller.admin.{biz}.vo;

import com.lm.starter.framework.common.pojo.PageParam;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = true)
@Schema(description = "管理后台 - {BizName}分页 Request VO")
public class {Name}PageReqVO extends PageParam {

    @Schema(description = "名称")
    private String name;

    // 其他查询条件...
}
```

## DO 模板

```java
package com.lm.app.models.entity;

import com.baomidou.mybatisplus.annotation.*;
import com.lm.starter.framework.mybatis.core.dataobject.BaseDO;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;

@TableName("t_{name}")
@KeySequence("{name}_seq")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
public class {Name}DO extends BaseDO {

    @TableId(value = "id")
    private Long id;

    @TableField("name")
    private String name;

    // 其他字段...
}
```

## Mapper 模板

```java
package com.lm.app.models.mapper;

import com.lm.app.models.entity.{Name}DO;
import com.lm.starter.framework.mybatis.core.mapper.BaseMapperX;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface {Name}Mapper extends BaseMapperX<{Name}DO> {
}
```

## 变量替换说明

| 变量 | 含义 | 示例 |
|------|------|------|
| `{Name}` | 业务实体名（驼峰首字母大写） | `Announcement`、`DetectionTask` |
| `{name}` | 业务实体名（驼峰首字母小写） | `announcement`、`detectionTask` |
| `{NAME}` | 业务实体名（全大写） | `ANNOUNCEMENT`、`DETECTION_TASK` |
| `{biz}` | 业务标识（小写 kebab-case） | `announcement`、`detection-task` |
| `{BizName}` | 业务中文名称 | `公告`、`检测任务` |
