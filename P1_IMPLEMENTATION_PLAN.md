# P1 — 本地核心与知识资产基础：详细实施设计

> 日期：2026-10-03  
> 状态：实施设计；本轮未编写或运行产品代码，未将 P1 标记为完成。  
> 上位依据：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) P1、[TODO.md](TODO.md) P1。  
> Coding Agent 清单：[P1_TODO.md](P1_TODO.md)。

## 1. 设计结论与事实基线

P1 交付一个**单 Vault、单 Core 进程、显式扫描、文件为真相源**的本地系统。先通过独立 Core 完成“采集文件 → 登记 → 查看 → 搜索”，再让 Desktop 管理同一个 Core。SQLite 保存文件的可重建投影和当前扫描任务，不保存独占的用户知识。

不改变总体架构。本设计将 P1 中的 Job Engine 收窄为一个扫描任务执行器，将基础 Knowledge Search 收窄为已有 Markdown 的关键词和少量元数据查询，将 Derived Representation 收窄为可重建的文本检索投影；不提前建设 P2 及以后的业务。

### 1.1 已核实的本地事实

| 依据 | 事实及设计影响 |
|---|---|
| `D:/code/EngramWeave/engramweave` | 检查时目录为空；下文代码目录是未来实现建议，不是已存在模块。 |
| 两份总体方案 | `D:/document/EngramWeave/` 与本 `doc/` 中的 v0.4 内容哈希一致；以本目录副本作为便于携带的文档引用。 |
| [Clipper 模板](../engramweave-web-clipper/engreamweave/EngramWeave-Web-source.json) | 使用上游模板；正文为 `content`，目录为 `20_Sources/Web/YYYY-MM/`，文件名来自网页标题。模板包含 `title/source/published/captured_at/author/annotation/type/source_type`。注意资源目录实际拼写是 `engreamweave`。 |
| [Clipper 说明](../engramweave-web-clipper/engreamweave/README.md) | Clipper 只采集，不调用 Core，不创建内部 ID 或编译任务；P1 保持这条直接文件路径。 |
| [Frontmatter 序列化](../engramweave-web-clipper/src/utils/shared.ts) | 空文本可能输出 `annotation:`，即 YAML null；`captured_at` 使用 date 类型，通常为 `YYYY-MM-DD`；作者可能是列表。不能只接受非空 Annotation 或完整时间戳。模板字段是 `published`，不能按 README 中举例的 `published_date` 硬编码。 |
| 真实采集样本 | 用户提供 `C:/Users/18214/Downloads/testEW/testEW`。已读取其中两份真实 web Source，具体基线见第 4.1 节；二者都有 null Annotation、日期字符串、作者列表及模板文件未声明的 description 扩展字段。测试 Vault 当前没有 40_Knowledge。 |

### 1.2 本设计采用的最小选择

- 首轮验收环境建议为当前 Windows 开发环境、本地 NTFS、一个已有 Vault；不承诺跨平台、网络盘、云占位文件或多 Vault 同时运行。
- 只扫描 `20_Sources/` 和 `40_Knowledge/` 的 Markdown。非 Markdown Asset 仅通过 Source Record 引用检查，不做内容提取。
- 由用户或客户端显式请求扫描；Core 启动只恢复运行状态，不默认扫描。不做 watcher、定时扫描或 Scheduler。
- 全量枚举当前范围，按文件原始字节 SHA-256 判断变化。P1 不做依赖 mtime 的快速路径或增量文件事件系统。
- 一个活动扫描任务、一个 SQLite 连接、一次完成的扫描原子发布一代投影；不引入队列服务、事件总线、工作线程池或事件溯源。
- 基础查找使用规范化文本的字面包含匹配；暂不使用 FTS、分词服务、Embedding、向量库、关系图或结果重排模型。
- 两条 Capture 路径均保留：真实 Web Clipper 直接落盘；Core API 接受调用者提供的完整 Raw Source Markdown 并仅创建新文件。P1 不要求 Clipper 调用该 API。

## 2. Scope / Non-goals

### 2.1 Scope

| 能力 | P1 完成边界 |
|---|---|
| Core 独立运行 | 配置一个 Vault 和外置应用数据目录，通过 CLI 启停、本地 HTTP 查询；Desktop 不存在时功能仍可用。 |
| Source Discovery / Registry | 显式扫描、解析、校验、登记、变化识别、异常报告；重复扫描幂等；原文件不被修复或改写。 |
| Source 资产模型 | Markdown 合并存储；`.source.md` 引用本地 Asset 或外部 locator；Annotation 与正文分离；未知扩展字段保留。 |
| Job Engine | 只执行 `scan_vault`，支持 `refresh/rebuild` 两种模式、状态查询、并发请求合并和中断识别。 |
| 基础 Knowledge Search | 读取已存在的 `40_Knowledge` Markdown；查询 Source 或已有知识的标题、正文、Annotation 和选定元数据。 |
| Core API Capture | 接受 web/manual 的完整 Markdown，以调用者指定的 Source 相对路径创建文件；禁止覆盖现有不同内容。 |
| Desktop 最小控制中心 | 启停自己启动的 Core、连接已有 Core、查看状态、扫描、Source 列表和基础查询；编辑仍在 Obsidian。 |
| 可恢复性 | 删除或损坏 SQLite 后，文件仍可读；以明确的恢复操作重建登记与搜索；旧 Job 历史允许丢失。 |

### 2.2 Non-goals

- Compiler、LLM / Inference Provider、Prompt 执行、AI Notes、Draft 生成、Review、Recompile、Relation Analyzer、Integration Planner、ChangeSet 和正式知识写入。
- 创建或编辑 Canonical Knowledge、自动修改 Annotation、自动修复来源、给文件追加 `source_id` 或流程字段。
- Zotero 插件与 API 同步、Research / Evidence 工作流、Mobile、OCR、ASR、PDF 提取、图片理解、网络抓取与原网页在线校验。
- Obsidian 工作流插件、通用聊天、完整阅读器、Markdown 编辑器、文件浏览器、知识分类系统。
- 多 Vault、Remote Core、MCP、Agent、定时任务、跨设备同步、冲突合并、可靠分布式队列、永久操作日志。
- 自动 Git 提交、完整历史版本恢复、发行签名、自动升级、跨平台安装包、Node 单文件打包。P1 验证本机可运行 Desktop；发行工程留待后续。
- 预建未来业务的空表、空 API、抽象 Provider、插件协议和模板引擎。`90_System` 中已有配置可保留，P1 不消费 AI 规则或新建空 Prompt。

