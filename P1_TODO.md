# P1 详细 TODO — Coding Agent 执行清单

> 日期：2026-10-03  
> 更新日期：2026-10-04  
> 设计依据：[P1_IMPLEMENTATION_PLAN.md](P1_IMPLEMENTATION_PLAN.md)。上位范围：[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) P1。  
> 所有项目当前均为未实施/未验收；本轮只完成设计文件。

## 执行约束

1. 严格执行 P1 Scope。不要创建 Compiler、AI、Draft、Review、Relation、Integration、ChangeSet、MCP 或 Scheduler 的实现与占位。
2. 实现工作限于 `D:/code/EngramWeave/engramweave`；默认不修改其他组件仓库。下文模块路径均相对此目录，属于建议的新建位置，不是已存在文件。
3. 真实测试资料只读来源为 `C:/Users/18214/Downloads/testEW/testEW`。所有修改、删除、故障和数据库恢复实验都在隔离副本中进行；不得在用户原 Vault 做破坏性验收。
4. 先交付独立 Core 的第一条 Vertical Slice，再补齐故障语义和 Desktop。不得先花时间搭完整桌面外观或通用框架。
5. 每项完成时同时交付相关验证；断言行为与资产保留，不写只复述实现的测试。实际失败需要修复，不能修改真实样本来让解析器通过。
6. 任务完成不自动代表检查点或 P1-G 完成。未核实的依赖版本、平台能力和手工验收要明确记录，不编造通过结果。
7. 用户资料不因成为 fixture 就自动获得对外发布授权；默认本地使用。未来提交/公开时保留来源、分享范围和原样/派生标记。

## 依赖总览

`T00 → T01 → (T02 + T03) → T04 → T05 → T06 → 检查点 A`

`T06 → T07 → T09`，`T06 → T08`，`T07 + T08 + T09 → T10 → 检查点 B`

`T05 + T07 → T11`，`T06 + T08 + T11 → T12`，`T09 + T10 + T12 → T13 → T14 → P1-G`

T02 与 T03 可以在契约冻结后协调推进；Desktop Host 可在 API 稳定后启动。默认按本清单顺序执行，不要求并行开发。

## T00 — 固定真实输入、环境和最小决定

**依赖：** 无；读取设计第 4.1、14 节。  
**预计落点：** `tests/fixtures/README.md`、本地受控 fixture 目录、`docs/p1-decisions.md`。  
**规模：** 小；资料与决定记录。

- [ ] T00-A 为 R1a/R1b 建立原样副本，核对设计中的文件名、哈希、字节数、CRLF/LF；记录原路径和已知采集背景。
- [ ] T00-B 写出基线预期：2 个 Source、0 个 Knowledge、日期字符串、空 Annotation、作者数组、description 保留、4 个真实查询词。
- [ ] T00-C 记录 Windows/文件系统/Node LTS、SQLite 驱动与 Tauri 工具链的具体版本，关闭 D02/D08 的环境决定；不要把计划版本写成已安装版本。
- [ ] T00-D 固定 D03—D06/D09 的最小行为；标注实际浏览器模板/扩展版本未核实时的状态、性能基准机器与 fixture 分享范围。

**验收：** 主输入为真实文件且可复核；样本未被格式化，缺少 40_Knowledge 明确视为空库。  
**验证：** 比较原件与副本哈希；审查 fixture 说明，确认没有将 description 拼入正文或伪造非空 Annotation。

## T01 — 建立最小工程、运行配置和契约

**依赖：** T00。  
**预计落点：** 根 workspace 配置、`packages/contracts/`、`packages/core/src/{main,config,http}.ts`。  
**规模：** 中；只建立 P1 必需入口。

- [ ] T01-A 建立 TypeScript/npm workspaces 与最小构建/类型/测试命令，锁定实际依赖版本，不创建未来业务包。
- [ ] T01-B 按设计第 6、9 节定义共享 JSON Schema、类型推导、错误模型、路径和 Job/Source/Document/Search 契约；客户端不复制第二份契约。
- [ ] T01-C 实现单 Vault 配置加载，拒绝 Vault 内 data_dir、非法 host、冲突端口及同数据库的第二实例；不创建完整 Vault 空目录树。
- [ ] T01-D 提供独立 Core 入口与 health，落实 loopback、认证 token、Host/Origin 限制和日志脱敏；无数据库初始化时不假报 ready。

**验收：** 无 Desktop 也能启动/停止；合法配置与非法配置行为可区分，token 不出现在日志。  
**验证：** 启动冒烟、配置边界测试、未认证请求和非预期 Origin/Host 测试；契约能用于运行时校验与类型检查。

