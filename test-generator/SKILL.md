---
name: test-generator
description: Lingman-Starter 框架测试代码生成助手。当用户需要：(1) 生成 Controller 集成测试 (2) 生成 Service 单元测试 (3) 生成 Mock 测试数据 (4) 为已有接口补充测试用例 (5) 生成测试所需的 SQL 初始化数据 时触发此技能。不要在以下场景触发：生成业务代码（由 crud-generator 处理）、生成 SQL（由 sql-generator 处理）、错误排查（由 error-analyzer 处理）。
---

# 测试代码生成指南

## 测试体系概述

lingman-starter 项目使用以下测试框架：

| 框架 | 用途 |
|------|------|
| JUnit 5 | 测试运行 |
| Spring Boot Test | 集成测试 |
| MockMvc | MVC 接口测试 |
| Mockito | Service 层 Mock |

## 测试基类

### 集成测试基类

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
public abstract class AbstractSpringBootMvcIntegrationTest {

    @Resource
    protected MockMvc mockMvc;

    protected RequestPostProcessor integrationAuth() {
        // 返回模拟登录凭证
    }
}
```

**特性**：
- `@Transactional`：每个测试方法后自动回滚
- `@ActiveProfiles("test")`：使用 test 环境配置
- `integrationAuth()`：自动附带模拟登录令牌

## 生成范围

1. **Controller 集成测试** — MockMvc 调用 + 响应断言
2. **Service 单元测试** — Mockito 模拟 Mapper
3. **Mock 数据** — 测试用数据构造
4. **测试 SQL** — 初始化数据脚本

## Controller 测试模板

```java
class {Name}ControllerTest extends AbstractSpringBootMvcIntegrationTest {

    @Resource
    private MockMvc mockMvc;

    @Resource
    private {Name}Mapper {name}Mapper;

    @Test
    void page_returnsOk() throws Exception {
        mockMvc.perform(
                post("/admin/{biz}/page")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content("{}")
                    .with(integrationAuth()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data").exists());
    }

    @Test
    void get_returnsOk_whenRecordExists() throws Exception {
        // 查询数据库获取一条真实记录
        {Name}DO one = {name}Mapper.selectOne(
            Wrappers.lambdaQuery({Name}DO.class)
                .orderByDesc({Name}DO::getId)
                .last("LIMIT 1"));

        Assumptions.assumeTrue(one != null, "无数据时跳过");

        mockMvc.perform(
                get("/admin/{biz}/get")
                    .param("id", String.valueOf(one.getId()))
                    .with(integrationAuth()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data").exists());
    }

    @Test
    void create_returnsOk_writesDb() throws Exception {
        String body = """
            {
              "field1": "value1",
              "field2": "value2"
            }
            """;

        String resp = mockMvc.perform(
                post("/admin/{biz}/create")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(body)
                    .with(integrationAuth()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andReturn()
            .getResponse()
            .getContentAsString();

        // 验证数据库写入
        JsonNode root = IntegrationTestMvcSupport.parseResponseRoot(resp);
        String id = root.path("data").asText();
        assertNotNull(id);

        IntegrationTestMvcSupport.withTenant(tenantId, () -> {
            {Name}DO row = {name}Mapper.selectById(Long.valueOf(id));
            assertNotNull(row);
            assertEquals("value1", row.getField1());
        });
    }
}
```

## Service 单元测试模板

```java
@ExtendWith(MockitoExtension.class)
class {Name}ServiceTest {

    @Mock
    private {Name}Mapper {name}Mapper;

    @InjectMocks
    private {Name}ServiceImpl {name}Service;

    @Test
    void create_success() {
        // given
        {Name}CreateReqVO reqVO = new {Name}CreateReqVO();
        reqVO.setName("测试");

        // when
        Long id = {name}Service.create(reqVO);

        // then
        verify({name}Mapper).insert(argThat(entity ->
            "测试".equals(entity.getName())
        ));
    }

    @Test
    void get_throws_whenNotExists() {
        // given
        when({name}Mapper.selectById(1L)).thenReturn(null);

        // when/then
        assertThrows(ServiceException.class, () -> {
            {name}Service.get(1L);
        });
    }
}
```

## 测试工具类

| 工具方法 | 用途 |
|---------|------|
| `integrationAuth()` | 模拟登录请求 |
| `IntegrationTestMvcSupport.parseResponseRoot(resp)` | 解析响应 JSON |
| `IntegrationTestMvcSupport.readCode(resp)` | 读取响应 code |
| `IntegrationTestMvcSupport.withTenant(tenantId, action)` | 在租户上下文中执行 |

## 参考文档

| 场景 | 参考文档 |
|------|----------|
| 测试代码示例 | [test-examples.md](references/test-examples.md) |
| 框架规范 | [framework.md](../lingman-core/framework.md) |
