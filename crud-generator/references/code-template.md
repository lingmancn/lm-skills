# 各层代码模板

## Controller 模板

```java
package com.lm.app.controller.admin.{biz};

import com.lm.app.controller.admin.{biz}.vo.*;
import com.lm.app.controller.admin.common_vo.AdminDeleteReqVO;
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
    public CommonResult<Boolean> delete(@Valid @RequestBody AdminDeleteReqVO deleteReqVO) {
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

    /** 创建{BizName} */
    Long create({Name}SaveReqVO createReqVO);

    /** 更新{BizName} */
    void update({Name}SaveReqVO updateReqVO);

    /** 删除{BizName} */
    void delete(Long id);

    /** 获取{BizName}详情 */
    {Name}RespVO get(Long id);

    /** 分页查询{BizName} */
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
        // 1. 业务规则校验（按需补充，如唯一性、关联存在性校验）
        // 2. SaveReqVO → DO 转换
        {Name}DO entity = {name}Convert.convert(createReqVO);
        // 3. 持久化并回填主键
        {name}Mapper.insert(entity);
        return entity.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void update({Name}SaveReqVO updateReqVO) {
        // 1. 校验存在
        validate{Name}Exists(updateReqVO.getId());
        // 2. SaveReqVO → DO 转换，清理更新保护字段
        {Name}DO updateObj = {name}Convert.convert(updateReqVO);
        updateObj.clean();
        // 3. 按 ID 更新
        {name}Mapper.updateById(updateObj);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void delete(Long id) {
        // 1. 校验存在
        validate{Name}Exists(id);
        // 2. 按 ID 删除
        {name}Mapper.deleteById(id);
    }

    @Override
    public {Name}RespVO get(Long id) {
        // 1. 按 ID 查询，不存在则抛异常
        {Name}DO entity = {name}Mapper.selectById(id);
        if (entity == null) {
            throw exception(ErrorCodeConstants.{NAME}_NOT_EXISTS);
        }
        // 2. DO → RespVO 转换
        return {name}Convert.convert(entity);
    }

    @Override
    public PageResult<{Name}RespVO> page({Name}PageReqVO pageReqVO) {
        // 1. 构建查询条件（按需补充业务查询字段）
        LambdaQueryWrapperX<{Name}DO> queryWrapper = new LambdaQueryWrapperX<>();
        // queryWrapper.likeIfPresent({Name}DO::getName, pageReqVO.getName());
        queryWrapper.orderByDesc({Name}DO::getId);
        // 2. 分页查询并转换为 RespVO
        return {name}Mapper.selectPage(pageReqVO, queryWrapper)
            .convert({name}Convert::convert);
    }

    // 校验 {BizName} 是否存在，不存在则抛出 {NAME}_NOT_EXISTS 异常
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

### AdminDeleteReqVO（公共单 ID 请求类）

> 全项目统一的公共单 ID 请求类，**删除、单 ID 操作等仅需传递一个主键 id 的非 GET 接口统一使用本类**，禁止各业务模块自行定义 `{Name}DeleteReqVO` 等同义类。
>
> 路径固定为 `controller/admin/common_vo/AdminDeleteReqVO.java`。生成业务代码前若该类不存在，必须先新增；存在则直接引用。

```java
package com.lm.app.controller.admin.common_vo;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

/**
 * 管理后台通用单 ID 请求 VO
 * <p>
 * 适用于删除等仅需传递一个主键 id 的非 GET 接口。
 */
@Data
@Schema(description = "管理后台 - 通用单 ID 请求 VO")
public class AdminDeleteReqVO {

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

## 变量替换说明

| 变量 | 含义 | 示例 |
|------|------|------|
| `{Name}` | 业务实体名（驼峰首字母大写） | `Announcement`、`DetectionTask` |
| `{name}` | 业务实体名（驼峰首字母小写） | `announcement`、`detectionTask` |
| `{NAME}` | 业务实体名（全大写） | `ANNOUNCEMENT`、`DETECTION_TASK` |
| `{biz}` | 业务标识（小写 kebab-case） | `announcement`、`detection-task` |
| `{BizName}` | 业务中文名称 | `公告`、`检测任务` |