## T02 — 统一文件读取与 Source 解析

**依赖：** T01。  
**预计落点：** `packages/core/src/files/`、`packages/core/src/source/parse.ts` 及相邻测试。  
**规模：** 中；受限路径和一套解析器。

- [ ] T02-A 实现 Vault 相对路径规范化和范围检查，拒绝路径穿越、盘符、ADS、symlink/junction 及大小写冲突，不使用简单字符串前缀校验。
- [ ] T02-B 支持严格 UTF-8、BOM、LF/CRLF、Frontmatter/正文边界，计算原始字节 SHA-256；读取不写回原文件。
- [ ] T02-C 按第 6.1 节处理 null Annotation、日期字符串、作者/标签、未知 metadata 和可选 processing_status；缺失/空值按未归档读取，archived 与登记状态分开；区分 invalid、unsupported 与仅警告，拒绝重复键及不受支持结构。
- [ ] T02-D 提供已有 Knowledge 的只读解析与标题回退；40_Knowledge 中的 raw_source 冲突不能按 Canonical 返回。

**验收：** R1a/R1b 的 metadata、Annotation 与正文分别正确；片段、转义与代码块不被修复；未要求永久机器 ID。  
**验证：** 真实样本断言和针对性派生错误用例；检查解析前后字节哈希不变。

## T03 — 建立 SQLite Registry 与投影存储

**依赖：** T01；与 T02 共享已冻结的数据模型。  
**预计落点：** `packages/core/src/storage/{database,registry}.ts`、初始 schema 及相邻测试。  
**规模：** 中；三张表与事务接口。

- [ ] T03-A 建立 documents/jobs/meta 与 schema 版本，落实 path_key 唯一和单活动扫描约束；不创建 Draft/Review 等表。
- [ ] T03-B 提供登记读取、同路径更新、异常状态与 missing 投影；不同路径相同 URL/hash 不自动合并。
- [ ] T03-C 将文件投影、索引代次和成功 Job 结果放在同一次发布事务中；对非法 schema 或不匹配 Vault 明确报错。
- [ ] T03-D 正文、Annotation 和 metadata 的派生存储保持分列和可重建性质；归档属性从 metadata_json 读取，不新增状态机；错误/缺失记录不参与搜索，已归档记录仍可检索。

**验收：** 重复写入收敛到同一记录，发布失败不出现半代索引；数据库不要求用户文件补字段。  
**验证：** 使用临时真实 SQLite 验证唯一性、事务回滚、Vault 绑定与不支持版本；不要只 mock SQL 返回值。

## T04 — 实现显式 Source Discovery

**依赖：** T02、T03。  
**预计落点：** `packages/core/src/discovery/scan.ts`、扫描结果模型及相邻测试。  
**规模：** 中；一个全范围扫描流程。

- [ ] T04-A 只扫描 20_Sources/40_Knowledge 的受支持 Markdown，排除隐藏/临时目录和特殊链接；根目录从未存在时允许空范围。
- [ ] T04-B 稳定读取、计算 hash、解析并准备本代投影；refresh/rebuild 按设计区分解析复用和强制重建。
- [ ] T04-C 实现单文件与总量限额、分类计数、逐文件诊断；完整枚举失败不得做 missing 清扫或发布部分结果。
- [ ] T04-D 发布成功后得到确定的 added/updated/unchanged/missing/invalid/unsupported 与 generation；unchanged 和文件 indexed_at 只依据实际内容读取校验，不以路径发现替代；不写回 Source。

**验收：** 真实两文件副本可登记为 2 个 Source、0 个 Knowledge，重扫无重复记录。  
**验证：** 临时 Vault + SQLite 集成验证；目录枚举失败、文件持续变化与总量超限时上一代结果保留。

## T05 — 接入最小 Job Engine 与登记 API

**依赖：** T04。  
**预计落点：** `packages/core/src/jobs/`、`packages/core/src/http/{scans,jobs,sources,status}.ts`。  
**规模：** 中；一类任务及其 HTTP 外壳。

- [ ] T05-A 实现 scan_vault 的 queued/running/succeeded/failed/interrupted 和 refresh/rebuild 参数；只允许一个活动扫描。
- [ ] T05-B 相同活动模式返回已有 Job，不同模式返回 JOB_BUSY；完成后再次显式扫描可新建 Job。
- [ ] T05-C 提供 scans/jobs/sources/status API 的真实行为、分页与错误响应，Source 列表按 path_key 排序，不塞入所有正文。
- [ ] T05-D 报告实际进度和文件级异常；Core 启动不自动扫描，不注册任何编译任务。

