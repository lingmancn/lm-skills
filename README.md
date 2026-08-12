# Lingman AI Skills 平台

基于 Lingman-Starter 框架（派生自芋道 yudao）的 AI Skills 集合，帮助业务开发者快速生成符合公司规范的代码、SQL、接口文档等。

## 包含的 Skills（12 个 + 1 共享层）

| Skill | 触发场景 | 优先级 |
|-------|---------|--------|
| **crud-generator** | 生成业务模块的 CRUD 代码（Controller/Service/VO/Convert） | 核心 |
| **sql-generator** | 生成建表/改表 SQL、查询 SQL | 核心 |
| **doc-qa** | 查询 API 用法、框架使用问题 | 核心 |
| **error-analyzer** | 分析 Java/Spring/MyBatis/PostgreSQL 错误日志 | 必须有 |
| **permission-generator** | 生成菜单/按钮/接口权限 SQL 和数据权限配置 | 必须有 |
| **code-reviewer** | 检查代码是否符合框架规范 | 推荐 |
| **dict-generator** | 生成字典类型、字典值、枚举类 | 推荐 |
| **api-generator** | 设计 REST API 接口（URL、参数、响应结构） | 推荐 |
| **test-generator** | 生成 Controller 集成测试、Service 单元测试 | 推荐 |
| **project-md-generator** | 生成/规范化项目 AI 记忆 md 文件（CLAUDE.md / AGENTS.md 等） | 推荐 |
| **uni-app-feature** | 开发 uni-app 页面、组件、Pinia 状态、样式与跨端业务功能 | 核心 |
| **uni-app-tooling** | uni-app 工程探测、API 同步、运行构建与跨端排障 | 必须有 |
| **lingman-core** | 共享知识层（框架规范、API 文档、Web 与 uni-app 规范），供所有 Skill 引用 | 依赖 |

## 技术栈规范

- JDK 21 / Spring Boot 3
- PostgreSQL + MyBatis Plus
- MapStruct 转换器（非 BeanUtils）
- `@Resource` 注入（非 `@Autowired`）
- RESTful HTTP 方法（POST/PUT/DELETE/GET）
- URL 前缀：`/admin/{biz}`（kebab-case，如 `/admin/detection-task`）
- ID 策略：PostgreSQL 序列（`@KeySequence`）/ 自增 / 雪花算法（按场景选择）

### uni-app 移动端技术栈

- uni-app，覆盖 APP、H5、微信及其他小程序
- Vue 3 + TypeScript；实际版本以目标项目 `package.json` 为准
- CLI/Vite 或 HBuilderX 工程；先探测真实工程，不凭模板假定构建路线
- Pinia，用于认证、用户、主题、字典、业务角标及跨页面共享状态
- 优先复用目标项目已有 UI 库；p708 同类项目使用 Wot Design Uni
- UnoCSS + scoped SCSS，移动端尺寸优先使用 `rpx`，数值尺寸类显式带单位
- 自动生成 API + 项目请求桥接 + `uni.request`，不另建平行的 axios/fetch 请求体系
- H5、APP、小程序按平台能力适配媒体、网络、权限及生命周期

> 共享规范记录跨项目稳定规则。依赖版本、scripts、包管理器、页面配置源、环境地址和目标平台均须从当前项目实时读取；管理后台的 Element Plus、`@lingman/yd`、Table、Dialog 等规范不得直接套入 uni-app。

## 安装方式

### 方式一：cc-switch（推荐）

