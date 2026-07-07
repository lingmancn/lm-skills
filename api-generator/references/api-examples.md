# 接口设计示例

## 简单 CRUD 模块：车辆管理

### 模块概述
- **业务**：车辆信息管理（增删改查）
- **模块标识**：`vehicle`
- **包名**：`com.lm.app.controller.admin.vehicle`

### 接口清单

#### 1. 创建车辆
```
POST /admin/vehicle/create
```
- **权限**：`stuff:vehicle:create`
- **请求体**：VehicleCreateReqVO
  - plateNo (string, required): 车牌号
  - brandModel (string): 品牌型号
  - vehicleType (string): 车辆类型
  - maintenancePeriod (int): 保养周期（月）
  - status (int): 状态：0闲置 1使用中 2维修中
- **响应**：`CommonResult<Long>` 车辆ID

#### 2. 更新车辆
```
POST /admin/vehicle/update
```
- **权限**：`stuff:vehicle:update`
- **请求体**：VehicleUpdateReqVO
  - id (long, required): 车辆ID
  - plateNo (string, required): 车牌号
  - brandModel (string): 品牌型号
  - vehicleType (string): 车辆类型
  - maintenancePeriod (int): 保养周期
  - status (int): 状态
- **响应**：`CommonResult<Boolean>`

#### 3. 删除车辆
```
GET /admin/vehicle/delete
```
- **权限**：`stuff:vehicle:delete`
- **请求参数**：id (long, query, required)
- **响应**：`CommonResult<Boolean>`
- **业务规则**：仅闲置状态车辆可删除

#### 4. 查询详情
```
GET /admin/vehicle/get
```
- **权限**：`stuff:vehicle:query`
- **请求参数**：id (long, query, required)
- **响应**：`CommonResult<VehicleRespVO>`

#### 5. 分页查询
```
POST /admin/vehicle/page
```
- **权限**：`stuff:vehicle:query`
- **请求体**：VehiclePageReqVO
  - pageNo (int): 页码，默认1
  - pageSize (int): 每页条数，默认10
  - plateNo (string): 车牌号（模糊查询）
  - status (int): 状态
  - vehicleType (string): 车辆类型
- **响应**：`CommonResult<PageResult<VehicleRespVO>>`

#### 6. 状态统计
```
GET /admin/vehicle/status-count
```
- **权限**：`stuff:vehicle:query`
- **响应**：`CommonResult<Map<String, Long>>`
  - 返回各状态车辆数量统计

---

## 复杂业务模块：外出报备

### 模块概述
- **业务**：人员外出申请、审批、延期、归来
- **模块标识**：`outbound`
- **涉及 BPM 工作流**：外出审批流程

### 接口清单

#### 1. 提交外出申请
```
POST /app/outbound/submit
```
- **权限**：登录即可（业务校验）
- **请求体**：OutboundRecordSaveReqVO
  - outType (int, required): 外出类型 1公差 2休假 3事假 4其他
  - reason (string, required): 外出事由
  - location (string): 外出地点
  - leaveTime (datetime, required): 离开时间
  - returnTime (datetime, required): 预计返回时间
  - mobile (string): 联系电话
- **响应**：`CommonResult<String>` 记录ID（雪花ID）
- **业务规则**：
  - 不可与未结束的外出时间段重叠
  - 自动启动 Flowable 审批流程

#### 2. 我的外出列表
```
POST /app/outbound/my
```
- **权限**：登录即可
- **请求体**：OutboundRecordReqVO
  - pageNo / pageSize
  - status (int): 状态筛选
  - outType (int): 类型筛选
- **响应**：`CommonResult<PageResult<OutboundRecordRespVO>>`
- **数据权限**：仅查看自己的记录

#### 3. 外出详情
```
GET /app/outbound/detail
```
- **权限**：登录即可
- **请求参数**：id (string, query)
- **响应**：`CommonResult<OutboundRecordDetailRespVO>`
- **包含信息**：外出记录 + 审批历史 + 用户信息

#### 4. 申请延期
```
POST /app/outbound/extend
```
- **权限**：登录即可
- **请求体**：OutboundExtendReqVO
  - id (string, required): 记录ID
  - extendTime (datetime, required): 延期至
  - extendReason (string): 延期事由
- **响应**：`CommonResult<Boolean>`
- **业务规则**：仅"外出中"或"延期中"状态可申请

#### 5. 归来确认
```
GET /app/outbound/return
```
- **权限**：登录即可
- **请求参数**：id (string, query)
- **响应**：`CommonResult<Boolean>`
- **业务规则**：仅"外出中"状态可确认归来

#### 6. 撤销申请
```
GET /app/outbound/delete
```
- **权限**：登录即可
- **请求参数**：id (string, query)
- **响应**：`CommonResult<Boolean>`
- **业务规则**：仅"已驳回"状态可删除

---

## 接口设计规范速查

| 场景 | HTTP方法 | URL | 请求方式 |
|------|---------|-----|---------|
| 创建 | POST | /create | Body |
| 更新 | POST | /update | Body |
| 删除 | GET | /delete | Query |
| 详情 | GET | /get | Query |
| 分页 | POST | /page | Body |
| 列表 | POST | /list | Body |
| 统计 | GET | /count | Query/无 |