**验收：** HTTP 提交迅速返回 Job ID，后续查询能看到真实结果和 Source；无无限队列或任务调度框架。  
**验证：** 真实 HTTP/路由集成测试、并发重复提交、完成与失败状态查询；未知字段/非法 mode 被拒绝。

## T06 — 当前文件读取与基础 Search，完成首条切片

**依赖：** T05。  
**预计落点：** `packages/core/src/search/`、`packages/core/src/http/{documents,search}.ts`、首条端到端测试。  
**规模：** 中；完成独立 Core 的用户可见路径。

- [ ] T06-A Document API 读取当前磁盘文件，分别返回正文、Annotation 和 metadata，并返回当前 Source 归档属性，标记 indexed_revision 与 index_stale；不只返回 SQLite 缓存正文，不因详情读取自动更新索引。
- [ ] T06-B 实现 scope、q、fields、source_type/tag/path_prefix、AND 字面匹配、Unicode 归一化、固定排序和分页。
- [ ] T06-C 结果包含 matched_fields、snippet_field 和索引代次；Annotation 片段标为用户上下文；默认 scope 只查已有 Knowledge。
- [ ] T06-D 执行真实输入首条切片，验证 OpenClaw.NET、573KB、volatile、counter++；在隔离副本增加 K1，验证无机器字段的知识读取与优先级。

**验收：** 独立 Core 可发现、读取并找回真实 Source；数据库只是派生投影，普通读取不会改文件。  
**验证：** 固定真实词语、空 Annotation、中文/符号、多词 AND、字段隔离和知识优先的端到端断言；对比原始哈希。

### 检查点 A — 第一条可运行 Vertical Slice

- [ ] A-01 T00—T06 全部符合各自验收标准，真实 R1a/R1b 已通过完整 Core 路径。
- [ ] A-02 演示 Core 关闭时的独立 Clipper 采集，再显式扫描；记录现场扩展/模板版本，无法核实时如实标记。
- [ ] A-03 交付独立启动、扫描、查看和检索的可复现说明；确认没有 AI/Compiler/Review 代码或 Job。

此时尚不能勾选上位 P1-G。

## T07 — 固定变化、并发与中断恢复行为

**依赖：** T06。  
**预计落点：** discovery/jobs/storage 的既有模块及生命周期集成测试。  
**规模：** 中；补齐现有流程的真实异常行为。

- [ ] T07-A 验证普通编辑（含隔离副本归档属性的变化）仅更新下一次扫描投影；已登记或已归档 Source 仍执行 refresh 变化检测；移动/改名产生 missing+new，同 URL 不同路径保留两份记录。
- [ ] T07-B 无效文件不继续返回旧正文；修复后重新扫描恢复；曾存在的整个扫描根消失时停止发布缺失判定。
- [ ] T07-C 重启时将遗留 queued/running 标 interrupted，等待用户显式重试；清理只保留最近 100 个结束 Job，不能删资产。
- [ ] T07-D 验证活动唯一约束、重复扫描请求、发布前后进程退出和错误状态，无自动重编译或后台重试。

**验收：** 旧投影、新文件与失败状态不会混为一谈；再次执行可收敛且不产生重复工作。  
**验证：** 在隔离环境做进程中断、重启及目录失效试验；比较文件哈希、投影代次和任务状态。

## T08 — 完成 Asset 和 Provenance 的最小读取

**依赖：** T06。  
**预计落点：** `packages/core/src/source/resolve.ts`、Document 响应及引用测试。  
**规模：** 小至中；不涉及内容提取。

- [ ] T08-A 解析 `.source.md` 的本地 Asset 与外部 locator，区分 source_content 和 record_body。
- [ ] T08-B 解析受限 Wiki Link、显示别名和可选 .md；锚点仅保留，不做模糊 basename 猜测。
- [ ] T08-C 检查本地资产存在性并阻止越界引用；外部 URI 标 unverified，不访问网络，不执行任意 scheme。
- [ ] T08-D 在 K1 中关联 R1/A1，验证 Knowledge → Source Record → Asset；每次扫描重新检查本地资产是否仍存在。

**验收：** 即使没有 OCR/PDF 提取，也能展示正确来源与缺失状态；Record 说明不被当成二进制原文。  
**验证：** 完整/缺失/歧义/越界引用、二进制附件哈希、外部 URI 无网络调用；既有真实剪藏读取行为不退化。

## T09 — 实现 Core API 新建 Raw Source

