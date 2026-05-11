# Lingman AI Skills 平台

基于 Lingman-Starter 框架（派生自芋道 yudao）的 AI Skills 集合，帮助业务开发者快速生成符合公司规范的代码、SQL、接口文档等。

## 包含的 Skills（9 个 + 1 共享层）

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
| **lingman-core** | 共享知识层（框架规范、API 文档、前端规范），供所有 Skill 引用 | 依赖 |

## 技术栈规范

- JDK 21 / Spring Boot 3
- PostgreSQL + MyBatis Plus
- MapStruct 转换器（非 BeanUtils）
- `@Resource` 注入（非 `@Autowired`）
- RESTful HTTP 方法（POST/PUT/DELETE/GET）
- URL 前缀：`/app/{biz}`（kebab-case，如 `/app/detection-task`）
- ID 策略：PostgreSQL 序列（`@KeySequence`）/ 自增 / 雪花算法（按场景选择）

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

### 方式二：手动安装（备用）

如果不用 cc-switch，可手动 clone 并创建 symlink：

```bash
# 1. 克隆本仓库到 Claude Code 全局 Skills 目录
git clone https://github.com/lingmancn/lm-skills.git ~/.claude/skills/lingman-skills

# 2. 创建 symlink
cd ~/.claude/skills
for skill in crud-generator sql-generator doc-qa error-analyzer permission-generator code-reviewer dict-generator api-generator test-generator lingman-core; do
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
│   └── frontend/              # 前端规范（待补充）
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
```

## 与 CLI 工具配合

项目中有一套 CLI 工具（`@lingman/cli`），通过 `lm` 命令提供：
- **从数据库生成 DO + Mapper**：`lm mapper` / `lm mapper -n`（强制更新）
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

> `lm mapper` / `lm mapper -n` 为只读同步命令，Skills 可直接帮助用户运行，无需额外许可。

### 与 Skills 的分工

| 层 | 生成方式 | 负责方 |
|----|----------|--------|
| DO (`models/entity/`) | CLI 工具从数据库表结构生成 | CLI 工具 |
| Mapper (`models/mapper/`) | CLI 工具从数据库表结构生成 | CLI 工具 |
| VO/Convert/Service/Controller | AI 根据业务需求生成 | crud-generator Skill |

典型工作流：**CLI 生成 DO/Mapper → crud-generator 生成 Service/VO/Controller**

## 扩展计划

- [ ] bpm-generator — Flowable 流程配置生成
- [ ] cli-tool-guide — CLI 工具使用说明
- [ ] frontend-api-guide — Vue 前端接口对接指南