## 3. Core、Desktop、Vault、SQLite 的边界

| 对象 | 负责 | 允许的写入 | 明确禁止 |
|---|---|---|---|
| Vault | 用户长期资产、原始内容、Annotation、已有知识、来源链接及用户配置 | Obsidian / Clipper / 用户管理文件；Core 只通过 Capture API 创建新的 Source | Core 扫描时修改 Frontmatter、修复正文、迁移目录或删除文件 |
| Core | 路径约束、解析、登记、扫描任务、查询、Capture API、运行状态 | 应用数据；Capture 请求对应的新 Source 文件 | 依赖 Desktop 才能完成业务；写入 `40_Knowledge`；执行 AI |
| SQLite | 文件内容及元数据的派生投影、当前/近期扫描任务、投影代次 | Core 独占数据库访问 | 成为 Source、Annotation 或知识的唯一保存位置；Desktop 直接读写数据库 |
| Desktop React | 展示状态、列表、查询、用户动作 | 通过 Core API 发起扫描；通过壳管理进程 | 自己扫描 Vault、解析 YAML、运行 SQL、实现检索或保存正文 |
| Tauri / Rust 壳 | 进程句柄、启动配置、受限系统打开动作、向本地 Core 转发请求 | 本机启动配置、自己启动的 Core 生命周期 | 重复实现知识业务；开放任意命令执行或任意文件读取 |

运行数据建议位于 `%LOCALAPPDATA%/EngramWeave/p1/`，允许 CLI 显式指定测试数据目录，但必须在 Vault 外。该目录保存 `config.json`、`core.sqlite`、本地认证 token、运行描述及有限日志。SQL 日志、正文和 Annotation 不进入运行日志。

本机配置只含 `config_version=1`、`vault_path`、`data_dir`、`host=127.0.0.1`、`port`。默认端口建议为 `43127`，冲突时显式报错或由用户配置其他端口；不要自动连接未知服务。路径和端口不是知识配置，不放入 `90_System`。Vault 根和数据目录在运行中不可切换，修改后需重启；同一数据库不得静默绑定另一个 Vault。

首次配置不要求创建全部 Vault 目录；缺少 `20_Sources` / `40_Knowledge` 可作为空范围，但必须在状态中说明。Capture API 仅按明确请求创建其目标父目录，不创建完整空目录树。

### 3.1 进程生命周期与本地访问