**依赖：** T07；关闭 D07 的文件发布原语验证。  
**预计落点：** `packages/core/src/capture/create.ts`、`packages/core/src/http/captures.ts` 及故障测试。  
**规模：** 中；唯一允许创建 Vault 文件的 Core 业务。

- [ ] T09-A 实现完整 web/manual inline Markdown 校验、目标范围检查及请求大小限制；不接收 Canonical 目标，不自动抓取 URL。
- [ ] T09-B 验证同目录临时文件加无覆盖发布原语；目标出现时不覆盖，既有相同字节重试 200，不同内容 409。
- [ ] T09-C 成功 201 返回 path/revision/created/scan_required，明确保存不等于已经索引；后续显式 scan 走同一 Registry。
- [ ] T09-D 验证发布前中断、发布后响应丢失、数据库失败和并发目标创建；只处理 Core 自有临时文件，不影响其他文件。

**验收：** Core API 构成真实的第二条 Capture 路径；新文件完整可见、重试幂等、现有文件永不被替换。  
**验证：** 在隔离 Vault 中提交真实 Source 副本与 Manual 样本；并发和故障注入后比对目标字节，随后扫描并查询。

## T10 — 完成索引重建与数据库恢复

**依赖：** T07、T08、T09。  
**预计落点：** Core 恢复入口、storage/discovery 既有模块、恢复集成测试和说明。  
**规模：** 中；不建立备份平台。

- [ ] T10-A 实现显式 rebuild，新代发布前保留旧查询投影；不能产生半代结果。
- [ ] T10-B 缺库时建立空库并允许显式扫描；损坏/较新 schema 报错，不自动抹掉数据库。
- [ ] T10-C 提供停止 Core 后隔离数据库及 journal 文件的明确恢复入口；只操作配置数据目录并保留旧备份。
- [ ] T10-D 用 R1、带 Annotation 派生样本、预置归档属性的 R3、K1、A1 验证全部登记与查询可重建；归档属性保持、缺失仍为未归档，重建不触发编译/审核/标记写入；内部 ID/Job 历史允许变化。

**验收：** 数据库故障不导致任何用户资产丢失；恢复失败仍能诊断并保留文件与备份。  
**验证：** 临时库删除/损坏/版本不匹配演练；恢复前后比较资产哈希、路径、关键字段与语义查询结果，不要求旧 ID 相同。

### 检查点 B — 完整 Core P1 能力

- [ ] B-01 两条 Capture 路径、显式扫描、基础查找与来源定位均通过集成验证。
- [ ] B-02 变化、并发、中断和数据库恢复符合设计，无资产覆盖、错误删除或自动编译。
- [ ] B-03 Core 能单独演示全部 P1 后端能力，Desktop 只需调用既有 API。

## T11 — 实现 Desktop 的 Core Host

**依赖：** T05、T07；建议在检查点 A 后进行。  
**预计落点：** `apps/desktop/src-tauri/` 的进程管理与固定 API 桥接。  
**规模：** 中；限制为本机运行。

- [ ] T11-A 建立最小 Tauri/React 壳，使用锁定 Node 路径和构建后的 Core 入口，不嵌入另一套 TypeScript 业务。
- [ ] T11-B 区分自己启动和连接已有 Core；只停止自己持有的子进程，握手校验 API 版本、实例和 Vault。
- [ ] T11-C 通过受限命令和固定路由桥接访问 Core，token 不进入网页存储，禁止任意 shell 命令/任意文件读取。
- [ ] T11-D 处理启动失败、端口冲突、断线、退出和重连；不删除数据库、Vault 或不确定归属的锁。

**验收：** Desktop 可管理同一个独立 Core，关闭 Desktop 不误杀用户自行启动的 Core。  
**验证：** 两种进程归属的人工冒烟；检查受限命令、握手及失败提示；确认 Desktop 无直接 SQLite 或 Vault 业务访问。

## T12 — 实现最小状态、Sources/Jobs 与 Search 界面

**依赖：** T06、T08、T11。  
**预计落点：** `apps/desktop/src/` 中入口、Core client 和三组轻量视图。  
**规模：** 中；不做完整产品导航。

- [ ] T12-A 显示 Core/Vault/最后扫描时间，提供扫描、索引重建及受限启停动作。
- [ ] T12-B 显示 Source 元数据/Annotation、Asset 状态和 Job 进度/错误，将已归档/未归档与登记状态分开展示；不提供归档动作、完整网页或正文编辑器。
- [ ] T12-C 提供基础搜索范围、结果和字段标签，保留 generation/新鲜度信息；不得把 Annotation 命中标成原文事实。
- [ ] T12-D 提供用户点击触发的 Obsidian/原网页打开动作，限制 scheme；活动期间轮询，结束后停止，无后台任务重提。