[cc-switch](https://github.com/farion1231/cc-switch) 是一个跨平台的 AI CLI 工具管理器，支持一键安装和管理 Skills。

#### 1. 安装 cc-switch

| 平台 | 安装方式 |
|------|---------|
| macOS | `brew tap farion1231/ccswitch && brew install --cask cc-switch` |
| Windows | 下载 `.msi` 或 `.zip` 从 [Releases](https://github.com/farion1231/cc-switch/releases) |
| Linux | 下载 `.deb` / `.rpm` / `.AppImage` 从 [Releases](https://github.com/farion1231/cc-switch/releases) |

#### 2. 安装 Skills

打开 cc-switch → 点击 **Skills** → 输入 GitHub 仓库地址：

```
https://github.com/lingmancn/lm-skills
```

点击 **Install**，cc-switch 会自动克隆仓库并将所有 Skills（含 `lingman-core`）安装到 `~/.cc-switch/skills/` 目录。

> `lingman-core` 为共享知识层（`type: library`、`hidden: true`），cc-switch 会自动一并安装。所有 Skill 通过相对路径引用，无需额外配置。

#### 3. 验证安装

在 Claude Code 中输入 `/skills`，确认以下 Skills 已加载：
- `crud-generator`
- `sql-generator`
- `doc-qa`
- `error-analyzer`
- `permission-generator`
- `code-reviewer`
- `dict-generator`
- `api-generator`
- `test-generator`
- `project-md-generator`
- `uni-app-feature`
- `uni-app-tooling`

### 方式二：手动安装（备用）

如果不用 cc-switch，可手动 clone 并创建 symlink：

```bash
# 1. 克隆本仓库到 Claude Code 全局 Skills 目录
git clone https://github.com/lingmancn/lm-skills.git ~/.claude/skills/lingman-skills

# 2. 创建 symlink
cd ~/.claude/skills
for skill in crud-generator sql-generator doc-qa error-analyzer permission-generator code-reviewer dict-generator api-generator test-generator project-md-generator uni-app-feature uni-app-tooling lingman-core; do
  ln -s lingman-skills/$skill .
done
```

## 目录结构

```
lingman-skills/
├── lingman-core/              # 共享知识层（非 Skill，Skill 引用）
│   ├── framework.md           # 框架核心规范
│   ├── db-spec.md             # 建表规范
│   ├── SKILL.md               # 共享库标记（type: library, hidden: true）
│   ├── api-docs/              # API 文档集合 (20 个文档，10 个模块)
│   │   ├── bpm/               # 工作流 API
│   │   ├── config/            # 参数配置 API
│   │   ├── dept/              # 部门、岗位 API
│   │   ├── dict/              # 字典数据 API
│   │   ├── file/              # 文件存储 API
│   │   ├── logger/            # 日志 API
│   │   ├── mail/              # 邮件发送 API
│   │   ├── notify/            # 站内信、通知分发 API
│   │   ├── permission/        # 权限、角色 API
│   │   ├── sms/               # 短信 API
│   │   ├── social/            # 社交应用 API
│   │   ├── user/              # 管理员用户 API
│   │   └── websocket/         # WebSocket 推送 API
│   ├── frontend/              # 管理后台 Web 规范（列表页/表单弹窗/Search/@lingman/yd）
│   └── uni-app/               # uni-app 共享规范
│       ├── uni-app-spec.md    # 页面、API、状态、UI 与跨端通用规范
│       ├── tooling-guide.md   # 运行、环境、网络、生成与构建指南
│       └── p708-verified-patterns.md # p708 已验证架构模式索引
│
├── crud-generator/            # Skill: CRUD 代码生成
├── sql-generator/             # Skill: SQL 生成
├── doc-qa/                    # Skill: 文档问答
├── error-analyzer/            # Skill: 错误分析
├── permission-generator/      # Skill: 权限配置生成
├── code-reviewer/             # Skill: 代码审查
├── dict-generator/            # Skill: 字典配置生成
├── api-generator/             # Skill: 接口设计
├── test-generator/            # Skill: 测试代码生成
├── project-md-generator/      # Skill: 项目 AI 记忆 md 生成/规范化
├── uni-app-feature/           # Skill: uni-app 页面、组件、状态与跨端业务开发
│   └── evals/evals.json       # 页面/API/Pinia/UI/媒体评测场景
├── uni-app-tooling/           # Skill: uni-app 工具链、API 同步与平台排障
│   └── evals/evals.json       # 运行/网络/401/同步/构建评测场景
```

## 使用方式

在 Claude Code 中直接描述需求，Skills 会根据触发条件自动匹配：

```
"帮我生成一个公告管理模块的 CRUD 代码"    → 触发 crud-generator
"建一张通知配置表，包含标题内容状态字段"   → 触发 sql-generator
"AdminUserApi 的 getUser 方法怎么用？"     → 触发 doc-qa
"分析一下这个错误日志"                      → 触发 error-analyzer
"给 XX 模块配置权限"                        → 触发 permission-generator
"review 一下这段代码"                       → 触发 code-reviewer
"新增一个公告类型字典"                      → 触发 dict-generator
"设计一下公告管理接口"                      → 触发 api-generator
"给公告管理生成测试"                        → 触发 test-generator
"初始化项目，生成 CLAUDE.md"               → 触发 project-md-generator
"给 uni-app 新增业务详情页并接入已有 API"   → 触发 uni-app-feature
"搜索分页列表快速筛选时旧请求覆盖新结果"     → 触发 uni-app-feature
"安卓真机访问电脑后端失败，H5 正常"        → 触发 uni-app-tooling
"后端接口已更新，重新同步 uni-app API"      → 触发 uni-app-tooling
"帮我启动 uni-app 的 H5 项目"              → 触发 uni-app-tooling，并在启动前询问
```

## 与 CLI 工具配合

项目中有一套 CLI 工具（`@lingman/cli`），通过 `lm` 命令提供：
- **从数据库生成 DO + Mapper**（可选，**不推荐日常使用**，以开发者自行维护 DO/Mapper 为准）：`lm mapper` / `lm mapper -n`（强制更新）
- **从 Swagger 生成前端 API**：`lm api`
- **CLI 自身更新**：`lm u`

### CLI 环境要求

- **Node.js 22+**（推荐）
- 项目根目录需配置 `lingman.config.json`

### CLI 安装（用户手动安装）

```bash
# 1. 配置 npm registry
npm config set registry https://registry.npmmirror.com/
npm config set @lingman:registry=https://git.lingman.tech:8081/repository/npm_hosted/

# 2. 安装（如已安装旧版可先执行 npm uninstall -g lingman-cli）
npm install @lingman/cli -g
```

**注意**：Skills 不会自动安装 CLI 工具。如果使用 `lm` 命令时提示未安装，请按以上步骤手动安装。

### 项目配置文件

```json
{
  "lang": "java",
  "template": "",
  "db": {
    "url": "jdbc:postgresql://<host>:<port>/<database>",
    "username": "<username>",
    "password": "<password>",
    "hasBase": false
  },
  "fileOverride": false
}
```

- `fileOverride`：是否覆盖已存在文件。`true` 覆盖更新，`false` 不覆盖（默认）
- 数据库连接信息需根据项目 `application.yaml` 中的数据源配置替换

### 命令运行目录

| 命令 | 运行目录 |
|------|---------|
| `lm init java` | 后端工程根目录（含 `pom.xml`） |
| `lm mapper` / `lm mapper -n` | 后端工程根目录（含 `lingman.config.json` + `pom.xml`） |
| `lm api` | 前端工程根目录 |
| `lm u` | 任意目录 |

> `lm mapper` / `lm mapper -n` 为只读同步命令；**不推荐日常使用**，DO/Mapper 以开发者自行创建维护为准，仅在全新库批量初始化时可考虑，且生成后须人工核对。

### 与 Skills 的分工

| 层 | 生成方式 | 负责方 |
|----|----------|--------|
| DO (`models/entity/`) | 开发者自行创建维护（`lm mapper` 仅可选辅助，不推荐） | 开发者 |
| Mapper (`models/mapper/`) | 开发者自行创建维护（`lm mapper` 仅可选辅助，不推荐） | 开发者 |
| VO/Convert/Service/Controller | AI 根据业务需求生成 | crud-generator Skill |

典型工作流：**开发者自行创建 DO/Mapper → crud-generator 生成 Service/VO/Controller**

### 移动端接口设计、同步与页面接入分工

| 阶段 | 负责方 | 主要职责 |
|------|--------|----------|
| 接口契约设计 | `api-generator` | 设计移动端 URL、HTTP 方法、请求参数、响应结构、权限与 Swagger 契约 |
| 后端业务实现 | `crud-generator` / 后端开发者 | 实现 Controller、Service、VO；DO、Mapper 仍由开发者维护 |
| Swagger 更新 | 后端工程 | 保证实现、接口文档与最新契约一致 |
| 前端 API 同步 | `uni-app-tooling` + `lm api` | 在前端根目录探测配置并执行同步，检查生成差异与类型 |
| 页面 API 接入 | `uni-app-feature` | 从真实自动目录导入 API，复用请求桥接并处理页面状态 |
| 静态与平台验证 | `uni-app-tooling` | 执行真实存在的 typecheck、lint、目标平台 build；运行态验证前询问 |

推荐流程：

```text
api-generator 设计接口契约
  → 后端实现并更新 Swagger
  → uni-app-tooling 在前端根目录执行 lm api
  → uni-app-feature 将生成 API 接入页面或业务组件
  → uni-app-tooling 执行静态检查和目标平台构建
  → 获得用户同意后进行 H5、模拟器或真机运行验证
```

自动 API 约束：

- 自动生成 API、类型和元数据目录禁止手工修改
- 接口缺失时先更新后端契约或 Swagger，再执行项目真实同步流程
- 不在自动目录创建空壳 API、伪造 DTO 或重复手写 URL
- uni-app 请求适配放在自动目录外的请求桥接或窄幅 wrapper 中
- 页面优先使用 `Awaited<ReturnType<typeof Api.method>>` 推导返回类型
- Java `Long` 按生成类型和项目现状处理，不得无条件转为 `number`

### uni-app 开发验证

完成移动端业务开发后，先读取目标项目 `package.json#scripts`，只执行真实存在的命令：

1. typecheck
2. lint
3. 受影响平台的一次性 build
4. 检查 pages/manifest 生成结果及 `git diff`
5. dev、模拟器、真机、开发者工具或 HBuilderX 验证前，先征得用户同意

```bash
# 仅当目标项目真实定义对应 script 时执行
<项目包管理器> run <typecheck-script>
<项目包管理器> run <lint-script>
<项目包管理器> run <目标平台-build-script>
```

> 不默认执行会修改文件的 `lint:fix`；H5 build 成功不能代表 APP 或小程序构建通过；APP build 也不一定直接产出 APK/IPA。

## 扩展计划

- [ ] bpm-generator — Flowable 流程配置生成
- [ ] cli-tool-guide — CLI 工具使用说明
