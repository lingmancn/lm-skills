# 分层 md 文件指南

> 分层 md 只在**庞大项目**里使用（判定见 [../SKILL.md](../SKILL.md)「何时生成分层 md」）。它的作用是"渐进式披露"：AI 进入某个子目录工作时，IDE 会自动加载该目录下的 md，拿到只与这一层相关的高密度信息，而不必每次读整个根 md。

## 命名

分层 md 的文件名**与根 md 一致**：

- 根是 `AGENTS.md` → 分层也叫 `AGENTS.md`
- 根是 `CLAUDE.md` → 分层也叫 `CLAUDE.md`

这样 Claude Code / Codex / Cursor 进入子目录时会自动加载对应文件。不要起 `layer-note.md`、`README.md` 之类不会被 IDE 自动识别的名字（除非该目录本来就需要一个给人看的 `README.md`，那是另一回事）。

## 放置位置示例

### 后端 SpringBoot（Lingman 框架）

```
src/main/java/com/lm/app/
├── controller/
│   └── admin/
│       └── AGENTS.md        ← controller 层：接口路径、参数约定、Swagger、AdminDeleteReqVO 等
├── service/
│   └── AGENTS.md            ← service 层：事务、校验、异常抛出、日志、MapStruct 转换
├── convert/
│   └── AGENTS.md            ← convert 层：MapStruct 单例、分页转换
├── models/
│   └── AGENTS.md            ← DO / Mapper 维护约定（开发者自行维护，不自动生成）
└── enums/
    └── AGENTS.md            ← 错误码段、枚举约定
```

真实示例路径：

- `src/main/java/com/lm/app/controller/AGENTS.md`
- `D:\project\lmProject\p706\p706-api\src\main\java\com\lm\app\service\AGENTS.md`

### 前端 Vue / uni-app

纯 Web Vue 可按 `views/components/api/store/utils` 放置；发现 uni-app 信号时必须 **APP 优先**，先围绕 `manifest.json`、`pages.json`、APP 入口、原生能力与条件编译组织分层说明，不要默认按浏览器项目处理。

```
src/
├── views/
│   └── AGENTS.md            ← 页面规范：列表页必须 useTable + <Table>、表单弹窗 <Dialog>+<Form>
├── components/
│   └── AGENTS.md            ← 组件约定：优先复用 @lingman/yd，禁止重复封装
├── api/
│   └── AGENTS.md            ← 接口调用：src/api/auto/ 由 lm api 自动生成，禁止手改
├── store/  (或 stores/、pinia)
│   └── AGENTS.md            ← 状态管理约定
└── utils/
    └── AGENTS.md            ← 工具函数约定
```

uni-app 可按真实目录补充：

```text
src/  (或项目根目录，按实际结构)
├── pages/
│   └── AGENTS.md            ← APP 页面生命周期、导航、平台差异与条件编译
├── subPackages/             ← 仅真实存在时；可在约定密集的业务分包内放同名 md
├── components/
│   └── AGENTS.md            ← uni-app 组件兼容性、easycom/第三方组件约定
├── api/
│   └── AGENTS.md            ← 请求封装、鉴权、环境基址；只写变量名，不写 token/密钥值
├── store/
│   └── AGENTS.md            ← 跨页面状态与持久化约定
├── uni_modules/
│   └── AGENTS.md            ← 插件来源、可修改边界、APP 原生依赖
└── nativeplugins/  (或其他原生插件目录)
    └── AGENTS.md            ← Android/iOS 配置、权限、签名与打包注意事项
```

APP、小程序、H5 有差异时，在最近的分层 md 中明确“适用端、条件编译入口、不可共用能力”；APP 规则优先写，其他端仅补充差异。签名证书、AppSecret、推送密钥等只写文件路径或变量名，绝不写真实值。

### monorepo

```text
repo/
├── AGENTS.md                ← workspace、统一包管理器、跨项目命令与全仓约束
├── apps/
│   ├── admin/AGENTS.md      ← 该应用入口、命令作用域、端特有规则
│   └── app/AGENTS.md        ← uni-app/APP 规则与运行发行方式
└── packages/
    └── shared/AGENTS.md     ← 公共包 API、构建和兼容边界
```

根 md 不展开每个应用细节；应用/包分层 md 必须注明命令从仓库根还是当前目录执行。若出现多个锁文件，先依据 `packageManager`、workspace、CI 判断主包管理器；不能唯一判断时只记录冲突并询问用户，不执行安装、不删除锁文件。

> 不是每个目录都要放。只在该层“有足够多 AI 必须知道的约定/坑”时才放一个；一个只有两三个文件的目录不需要单独的 md。

## 分层 md 结构（自由，不套根模板）

分层 md **不使用**根 md 的章节模板。推荐这个轻量结构，按需增减：

```markdown
# {层名} 层说明

> 一句话本层职责

## 关键文件 / 入口
- 列出这一层最常改、最重要的几个文件或包

## 本层约定
- 只写"在本层工作才需要遵守"的规则（接口路径风格、事务边界、命名等）
- 每条带一句"为什么"

## 常见坑
- 真实踩过的坑 + 解法

## 相关文档
- 指向根 md、framework.md、前端规范等链接
```

## 内容取舍（避免和根 md 重复）

- 根 md 写"全局通用"的事（技术栈、顶层命令、公司级 skill 规范、日志要求）。
- 分层 md 写"只有在这一层才会用到"的事（这一层的命名约定、常见错误、关键文件）。
- **同一条规则不要在根 md 和分层 md 里都展开**——根 md 放一句话 + 指向分层 md 的链接即可。

举例：日志结构化公式是公司全局规则，放根 md 的 `## 开发约束`；但"service 层里哪些方法需要打日志、打日志时业务模块名怎么取"这种层内细节，放 service 层的 `AGENTS.md`。

## 命令与敏感配置的层内记录

- 命令必须写明执行目录，并标注验证等级：L1 配置核对、L2 不启动服务的静态验证、L3 完整构建/测试/lint。
- `dev`、`serve`、`start`、APP 调试基座、模拟器/真机运行等启动类命令，未经用户明确同意不得执行；可记录为“L1/L2，未启动验证”。
- 不为验证执行依赖安装或改写锁文件；多锁文件冲突未确认前尤其禁止。
- `.env*`、平台配置、签名证书与密钥仅记录文件位置、变量名、用途和获取渠道；禁止复制密码、token、AppSecret、私钥或证书口令。

## 行数控制

- 分层 md **硬上限 100 行**。
- 超长内容下沉到 `docs/{layer}/` 下更细的文件，在分层 md 留链接。
- 一旦某个分层 md 接近 100 行，说明这一层信息已经多到需要进一步拆分，考虑按子业务再分子目录 md。

## 不要做的事

- ❌ 不要给每个目录无脑铺 md——只在确有内容时才放。
- ❌ 不要把根 md 的内容复制一份到分层 md。
- ❌ 不要在分层 md 里重复"公司 skill 规范""维护触发条件"这些全局段落——它们只属于根 md。
- ❌ 不要用不会被 IDE 自动加载的文件名（如 `_note.md`）。