**验收：** 用户可通过桌面完成 P1 操作且看懂空库、失败、缺失资产和过时索引；没有 P2 按钮或伪功能占位。  
**验证：** 使用真实样本副本手工走一遍查询和扫描；检查空状态、失败状态、断线与外部打开。

## T13 — 完成契约、故障、边界与性能回归

**依赖：** T09、T10、T12。  
**预计落点：** `tests/integration/`、`tests/e2e/` 中聚焦 P1 的验证及结果记录。  
**规模：** 中；复用已有测试，补齐实际缺口。

- [ ] T13-A 核对九个 API 的输入/输出 Schema、错误码、分页和重复请求行为；没有隐含写入或任意读取端点。
- [ ] T13-B 覆盖路径穿越、symlink/junction、非法 Origin/Host、未知字段、大小限制与 YAML 边界；异常请求不改变用户资产。
- [ ] T13-C 复核任务发布、Capture 发布和恢复中断窗口，确认结果与契约一致；避免为相同风险重复建设测试套件。
- [ ] T13-D 在记录条件的 500 份/25 MiB 基准上测扫描耗时和热查询 p95，对照目标并报告限制。
- [ ] T13-E 运行工程的类型检查、聚焦测试和 Core/Desktop 本机构建，记录实际命令与结果；不声称已经完成跨平台发行。

**验收：** 核心风险有可复核证据，无未解释的失败；非功能目标达标或明确列为 P1-G 阻断项。  
**验证：** 汇总已有证据，只有新增失败或缺口才扩展测试；不得用仅 mock 的绿色结果替代 SQLite/文件/进程验证。

## T14 — 完成 P1-G 验收与交接

**依赖：** T13。  
**预计落点：** `docs/p1-acceptance.md`、运行/恢复说明，以及本清单和总体 TODO 状态。  
**规模：** 小；最终证据核对。

- [ ] T14-A 按设计第 13.3 节逐项执行 G1—G5，链接真实样本、命令、结果和资产哈希证据。
- [ ] T14-B 编写独立 Core、Desktop、扫描、API Capture、查询和恢复的可复现使用说明。
- [ ] T14-C 明确限制：单 Vault、显式扫描、只查 Source/Knowledge、改名产生新 ID、外部 URI 未在线验证、不恢复旧 Job 历史。
- [ ] T14-D 交接文件 Schema、API、revision/索引新鲜度和归档属性恢复边界；明确 P1 只读、P2 筛选、P3 成功后写标记，确认没有提前建设后续业务。
- [ ] T14-E 只有全部通过后更新本清单与总体 TODO 的 P1 状态；不得勾选 P2 或其他阶段。

**验收：** 文档、接口与实际运行一致，真实数据路径和恢复路径均得到验证。  
**验证：** 以设计 P1-G 对照表审核证据，缺一项则保留未完成状态并写清下一步。

## P1-G 总验收勾选

- [ ] G1 独立 Core 与 Desktop Host 两种方式均可用，知识资产和运行数据分离。
- [ ] G2 真实 Clipper 离线采集与 Core API 新建两条路径均能登记；未改造 Clipper 采集主流程。
- [ ] G3 metadata、Annotation 与原始正文分离，归档属性与 ready 独立且不影响登记检索；来源可定位，重复发现及请求幂等。
- [ ] G4 数据库/索引丢失或损坏后可从文件恢复全部登记及归档属性，Source、Annotation、Knowledge 与附件字节不变；恢复不触发编译或审核。
- [ ] G5 既有知识可读取与检索，普通编辑不触发编译，Core 不写 Draft/Canonical。
- [ ] **P1-G：检查点 A/B、T00—T14 与 G1—G5 全部验收通过，P1 才算完成。**

## 与总体 TODO 的对应关系

| 总体项目 | 本清单承接任务 |
|---|---|
| P1-01 Core / Registry / Job / Desktop | T01、T03—T05、T07、T11—T12 |
| P1-02 文件与运行数据边界 | T01—T03、T09—T10、T13 |
| P1-03 Manual / Web Clipper 两条路径 | T00、T04—T06、T09 |
| P1-04 Asset / Record / Annotation / Derived | T02、T03、T06、T08 |
| P1-05 已有知识读取与查找 | T02、T06、T12 |
| P1-06 保留、登记、重建及无自动编译 | T07、T09—T10、T13—T14 |
| P1-G 阶段验收 | 检查点 A/B、T14、G1—G5 |