- Core 先绑定本地端口，取得该实例的运行所有权，再打开数据库；绑定失败的第二实例不能修改第一个实例的 Job。P1 支持一个配置档案，不支持另一个端口绕过单实例约束访问同一数据库。
- 应用数据中的实例锁与健康握手共同用于识别已有 Core；不能只凭残留 PID 杀进程。清理失效锁前确认原实例已退出；无法确认时给出诊断，不抢占。
- Desktop 若连接到已独立启动的 Core，退出时只断开；若 Core 由它启动，显式停止或退出时关闭自己的子进程。优先使用受控进程通信完成停止，再超时终止；不向任意 PID 发信号。
- Core 支持终端启动及正常退出；意外退出后的任务恢复见第 7 节。Desktop 重启不创造新的知识工作流。
- HTTP 仅监听 loopback；除不含 Vault 内容的健康状态外，要求本地随机 token。拒绝不符合绑定地址的 Host 和未经允许的浏览器 Origin，不启用通配 CORS。
- Desktop 通过 Rust 壳的固定路由转发连接 Core，token 留在本机宿主侧；CLI 使用同一权限下的 token。它不是远程网关，不实现账户、角色或 OAuth。
- Tauri 对外部程序的启动及参数权限需要显式约束；P1 先验证受限启动本机 Node 与已构建 Core 的方式，后续打包不能反向污染 Core 业务边界。[Tauri 外部程序说明](https://v2.tauri.app/develop/sidecar/)

## 4. 第一条可运行的 Vertical Slice

**切片用户故事：** 用户在 Core 关闭时，用真实 Web Clipper 保存一篇网页；之后单独启动 Core，显式扫描，在本地 API 中找到这篇 Source，看到来源、Annotation 和原始 Markdown，并用其中的真实关键词找到它。

```text
真实网页 → 现有 Web Clipper 模板 → Vault/20_Sources/Web/YYYY-MM/真实标题.md
                                             ↓
独立 Core → POST /v1/scans → 一个 scan_vault Job
                                             ↓
                         Discovery → Parse → Registry / 文本投影 → SQLite
                                             ↓
               Sources 列表 → Document 读取 → Search → 原来源定位
```

**切片顺序：** T00 取得真实样本 → T01 运行与契约基础 → T02 文件解析 → T03 Registry → T04 扫描 → T05 最小任务与 API → T06 文档读取与关键词检索。

**输入准备：** 将真实剪藏产物按相对路径原样复制到隔离测试 Vault。记录网页 URL、Clipper 版本、模板哈希、文件字节哈希、采集方式及预期字段。空 Annotation 的真实样本也有效；非空 Annotation 和错误输入可以另建明确标记的派生 fixture，但不能替代原始样本。

**演示验收：**

1. Core 关闭期间能由 Clipper 完成采集；启动后健康接口可用，尚未扫描时登记为空。
2. 请求扫描得到 `202` 和 Job ID，任务完成后 Source 列表包含真实文件。
3. Document API 返回当前原文件的元数据、Annotation 与正文，三者不混合；URL 与文件内一致。
4. 用标题或正文中的真实词语查询 `scope=sources` 命中它；单独查询 Annotation 时结果注明命中的是用户上下文。
5. 再次扫描不新增重复记录，第一次与第二次扫描前后 Source 字节完全相同；全程没有 AI 调用或 Draft。

此切片不要求 Desktop、Core API Capture、非 Markdown 资产、完整恢复流程同时完成，因而不能单独代表 P1-G。

### 4.1 已取得的真实样本与确定性预期

样本根：`C:/Users/18214/Downloads/testEW/testEW`。两份文件均位于 `20_Sources/Web/2026-10/`，本轮只读原件，不复制或修改用户 Vault。

| 编号 | 文件名 | 实测字节与编码 | SHA-256 |
|---|---|---|---|
| R1a | `把 OpenClaw.NET 的 83 篇 Markdown 变成一个网站：完整复盘.md` | 1,244 bytes，UTF-8 无 BOM，CRLF | `6A46D92D9C03BB1C86D5DFF939D5C68B9A4F84F139EEBF13EA90B0ACA783BB63` |
| R1b | `并发编程（七）：volatile——从语言规则到 CPU.md` | 1,203 bytes，UTF-8 无 BOM，LF | `10BFE3381131ACAD9BEF633EB64A78C4277199F5DA46600109F7C7F7C2DB6E14` |

确定性断言：

- 仅复制这两份文件的测试 Vault，扫描后有效 Source 数为 2；Knowledge 数为 0，缺少 40_Knowledge 是正常空范围。
- 二者均解析为 `type=raw_source`、`source_type=web`、`captured_at=2026-10-03`、`published=2026-10-03`；`annotation` 读取为 `""`，原文件仍保持空 YAML 值；author 保持列表。
- R1a locator 为 `https://www.cnblogs.com/shanyou/p/23192862`，R1b 为 `https://www.cnblogs.com/ThinkerQAQ/p/23192943`；这些 URL 是待显示的资产来源，不要求测试联网成功。
- 以下查询均显式使用 `scope=sources`：`q=OpenClaw.NET` 命中 R1a；`q=573KB&fields=body` 命中 R1a；`q=volatile` 命中 R1b；正文词 `counter++` 命中 R1b。HTTP 中加号必须编码为 `q=counter%2B%2B&fields=body`，不能被查询串解析为空格。
- 只查询 Annotation 时，两份原样本不能产生非空文本命中；带 Annotation 的用例需另建明确标记的派生副本。
- 两份正文都是实际保存的片段，未必有一级标题；保留原样代码块、Markdown 转义与排版，不补全网页、不重排代码、不把 description 混进原文。
- 重复扫描、查询、重启和重建后，副本的字节哈希保持一致；CRLF 与 LF 不得在扫描时被统一写回。

仓库模板的参考 SHA-256 为 `8A6B92CB13785D034A6823B633B1A92C9A0F78D0F7DF91538C4A0C58DAFB102A`。这不证明浏览器当时安装的模板字节完全相同；真实文件有额外 description，必须兼容。实际扩展版本及浏览器模板版本尚未核实，在后续现场采集验收中记录，不能用仓库 package 版本冒充。

## 5. Source Discovery 与 Registry

### 5.1 扫描范围和触发

只枚举配置 Vault 中 `20_Sources/**/*.md` 与 `40_Knowledge/**/*.md`。所有路径按段检查，不用字符串前缀判断目录归属；跳过临时文件、隐藏目录、符号链接 / junction / reparse point，不跟随它们进入其他位置。其他 Vault 内容保持原样。

`20_Sources` 中只有 Frontmatter 的 `type: raw_source` 才成为有效 Source；缺失该标记的 Markdown 记为 `unsupported` 诊断，不根据网页 URL 猜测。`40_Knowledge` 的普通 Markdown 按其已由用户放入该目录的既有知识读取，不补写类型，不执行 AI 认证。若其中出现 `raw_source` 标记，报告目录/类型冲突并排除查询，不把它升级为 Canonical Knowledge。

扫描与读取都只在受限路径内执行；拒绝绝对路径、`..`、盘符、UNC、Windows ADS 和大小写折叠后发生冲突的路径。保留展示用原始大小写，Windows 登记键使用规范化大小写；不通过模糊文件名跨目录自动寻找 Source。

### 5.2 一次扫描的执行过程

1. 验证 Vault、根目录状态、数据目录边界及扫描限额；保存当前任务状态。
2. 完成范围枚举，逐个稳定读取文件。读取前后文件尺寸/修改信息变化时最多重读一次；仍变化则记录 `FILE_UNSTABLE`，不发布该文件的半截内容。
3. 从原始字节计算 revision（SHA-256），解析 Frontmatter / 正文；读取和检索归一化不回写文件。
4. 构建本轮登记及文本投影。文件级格式错误记入该文件诊断，其他文件继续处理；不可枚举目录、Vault 不可用或资源总量超限属于任务失败。
5. 只有枚举成功且整体处理完成，才用一个 SQLite 事务更新 Registry、搜索投影、missing 状态、投影 generation，以及该任务成功结果。扫描失败保留上一代完整投影。
6. 清楚返回 added / updated / unchanged / missing / invalid / unsupported 数量、警告和完成时间。下一次扫描可重新尝试异常文件。

文件读取的瞬间仍可能与用户操作交错；P1 不宣称提供 Vault 全局文件快照。Source 修改后须显式再扫描，查询结果标明最后索引时间，详情读取以当前文件为准。

**初始保护限额：** 单 Markdown 最大 5 MiB、最多 10,000 个候选 Markdown、本轮累积读取最多 100 MiB、YAML aliases 上限 50。单文件超限产生 `unsupported/FILE_TOO_LARGE`；总量超限使任务失败且不发布缺失判定。限额应集中声明并在状态中可见，不静默截断正文。

### 5.3 身份、更新、删除与改名

| 情况 | P1 行为 |
|---|---|
| 同一路径、同一字节哈希 | 保持内部 ID 和 revision，只更新扫描观测信息；不创建重复处理任务。 |
| 同一路径、内容发生变化 | 保持内部 ID，更新 revision 和当前投影；不保留历史正文，不执行编译。 |
| 不同路径但 URL 或正文相同 | 视为两个文件；不按 URL/hash 合并、不删除任何文件。 |
| 文件改名或移动 | 旧路径标 missing，新路径登记新 ID；P1 不猜测重命名，不修复引用。 |
| 文件暂时无效 | 当前行标 invalid/unsupported，搜索排除；不继续把旧解析内容当成当前有效内容。文件修复并扫描后恢复。 |
| 本轮完整枚举中不存在原路径 | 标 missing、从查询结果排除，不删除用户文件；只有成功完整枚举才做这一判断。 |
| 某个扫描根以前存在、现在整体消失 | 报告范围不可用并失败，不将其中所有文件批量当成用户删除。重新配置范围需要显式处理。 |
| 数据库重建 | 路径、内容与元数据从文件恢复；内部 ID 和旧 Job ID 不保证保留，不写入 Source 文件作为补救。 |

Registry 是文件现状的投影，不能把 `ready` 解释为“已编译”“已确认”或“来源可信”。

## 6. P1 冻结的最小 Schema

### 6.1 文件约定：兼容真实 Clipper，读取宽容、创建明确

Frontmatter 必须是 YAML mapping；拒绝重复关键字段、非预期自定义 tag、无界 aliases 和错误结构。建议使用 YAML 1.2 core schema，保持日期为字符串，不隐式转成 JS Date。[YAML 官方文档](https://eemeli.org/yaml/)

| 属性 | 类型和规则 | 缺失 / 空值处理 |
|---|---|---|
| `type` | Source 必须等于 `raw_source` | 缺失为 unsupported；不补写 |
| `source_type` | 非空字符串；web/manual 是 P1 创建支持范围，其他来源可登记其记录 | 缺失或类型错误为 invalid；不预建全部来源枚举 |
| `title` | 字符串 | 空或缺失时取首个一级标题，否则取文件名；保留原 metadata，不回填文件 |
| `source` | 原始 locator 字符串；web 要求 http/https URL | web 缺失或非法为 invalid；manual 允许无外部 URL |
| `captured_at` | 日期 `YYYY-MM-DD` 或带时区的 ISO 时间字符串 | 缺失/null 为未知并提示；其他文本原样保留并给格式警告；禁止用扫描时间冒充采集时间 |
| `annotation` | 字符串，允许多行；missing/null/空字符串在读取模型中归一为 `""` | 其他类型为 invalid；从正文与一般 metadata 中单独取出；不改写原表示 |
| `asset` | 可选本地 Wiki Link 或允许的外部 URI | 有值时按资产引用处理；不得自动把 Record 正文作为该 Asset 转写 |
| `author`、`tags` | 允许字符串或字符串数组；null 视为空列表；scalar 作为一个值，不猜测逗号拆分 | 原 metadata 保留，查询用归一化列表 |
| `published` 及扩展属性 | JSON 兼容的普通 YAML 数据；不得执行内容 | 原样进入 metadata；不强制改名或丢弃未知字段 |

Canonical 文件不要求新增 Frontmatter。其标题、标签和 `sources` 可以读取；`sources` 支持字符串或字符串数组形式的 Wiki Link。不理解的复杂值保留并报告，不进行自动迁移。

### 6.2 Source / Asset 读取模型

| 字段 | 含义 |
|---|---|
| `record_path` | Vault 相对路径，使用 `/`；Provenance 指向它，API 不要求永久 Source ID |
| `source_type`, `title`, `original_locator`, `captured_at` | 由第 6.1 节解析得到；未知可为 null |
| `annotation` | 用户认知上下文字符串，独立返回 |
| `metadata` | Frontmatter 中除 annotation 外的原始属性；不将运行状态混入 |
| `asset.kind` | `inline_markdown`、`vault_file` 或 `external_ref` |
| `asset.locator` | 当前 Record 路径、本地 Asset 相对路径或外部 URI |
| `asset.availability` | `available/missing/unverified/unsupported`，与 Record 本身是否有效分开 |
| `source_content` | 仅 inline_markdown 返回 Markdown 正文；非 Markdown / 外部 Asset 为 null |
| `record_body` | 独立 `.source.md` 的说明正文；不能当作 PDF、图片等原始内容 |
| `revision` | Record 文件原始字节 SHA-256；不代表外部网页或 PDF 的版本 |

普通 Source `.md` 没有 `asset` 时，正文是 inline Markdown Asset，`source` 只是原始网页 locator。`.source.md` 必须有 `asset`，或有可识别的外部 `source` locator；否则报错，不假装具备原始资料。

P1 解析本地 Wiki Link 时支持 Vault 相对路径和 Record 同目录的显式相对路径，允许显示别名；Provenance 文档链接可省略 `.md`。不支持模糊全 Vault basename 搜索；出现多个候选时标 ambiguous。锚点保留，但不验证具体段落；不实现完整 Obsidian 链接语义。

本地 Asset 只检查定位和可访问性；不读取二进制正文或提取文本。外部 `http/https/zotero` URI 只标为 unverified，不自动联网、下载或判断失效。Canonical → Source Record → Asset 可通过 Document API 连续查看，原始引用字符串始终可见。

### 6.3 内部 SQLite 最小结构

只创建三张表，不引入 ORM、向量扩展、事件流水表或未来工作流表。初始 schema 使用 `PRAGMA user_version=1`；不支持的较新版本拒绝打开，不自动降级。[SQLite user_version](https://www.sqlite.org/pragma.html#pragma_user_version)

| 表 | 最小列组与约束 | 真相归属 |
|---|---|---|
| `documents` | `id` 主键；`path_key` 唯一；`path`；`kind=source/knowledge`；`state=ready/invalid/missing/unsupported`；`revision/size/mtime`；`title/source_type/captured_at/original_locator`；`metadata_json/asset_json/diagnostics_json`；`annotation/body_markdown`；`title_norm/body_norm/annotation_norm/metadata_norm`；`indexed_at` | 全部可从文件重建；body 对独立 Record 是 Record 正文，不能伪称 Asset 内容 |
| `jobs` | `id` 主键；`kind=scan_vault`；`mode=refresh/rebuild`；`status`；`created_at/started_at/finished_at`；`summary_json/error_json`；活动状态唯一约束，防止并发创建两个扫描 | 当前与近期运行状态；无永久知识意义 |
| `meta` | 单行 `vault_path_key/schema_version/index_generation/last_scan_at/known_scan_roots` | 本机绑定与投影状态；重建后重新建立 |

`documents` 同时承担 Source Registry 与已存在 Knowledge 的搜索投影，避免另建一份正文表。`source_type` 等查询列是 metadata 的派生字段。invalid/missing/unsupported 行不保留可供查询的旧正文；只保留诊断、路径及最后观测必要信息。

metadata 查询先按固定字段过滤，tags 可从候选的 `metadata_json` 归一化读取；不提供任意 JSONPath / SQL 查询。内部 UUID 不进入 Markdown；schema 中不出现 compiled、reviewed、draft_id、provider 或 changeset 字段。

扫描完成以事务同时发布文件投影、generation 和成功 Job 结果。进程中断不能使新旧两代投影混合；这是 SQLite 内部事务边界，不包括文件写入。[SQLite 事务说明](https://www.sqlite.org/transactional.html)

## 7. Job Engine：一个受控扫描任务执行器

| 项目 | P1 规则 |
|---|---|
| 任务类型 | 只有 `scan_vault`；两种模式均全量枚举、读取字节并计算 hash。refresh 可复用 hash 相同且 ready 的解析/文本投影，rebuild 强制重新解析；两种模式都重新检查本地 Asset 的存在性，异常记录始终重新尝试。 |
| 状态机 | `queued → running → succeeded / failed / interrupted`。succeeded 可带文件级诊断，不代表所有 Source 有效。 |
| 并发 | 一次最多一个 queued/running。相同 mode 的重复请求返回已有 Job 和 `reused=true`；不同 mode 返回 `409 JOB_BUSY`。不建立无限排队队列。 |
| 幂等 | 同一文件多次扫描不增加记录；同一活动扫描多次请求不生成多份任务。一次已完成后再次显式扫描可有新的 Job ID。 |
| 中断 | 进程重启时把遗留 queued/running 标 interrupted；不自动重试。用户重新请求扫描产生新 Job。 |
| 文件级错误 | 在结果和 Source 列表可见，正常文件继续；无法读取文件的当前投影不可冒充有效。 |
| 任务级错误 | Vault 不可用、目录枚举失败、总量超限或数据库错误：任务失败，上一代完整投影不变。 |
| 进度 | 返回已处理文件数量和最终分类计数，不声称精确剩余时间；日志不含正文或 Annotation。 |
| 保留 | 默认保留最近 100 个结束 Job 及全部活动 Job，清理只作用于 jobs；不是 Maintenance 系统。 |

变更文件的解析与文本索引更新是本阶段基础登记行为；不创建 compile/recompile/re-analyze 等任务，也不把扫描称为“知识编译”。

## 8. 基础 Knowledge Search

### 8.1 查询范围与语义

- `scope=knowledge` 默认，仅搜索已有 `40_Knowledge`；`scope=sources` 搜索有效 Source；`scope=all` 合并两者。
- P1 不扫描 `10_Ideas` 和 `50_Research`，不虚构这两档结果；后续接入时遵守上位优先级。all 中 Canonical 在 Source 前，Source 中 paper 在其他类型前。
- `q` 按空白拆成最多 8 个词，词间 AND；每个词可在选定字段中任意一处命中。中文连续字符按字面子串匹配，不依赖中文分词。
- 文本建立投影时使用统一 Unicode NFC 和 Unicode 小写处理；原文件与 API 返回正文不变。查询同样归一化。SQLite 的 `lower()` 默认只处理 ASCII，因此不将它用于中文/Unicode 归一化；字面包含可使用参数化 `instr`。[SQLite 字符串函数](https://www.sqlite.org/lang_corefunc.html)
- 搜索字段为 `title/body/annotation/metadata`；默认前三者和选定 metadata。metadata 仅包括 source URL、source_type、tags，不把 Job 状态和异常文本拼入语料。
- 元数据过滤只支持 `source_type`、单个 `tag` 和 `path_prefix`；前缀按目录段判断。空 q 必须至少有一个过滤条件，否则 `400 EMPTY_QUERY`。
- `%`、`_`、引号等按字面字符处理，不成为 SQL、正则表达式或高级查询语法。

### 8.2 结果与新鲜度

排序固定为“knowledge 优先 → Source 的 paper 优先 → 标题精确匹配优先 → 标题包含优先 → 其他字段匹配 → path_key”。不引入语义分数或复杂加权模型。

每项返回 `id/path/kind/title/source_type/revision/matched_fields/snippet/snippet_field`，片段最多 240 字符；若命中 Annotation，标签明确为用户上下文；独立 Record 的正文片段标为 Record 说明。元数据片段只显示实际匹配的字段。

列表包含 `total/limit/offset/index_generation/indexed_at`；默认 limit 20，最大 100。offset 分页只对同一 generation 稳定，客户端发现代次变化后重新查询。结果代表最后一次成功扫描，不保证实时；打开详情时重新读取文件并报告 `index_stale`。

P1 基准集建议 500 份 Markdown、合计不超过 25 MiB，在记录硬件条件后验证热查询 p95 ≤ 1 秒、一次扫描 ≤ 15 秒。这是待实测的验收目标，不是已有性能结论；超限优先定位瓶颈，不以此提前引入向量库或独立搜索服务。

## 9. 最小 Core API 与 Capture 写入

### 9.1 API 约定

前缀 `/v1`；JSON UTF-8；运行时间戳使用带时区 ISO 字符串，采集日期保留原精度；所有文件路径为 Vault 相对路径。请求字段不接受未知参数，扩展的 Source metadata 除外。错误统一为 `error.code/message/details`，不返回堆栈、token 或文件正文。Schema-first 契约在 shared contracts 中维护一份，Core 校验与 Desktop 类型使用它。

公共 JSON 字段使用 snake_case；id 是不透明字符串；列表字段始终为数组，未知 scalar 用 null。Source 详情中 inline Source 的 source_content 是字符串、record_body 为 null；独立 Record 的 source_content 为 null、record_body 是说明字符串。Knowledge 详情返回 body 字符串，Source 专属的 source_content/record_body/asset 为 null。没有已发布索引时 indexed_revision/indexed_at 为 null、index_generation 为 0，详情的 index_stale 为 true；以后以当前文件 revision 与已登记 revision 是否相同判定。所有列表统一用 items/total/limit/offset，不混用多种分页包络。

建议采用 TypeScript + npm workspaces，Core 使用 Fastify、`yaml`、`better-sqlite3`；契约使用 JSON Schema 和由其推导的 TypeScript 类型，测试使用 Vitest。仅引入当前 API 必需依赖，不创建通用框架。Fastify 可直接使用请求及响应 Schema；具体版本在 T00 与 Node LTS 一起锁定。[Fastify 校验文档](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/)、[better-sqlite3 官方说明](https://github.com/WiseLibs/better-sqlite3)

| 方法与路径 | 请求 | 成功响应与主要规则 |
|---|---|---|
| `GET /v1/health` | 无 | `200 {status, core_version, api_version}`；不暴露路径。初始化/数据库不可用返回 503，不能假装 ready。 |
| `GET /v1/status` | 认证 | Vault 与 data_dir、运行状态、活动 Job、投影代次/时间、有效与异常记录计数、固定限制；不返回 token。 |
| `POST /v1/scans` | `{mode: refresh 或 rebuild}` | `202 {job, reused}`；活动同类合并，异类冲突 409；请求本身不阻塞等待扫描完成。 |
| `GET /v1/jobs` | `limit/offset` | 当前与近期任务列表，按 created_at 降序、ID 破同序。 |
| `GET /v1/jobs/{id}` | Job ID | 单个 Job 状态、summary、error；未知为 404。 |
| `GET /v1/sources` | `state/source_type/path_prefix/limit/offset` | 返回 id/path/title/source_type/state/revision/original_locator/captured_at/asset.availability/diagnostics，以及分页和索引代次；默认 ready，path_key 升序，允许筛选异常；不返回所有正文。 |
| `GET /v1/documents?path=…` | 限定范围内 Markdown 路径 | 从当前文件读取；返回 kind/path/revision/indexed_revision/index_stale、metadata、annotation、source_content/record_body 或 knowledge body、原始来源引用及解析结果。即使尚未登记也可读取合法路径。 |
| `GET /v1/search` | 第 8 节查询字段 | 带分类和命中字段的搜索结果；数据库未扫描时为空并注明 generation=0。 |
| `POST /v1/captures` | `{path, markdown}` | path 只能位于 `20_Sources`，输入是完整 web/manual inline Raw Source Markdown；创建成功 201，相同路径相同字节重放 200；返回 path/revision/created/scan_required=true。 |

分页参数是整数，默认 limit=20、最大 100、offset≥0；q≤200 字符；search 的 fields 是逗号分隔的 title/body/annotation/metadata 子集。HTTP 运行状态只用 starting/ready/degraded；ready 表示 Core 与数据库可用，不表示已经扫描或知识已经确认。Capture 接收 JSON 最大 8 MiB，解码后 Markdown 最大 5 MiB；过大为 413。常用错误：`VALIDATION_ERROR/EMPTY_QUERY` 400、`UNAUTHORIZED` 401、`PATH_OUTSIDE_SCOPE` 403、`DOCUMENT_NOT_FOUND` 404、`JOB_BUSY/PATH_CONFLICT` 409、`INVALID_SOURCE` 422、`CORE_UNAVAILABLE` 503、`IO_ERROR` 500。诊断细节区分文件问题与运行问题。

不开放任意 SQL、任意文件读写、通用 Job 提交、删除/更新 Source、Annotation 编辑、Canonical 写入或编译端点。停止 Core 由宿主进程控制，不开放无约束 shutdown HTTP 接口。

### 9.2 第二条 Capture 路径：最小且可恢复

Core 不根据 URL 抓网页，不替调用者写总结。调用者传入完整 Markdown；Core 使用同一解析规则验证 web/manual inline Source 后，原样保存 UTF-8 字节。Manual 可以没有外部 URL，但需要可识别的 Source 类型和正文。

**发布规则：**

1. 验证路径、父目录和输入，拒绝覆盖已有不同内容；已有完全相同内容视为幂等成功。幂等键是目标路径 + 内容字节，不是 URL 或 SQLite 中的请求历史。
2. 在目标同目录创建 Core 自己命名的临时文件，排他写入完整内容、flush 并关闭，然后使用本地文件系统支持的“不覆盖已有目标”的发布操作。
3. 本设计对 Windows 本地 NTFS 的首选实现是临时文件创建完成后，用硬链接发布到尚不存在的最终路径，再移除临时名字。必须先通过目标环境原语验证；不支持时明确失败，不回退到可覆盖目标的 rename/copy。
4. 只有最终文件完整可见才返回成功；扫描只识别最终 `.md`，忽略临时文件。进程在发布后、响应前中断，调用者重试同一路径同一内容能收敛。
5. 保存成功不依赖 SQLite 事务，也不自动扫描。响应明确要求显式扫描；登记失败或数据库丢失时，已保存文件仍是资产。临时残留只报告并按 Core 自有文件处理，不在重启时删除其他内容。

这是本设计的发布策略，必须通过并发和故障测试验证。Node 的排他创建可以避免覆盖，而 `copyFile` 不保证原子性，不能把“先判断不存在，再普通写入/复制”当成安全发布。[Node 文件系统说明](https://nodejs.org/api/fs.html)

### 9.3 Schema 冻结范围

开始跨模块编码前固定：文件属性解释、Source/Asset 读取模型、路径语义、Job 状态与重复请求规则、分页/排序、错误码、上述 API 请求响应、数据库初始 schema。允许新增可选响应字段，但修改字段意义或错误行为必须同步契约与测试。

不冻结 P2 的 Draft / Review Schema、未来关系、Provider、MCP 或远程部署协议；不要为它们增加占位参数。数据库内部布局以后可迁移/重建，文件约定与 `/v1` 行为的兼容性优先。

## 10. Desktop 最小交互

只做三组功能，可放在一个窗口内：

1. **连接与状态：** 显示 Vault、Core 状态、最后扫描时间，启动/连接、停止自己启动的 Core、显式扫描和重建索引。
2. **Sources / Jobs：** 显示标题、路径、来源类型、Record/Asset 状态及任务进度/错误；点击 Source 查看元数据和 Annotation，提供在 Obsidian 中打开及打开原网页的动作。
3. **Search：** 输入关键词、选择 knowledge/sources/all，显示标题、路径、命中字段和短片段，在 Obsidian 中继续阅读。

不渲染完整网页 HTML，不提供正文编辑、采集编辑器、AI 设置或 Review 占位界面。外部打开必须是用户点击触发，且 URI scheme 受限；无法处理的 Zotero locator 只显示，不自动执行。活动任务期间可短间隔轮询，无任务时停止轮询；断线保留清楚的状态，禁止无限创建重试任务。

## 11. 重建与故障恢复

| 场景 | 行为与恢复路径 |
|---|---|
| 搜索投影过时 | 显式 refresh；详情读取可提示当前文件与 indexed_revision 不同。 |
| 想重新解析全部文件 | 显式 rebuild；上一代投影在新事务提交前仍可用；文件不变。 |
| 正常进程重启 | 读取配置与 SQLite，标记遗留活动 Job 为 interrupted；等待显式扫描。 |
| SQLite 不存在 | 建立空库，generation=0；Vault 仍可用，用户扫描后重建。 |
| SQLite 损坏或不兼容 | 报错且停止数据库业务，不自动删除文件；记录可操作诊断。 |
| 损坏数据库恢复 | 先停止 Core，再由显式恢复命令将应用数据中的数据库及相关 journal 文件整体隔离备份，创建新库并扫描。恢复失败保留旧备份与新诊断；不触碰 Vault。 |
| Vault 不可用/被移动 | 报错，不把整个库登记为已删除。用户恢复路径或显式修改配置；不得自动搜索其他磁盘认领 Vault。 |

恢复范围只有文件可重建的登记和查找；旧内部 ID、Job 历史和扫描时间不保证恢复。P1 不承诺恢复用户主动删除的文件或外部网站消失的内容；这些依赖 Vault / 外部资产自身备份。

恢复演练必须在临时 Vault 与临时应用数据目录进行，先确认绝对路径范围。禁止测试脚本删除真实 Vault 或真实数据库。用正文、Annotation、已有知识和二进制附件的字节哈希验证恢复前后资产保持不变。

## 12. 按依赖顺序的实施步骤

| 顺序 | 任务 | 本步产出 | 依赖 |
|---|---|---|---|
| 0 | T00 真实样本与决定基线 | 真实 Clipper fixture、环境与行为决定记录 | 本设计 |
| 1 | T01 工程、配置与最小契约 | 独立入口、认证与路径边界、契约目录、health | T00 |
| 2 | T02 文件与 Frontmatter 读取 | 同一套 Source / Knowledge 解析规则 | T01 |
| 3 | T03 SQLite Registry | 三张表、投影写入与事务、schema 版本 | T01 |
| 4 | T04 Discovery | 限定范围枚举、稳定读取、hash 与完整发布 | T02、T03 |
| 5 | T05 最小 Job 与登记 API | 扫描请求、状态查询、Source 列表 | T04 |
| 6 | T06 读取与基础 Search | 当前文档读取、词语及元数据查询、首条切片演示 | T05 |
| 检查点 A | 第一条 Vertical Slice | 真实 Source 在独立 Core 中可发现、读取和找回 | T00—T06 |
| 7 | T07 变化与故障语义 | 重复请求、改名/缺失、无效文件、中断和重试验证 | T06 |
| 8 | T08 Asset 与 Provenance | `.source.md`、本地/外部引用、已有知识来源链 | T06 |
| 9 | T09 Core API Capture | web/manual 新文件创建、幂等和发布恢复 | T07 |
| 10 | T10 重建与恢复 | 索引重建、空库恢复、损坏库隔离恢复 | T07、T08、T09 |
| 检查点 B | Core 完整 P1 能力 | 两条采集路径、资产引用与恢复均可独立验收 | T07—T10 |
| 11 | T11 Desktop Core Host | 启停/连接同一个 Core，受限进程与 API 桥接 | T05、T07 |
| 12 | T12 Desktop 最小界面 | 状态、Source / Job、Search 和原生打开 | T06、T08、T11 |
| 13 | T13 契约与边界回归 | HTTP/文件边界、并发、限额、性能与恢复证据 | T09、T10、T12 |
| 14 | T14 P1-G 验收与交接 | 运行说明、验收报告、已知限制、P2 可用边界 | T13 |

允许 T02/T03 在契约确定后协调推进，T11 可在 Core API 稳定后进行；首个优先目标始终是检查点 A。此表不授权本轮开始编码。

## 13. 测试与 P1-G 验收方案

### 13.1 样本矩阵

| 样本 | 来源 | 用途 |
|---|---|---|
| R1a / R1b 真实 Web Clipper 剪藏 | 第 4.1 节已读取的两份文件，按各自哈希原样留存 | 主要端到端输入；不能用模板渲染模拟替代；保留 CRLF/LF 差异 |
| R2 带用户 Annotation 的剪藏 | 优先真实采集时输入；若由 R1 修改则标明派生 | 多行、引号、中文与上下文独立查询 |
| K1 已有知识 | 在隔离 Vault 中由用户认可的普通 Markdown 或明确标为测试构造的样本 | 无强制机器字段、知识优先、`sources` 回链 |
| A1 独立 Asset | 测试图片/附件 + `.source.md`；另加外部 URI Record | 引用与存在性检查，不验证 OCR 或外部网站存活 |
| E1 异常组 | 从 R1 派生并逐个说明修改 | 空/null 字段、坏 YAML、缺字段、CRLF/BOM、非 UTF-8、超限、重复键 |
| L1 生命周期组 | R1 在临时 Vault 中复制/编辑/移动/删除 | 重复扫描、同 URL 不同路径、改名、missing、扫描失败和重建 |

真实样本在进入仓库前检查可共享性；若含个人信息或不可共享正文，原样私有 fixture 留在本机，公共测试使用标记清楚的派生样本，P1-G 在本地运行真实样本验收并保存不含正文的证据。测试只读取受控副本，不改变用户原剪藏文件。

### 13.2 分层验证

- **解析与契约：** null / 日期 / 作者数组、Frontmatter/Annotation/body 边界、路径约束、支持范围、错误码和限额。
- **集成：** 临时目录 + 真实 SQLite；重复扫描、原子发布、完整枚举失败、无效文件恢复、API 新建与幂等、二进制附件保持不变。
- **真实端到端：** 主流程使用 R1；独立启动 Core → 扫描 → 查询 → 再扫描 → 修改副本 → 再扫描 → 关闭 / 重启 → 重建。
- **桌面人工冒烟：** 创建宿主/连接已有实例两种模式，点击扫描、查看错误和结果、在 Obsidian 打开；关闭 Desktop 的归属行为正确。
- **故障注入：** 进程在扫描发布前后、Capture 发布前后退出；目标路径并发出现；数据库故障；目录不可读；索引重建失败。验证现有文件不被覆盖，不以“测试全部绿”代替资产哈希比较。
- **性能：** 固定样本规模、机器配置和依赖版本记录扫描耗时与查询 p95；结果不达目标时明确报告，不降低功能正确性。

### 13.3 P1-G 对照验收

| 上位标准 | 本轮验收场景与通过条件 | 证据 |
|---|---|---|
| G1 Core 独立 + Desktop 管理，资产/运行数据分离 | 无 Desktop 启动全 Core 流程；Desktop 启动/连接/关闭行为符合归属；SQLite 和日志在 Vault 外 | 两种启动记录、配置、目录检查 |
| G2 两条 Capture 路径，Core 离线仍可采集 | Core 关闭时用真实 Clipper 保存 R1，之后扫描成功；Core API 创建 web/manual 文件后显式扫描，结果语义一致 | 真实采集说明、API/Job 结果、原文件哈希 |
| G3 区分上下文与事实、来源可定位、重复发现幂等 | 分别查询/读取 metadata、annotation、body；K1 → R1 → 网页 locator / A1 文件可追踪；重复扫描和重复请求不重复登记/执行 | 字段断言、查询排序、文件数和 Job 数对照 |
| G4 数据库/索引丢失后资产仍在且可重建 | 在隔离环境清空或损坏 SQLite；原始 Source、Annotation、Knowledge、附件仍可读；恢复后 path/revision/字段和预期检索一致 | 恢复前后资产哈希、语义查询结果；不比较内部 ID/历史 Job |
| G5 既有知识可读取查找，普通编辑不编译 | K1 可通过文档 API 和关键词/metadata 查询找到；编辑 R1 后只改变文件与下次扫描投影 | 无 AI/Compiler/Draft API 或任务类型，30_Drafts/40_Knowledge 无 Core 写入差异 |

只有 G1—G5 全部通过、真实 R1 到位且检查点 A/B 及边界回归完成，才可勾选总体 `P1-G`。文档完成、mock 通过或单一成功场景都不能代替这个门槛。

## 14. 实现前必须决定的问题

以下区分本设计已给出的最小选择与仍需证据的事项；不要求用户为每个实现细节再次审批。

| 编号 | 决定 | 当前采用 / 建议值 | 关闭条件与影响 |
|---|---|---|---|
| D01 | 真实主样本来自哪里 | 已取得 testEW Vault 中 R1a/R1b，见第 4.1 节 | 来源决定已解决；T00 将其落实为受控 fixture 与预期断言，仍不表示端到端运行已通过 |
| D02 | 首轮平台和存储范围 | 当前 Windows、本地 NTFS、单 Vault | T00 记录 OS/文件系统/Node LTS；若用户要求网络盘或多平台，先调整发布原语和测试范围 |
| D03 | 扫描发现方式 | 显式全范围扫描，无 watcher/定时任务 | 按此编码；如果后来要求自动发现，另行评估，不偷偷增加后台任务 |
| D04 | 知识读取范围 | 只读现有 40_Knowledge；Source 在 20_Sources | 确认实际 Vault 的目录匹配；不自动移动现有笔记，不为兼容加入自由配置分类系统 |
| D05 | Source 兼容约定 | 允许日期精度和空/null Annotation，记录未知 metadata | 用 R1 核验后固定契约；发现差异先修设计，不改原样本来迎合解析器 |
| D06 | Core API 采集边界 | 完整 Markdown 新建；path+字节幂等；不覆盖、无网络抓取 | 固定为两条路径中第二条的 P1 验收定义，不改造现有 Clipper |
| D07 | 文件发布原语 | 同目录临时文件 + NTFS 硬链接无覆盖发布 | T09 前在锁定环境验证并发/中断；不支持时属于实现阻塞，不能降级为可能覆盖的写法 |
| D08 | 依赖与宿主运行方式 | Node LTS、TypeScript、Fastify、yaml、better-sqlite3、React/Tauri；P1 使用本机 Node | T00 锁版本、确认 SQLite 驱动与 Tauri 工具链可用；这不是整套发行打包决策 |
| D09 | 改名、重建与历史保留 | 改名按 missing+new；ID 可变；任务保留 100 条；不保存正文历史 | 写入使用说明并以测试固定，不为了未来 Draft 引用引入永久机器 ID |
| D10 | 规模与时延目标 | 第 5 节限额；500 份/25 MiB 基准，查询 p95≤1 秒、扫描≤15 秒 | T00 记录机器条件，T13 实测；未达标先分析实际瓶颈 |
| D11 | 真实样本是否可提交仓库 | 默认只在本地受控测试目录使用，公开前核实分享范围 | 在 fixture README 中说明 private/public/derived；避免把用户资料随测试夹具发布 |

若样本或运行环境证据与上述默认值冲突，应更新本文件与 TODO 对应项后再实施相关步骤；其余独立设计工作不因此停止。

## 15. Coding Agent 的交付约束

实现位置建议为 `engramweave/` 下 `packages/contracts`、`packages/core`、`apps/desktop` 和 `tests/fixtures`，不建立额外服务仓库。P1 默认不修改 Web Clipper、Obsidian、Zotero、Mobile 仓库；发现真实输入不兼容时先按既定语义修 Core，只有证据表明模板有错才单独提出变更。

每个 TODO 必须包含实现、针对风险的验证和对应说明；不添加仅复述实现的冗余测试。执行到检查点时交付可复现结果及限制，不以进入后续阶段补齐当前缺口。P1 完成后交接真实 Source Schema、读取与查找接口、文件 revision 语义和恢复边界；P2 继续单独设计，不能以“方便下一阶段”为由提前编码 AI 或 Review。
