# P2 — 编译与人工审阅：实施 TODO

> 配套设计：[P2_IMPLEMENTATION_PLAN.md](P2_IMPLEMENTATION_PLAN.md)。架构与上位验收：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) 的 P2-G。  
> 本清单描述待实施工作，不表示功能已经支持。终点为 **Reviewed Draft + Integration Intent**。

## 1. Coding Agent 执行约束

- 按当前实现仓库及相关组件仓库的 `AGENTS.md`、`docs/AGENTS.md`、`tests/AGENTS.md` 工作；开始任务前重新核对代码，不能把规划中的建议模块当成已存在实现。
- 保持 P1 Source/Knowledge、扫描、Capture、认证及 Desktop Host 行为。复用 TypeBox、Vitest、现有 SQLite/HTTP/文件 helpers，不新建通用工作流、Provider 平台或独立验证框架。
- 不实现 Integration Planner、ChangeSet、Canonical 写入、Source 归档标记写入、自动批处理或后续科研功能。
- 真实测试资料仅在隔离 Vault 的副本中使用。测试证据、质量评估、临时 fixture 来源说明放实现仓库 `.local/p2/`；维护性接口和操作说明放 `docs/`。不把原始私人资料、API key 或模型原始日志提交进仓库。
- 新 revision、新快照、新 Review 均采用不覆盖发布；Source Annotation 更新只使用计划定义的受限 prepare/apply/commit。不能为了尽快通过演示直接覆盖原 Draft 或重写整份 Source。
- 每项完成以可观察行为与对应验证为依据；不以类型检查通过代替保存、恢复或用户交互验收。实际模型调用和 Obsidian 现场操作不能由 HTTP stub 代替。
- 不为后续阶段预建目录、空模块、接口和占位页面。下文新增模块位置是职责建议，实施时可按当前代码的最小合理拆分调整。

## 2. 推荐顺序与依赖

| 顺序 | 任务 | 直接依赖 | 本组出口 |
|---|---|---|---|
| A | A01 → A02/A03 → A04 | P1 基础 | 最小契约、长期资产与任务隔离 |
| B | B01/B02 → B03 → B04 → B05 → B06 → B07 → B08 | A；逐项依赖见下文 | 首条真实 Source 到 Reviewed 的链路 |
| C | C01 → C02；C03；C04 → C05 | B；C04 依赖 C02 | 三种 Review Note 动作与版本保护 |
| D | D01、D02 | C | Desktop 可操作、失败及恢复可解释 |
| E | E01 → E02 → E03 | B—D | P2-G 与维护性文档 |

依赖就绪时，Provider 适配和上下文构建可独立推进；不要求并行。优先完成 B08，再扩展审阅动作。A01 中 Obsidian 保存/Annotation 的小范围技术验证应前置，避免到 C 阶段才发现路径不成立；该验证不等于提前制作完整插件。

## 3. A — 契约与持久化基础

### P2-A01 — 冻结最小跨组件契约与实现决定

**依赖：** P1。**涉及：** `packages/contracts`、现有 Core 文档、Obsidian 目标运行环境。

- [ ] 对照主计划 §2、§14 核对当前 P1 类型、路由、SQLite DDL、Job 查询和文件发布行为，标出具体差异；不重做 P1。
- [ ] 冻结主计划 §6—§11 的 TypeBox 字段、UUID/hash/路径约定、动作判别联合、长度限制、状态含义、错误及版本支持范围。共享一个契约包，不维护插件的第二套 Schema。
- [ ] 落实一个真实 OpenAI-compatible base_url/model、凭据环境变量和 token 参数；锁定目标 Obsidian API/minAppVersion 与本地 contracts 构建包的安装方式。
- [ ] 在隔离样本上验证原生请求接入现有 Core、编辑缓冲区检查与 `Vault.process` 的 BOM/CRLF 保留。若不成立，记录明确阻塞和最小修正，先更新契约，不改成无条件覆盖。

**完成条件：** D01—D10 均有明确实现选项或有界技术验证出口；两个人工确认点、P2 终点和归档边界未变。新增类型只声明当前任务需要的能力。

### P2-A02 — SQLite v1→v2 迁移

**依赖：** P2-A01。**涉及：** `packages/core/src/storage/database.ts`、存储测试与启动检查。

- [ ] 增加一次明确的迁移，保留 documents/meta 与现有扫描记录；扩展 jobs，增加 drafts、draft_actions、review_metadata 三张投影表。
- [ ] 区分文件资产、工作流投影、短期动作参数和可重建 AI 元数据；不得将 Draft 正文或 Integration Intent 仅存数据库。
- [ ] 更新严格 DDL 校验、schema version 和扫描 active 唯一约束的 kind 范围；设置版本不受支持时的明确错误。
- [ ] 用真实 SQLite 验证有数据的 v1 迁移、再次打开、迁移失败保留原库及较新 schema 拒绝。备份/恢复操作不能覆盖唯一原库。

**完成条件：** 既有 Source/Knowledge 可查询；迁移失败无隐式清库；新旧 Job 能表达各自字段。

### P2-A03 — Draft / Snapshot / Review 文件资产基础

**依赖：** P2-A01。**涉及：** 复用 `files/paths.ts`、`files/read.ts`、`files/publication.ts`；新增凝聚的 Draft Store 模块。

- [ ] 实现受限的 Draft、snapshot、revision manifest、Review 与 Idea 路径生成和解析，不扩大 P1 Scan Roots、documents 或 Capture 的范围。
- [ ] 按主计划 §7 创建 Markdown 与最后发布的 JSON 完成描述；动作前快照独立可发现，保留 hash、来源版本、原因与 action_id。
- [ ] 验证线性 revision 链、父节点、完整性与原生工作文件；文件名/目录身份与 Frontmatter 正文语义分离。
- [ ] 在文件集成测试中覆盖路径逃逸/重解析点、同路径碰撞、部分发布、历史文件 hash 不匹配，以及发布失败不删除用户文件。

**完成条件：** 能创建并重新读取一个 Draft 与快照；停止 Core 或移走测试数据库后，文件仍可识别和阅读。复用 P1 安全原语，不复制一套路径检查。

### P2-A04 — 扫描与推理 Job 隔离

**依赖：** P2-A02。**涉及：** `jobs/scans.ts`、`jobs/retention.ts`、新增具体 KnowledgeJobs 执行器。

- [ ] 将 scan_vault 的 active、restart、list、retention 查询限制在其 kind；保留 `/v1/status.active_job` 的扫描含义。
- [ ] 支持 compile_source、recompile_draft、analyze_relations，只有一个串行推理执行器；编译内的关系分析是顺序步骤，不能嵌套等待同一队列。
- [ ] 保留 queued/running/succeeded/failed/interrupted 生命周期；重启不自动调用远端。清理完成 Job 不清理用户文件或尚需恢复的动作。
- [ ] 验证扫描和推理互不误判 busy、恢复不会跨类型中断、局部关系失败不撤销完整 Draft。

**完成条件：** 旧扫描契约可用；新增任务能独立报告进度、错误和中断，尚不需要任何通用 Scheduler。

## 4. B — 首条可运行 Vertical Slice

### P2-B01 — 单一 OpenAI-compatible Adapter 与配置

**依赖：** P2-A01。**涉及：** Core 配置读取与单一 inference 模块。

- [ ] 从 Vault 外 inference.json 和指定环境变量读取配置；实现非流式 Chat Completions 兼容子集、超时、响应上限与确定性 JSON 校验入口。
- [ ] 独立维护 Compiler/Relation 指令，支持主计划指定的两个 `90_System/Prompts` 覆盖文件；记录所用 hash 和版本，不创建配置管理平台。
- [ ] 提供脱敏设置/明确连接验证能力；连接验证不读取 Vault，日志不输出 key、完整请求或私有全文，不自动重试或切换服务。
- [ ] 用本地 HTTP stub 覆盖认证失败、超时、错误体、截断/超限响应、无效 JSON；用所选真实服务完成一次最小兼容性验证。

**完成条件：** Headless Core 可使用已配置服务；没有配置时错误清晰。真实服务能力验证与普通自动化测试分开。

### P2-B02 — 编译候选与有界 Context Builder

**依赖：** P2-A01。**涉及：** 复用 Source parser、Registry、P1 Search/文件读取；新增编译上下文构建。

- [ ] 候选来自已登记 ready Source，但返回和执行前重读当前 Record；默认排除 archived，仅显式请求允许已归档输入。
- [ ] 初版只接受单个 web/manual inline Source；分离原文、metadata、Annotation、Review Instruction 与 Existing Knowledge。
- [ ] 复用字面检索构建至多 5 个 Knowledge 候选，允许有界显式候选路径；重读实际全文、记录版本，按整份候选控制预算。
- [ ] 验证归档状态在排队期间改变、类型/Asset 不支持、Source 超限、陈旧索引/缺失 Knowledge 和资料内命令式文本。Source 超限必须拒绝，不能静默截断。

**完成条件：** 输出可以解释每段内容来自哪里、使用哪个版本和哪些上下文被省略；不会把 snippet 或 Annotation 当作原始材料。

### P2-B03 — 初次 Compiler 到完整 Draft

**依赖：** P2-A03、P2-A04、P2-B01、P2-B02。**涉及：** Compiler、Draft Store、编译动作记录。

- [ ] 以 request_id 登记初编译动作和 Job；同 ID 同参数复用，同 ID 不同参数冲突，同 Source 的并发初编译不重复启动。
- [ ] 校验 CompilerOutput，Core 添加受控来源属性；分开保存 Markdown、AI Notes、规则/模型/来源与上下文版本。
- [ ] 成功输出新工作文件、基线快照和完成描述文件；失败不暴露半成品 Draft，发布后 Source 已变化则标 source_changed。
- [ ] 验证重复提交、发布中断、结果格式错误和输入变化；已完整发布却未更新 Job 时能从文件恢复，不能再次调用 Compiler。

**完成条件：** 一个显式动作得到可独立阅读、带有效 Source 引用的 Draft。Source 原文、Annotation 和 `40_Knowledge` 不变。

### P2-B04 — 审阅前的基础 Relation Analyzer

**依赖：** P2-B03。**涉及：** 独立 Relation 指令、review_metadata 与建议校验。

- [ ] 对生成的确切 revision/hash 和有限 Knowledge 候选分析；没有候选直接 empty，模型不能指定候选集外路径。
- [ ] 校验类型、目标和证据片段，分离 pending/running/ready/empty/failed/stale/unavailable；实际语义质量交由真实样本人工核验。
- [ ] 在首次 Review 开放前完成一次尝试；关系失败保留 Draft，返回明确警告，不假造建议或写入链接。
- [ ] 读取侧边栏状态时核对内容与来源/Knowledge 版本；变化使建议 stale，不隐式调用模型。提供独立显式重新分析动作。

**完成条件：** 关系信息可用于审阅参考，严格绑定所分析内容；不能作为已确认关系或正式整合计划。

### P2-B05 — Review Complete 与持久交接

**依赖：** P2-A03、P2-B03、P2-B04。**涉及：** Review Actions、ReviewHandoff、Draft 状态计算。

- [ ] 校验当前链尾、已保存 content_hash 与有效来源属性；pending/running 的关系尝试完成后才接受确认。
- [ ] 发布确切 Reviewed Snapshot，然后发布 review.json；包含 action/request hash、Intent、编译时及确认时 Source 版本、关系状态。
- [ ] 根据 active revision 与当前 hash 计算 needs_review/awaiting_integration；后续编辑或新 revision 不继承旧确认。
- [ ] 验证非空/空 Intent、文件发布中断、重复提交/响应丢失、Source 变化/缺失、确认后继续编辑和被修改的历史快照。

**完成条件：** 停止 Core 仍能阅读被认可的正文与 Intent；没有 Planner、ChangeSet、Canonical 写入或归档属性更新。

### P2-B06 — 暴露第一条链路所需 Core API

**依赖：** P2-A04、P2-B01、P2-B04、P2-B05。**涉及：** `packages/contracts`、Core HTTP handlers、现有认证。

- [ ] 注册已实现的配置/连接验证、候选、compilations、Draft 查询、Review Complete、动作查询及 Review 读取路由；未实现分支不要伪返回成功。
- [ ] Job 列表增加显式 kind 过滤，默认 scan_vault；P2 客户端请求 all/具体类型，保持旧扫描包络和字段。
- [ ] 统一 UUID、版本前置条件、请求 hash、状态码、错误和 input_consumed 语义；初编译复用同一动作记录及重试边界。
- [ ] 用有限 HTTP 集成验证 loopback/Host/token/Origin 防线、越权路径和同 ID 参数冲突；不新增任意文件编辑/HTTP 转发接口。

**完成条件：** 独立 Core 能通过公开合同完成初编译和正文确认；旧 P1 客户端不会在默认列表收到未知 Job。

### P2-B07 — Obsidian 最小原生 Review

**依赖：** P2-B06。**涉及：** `engramweave-obsidian` 仓库；TypeScript、原生 ItemView、同版 contracts 构建包。

- [ ] 建立必要插件入口、设置和构建安装步骤；只配置本机 Core 地址/token，检查 Core 版本与 Vault 一致性。
- [ ] 通过原生请求读取当前 Draft；打开普通 Markdown，侧边栏显示 Annotation、AI Notes、关系及新鲜度，来源使用链接。
- [ ] 实现首条切片的空 Intent Review Complete；执行前检查编辑缓冲区与磁盘一致，再传 Core 返回的 byte hash 作为前置条件。
- [ ] 手工验证原生编辑/保存、切换 Note、未保存内容、服务断线和接错 Vault；不使用私有强制保存 API，不扩展 Core CORS。

**完成条件：** 用户能在原生编辑器修改 Draft 并确认确切保存版本；插件没有 AI 调用、SQLite 或正文编辑器实现。

### P2-B08 — 首条真实 Web Clipper 端到端切片

**依赖：** P2-B07。**涉及：** 复用 P1 真实 fixture 准备方式、隔离 Vault、用户配置的真实服务。

- [ ] 使用 P1 真实 R1 Source 副本和一个相关 Knowledge；从显式编译开始，经关系分析、Obsidian 修改、Review Complete 到持久交接。
- [ ] 核验正文忠实度、来源/限制条件、用户修改进入确认快照；即使剪藏原文不完整也不能补编原文事实。
- [ ] 对照输入输出字节，确认 Source 和既有 Knowledge 没有变化；停止 Core 后检查 Draft、Review 和 Intent 可直接读取。

**完成条件：** 首条真实链路完成。若真实服务或 Obsidian 路径未完成验证，保留未勾选，不能仅凭 stub 标记通过。

## 5. C — 审阅动作、版本与并发保护

### P2-C01 — Annotation 受限追加与输入消费

**依赖：** P2-A03、P2-B07；沿用 A01 的原生编辑验证。**涉及：** Core Annotation 编辑计划、插件文件适配、动作记录。

- [ ] Prepare 保存动作前快照，产生只修改 Annotation 的确定性编辑计划；保留 Source 正文、其他 YAML、原有 Annotation、archived、BOM 和换行。
- [ ] 插件使用 `Vault.process` 内的文本基线检查应用计划；Source 有未保存编辑时阻止，不把 YAML 解析和拼接业务放进插件。
- [ ] Commit 由 Core 重读校验，再接受推理任务；添加 prepared 查询/继续及受限放弃能力，冲突终止能释放动作占用，不撤销已保存 Annotation。
- [ ] 测试缺失/空/多行 Annotation、中文引号、CRLF/BOM、重复提交、写入后丢响应、写入后再编辑、prepared 放弃和无法无损编辑的拒绝路径。

**完成条件：** 一段指令最多追加一次，成功保存与 AI 成败分开报告；失败时正文、已有 Annotation 和用户输入可恢复。

### P2-C02 — Recompile、幂等重试和新 revision

**依赖：** P2-C01、P2-B04。**涉及：** Review Actions、Compiler、KnowledgeJobs、Draft Store。

- [ ] 空指令直接创建显式重编译动作；非空指令只有 Annotation Commit 成功后才能进入推理步骤。
- [ ] 新 revision 只创建新文件；父 revision 和动作前快照可追溯，不覆盖用户继续编辑的旧文件。
- [ ] 新 revision 重新分析关系；已归档 Source 的显式 Recompile 合法，但不清除/重写 archived。
- [ ] 重试不再次追加；输入、Prompt 或配置变化则冲突。覆盖原进程中断、远端结果未知、重复响应、Compiler 成功/Relation 失败的不同结果。

**完成条件：** 编译失败保留已有资料与指令，成功可比较新旧内容；不承诺跨远端服务的恰好一次计费。

### P2-C03 — 新建 Idea 与双回链

**依赖：** P2-A03、P2-B07。**涉及：** Review Actions、Idea 的受限文件发布与插件入口。

- [ ] 使用动作 UUID 创建 `10_Ideas` 内文件；只将 Review Note 作为正文，添加有效 Source 与当次 Draft 回链。
- [ ] 拒绝全空白/超限输入；重复动作验证同一产物，相同目标但内容冲突不覆盖。
- [ ] 文件完整发布后才报告 input_consumed；不调用 AI、不修改 Source Annotation 或 Draft 正文。

**完成条件：** Idea 可由 Obsidian 独立打开并沿双回链追溯；丢响应后的查询不会再次创建。

### P2-C04 — 确定性 diff 与保留历史的 rollback

**依赖：** P2-C02。**涉及：** Draft Store、成熟文本 diff 依赖、版本 API 与插件版本视图。

- [ ] 提供版本/快照列表，比较生成基线、动作前编辑快照及当前文件；比较当前内容前通过 snapshot 动作保存有描述的快照，不让 GET diff 隐式写文件。
- [ ] 使用确定性行级 diff；输入只允许当前 Draft 的受限 snapshot ID，禁止任意路径读文件。
- [ ] Rollback 先保护当前编辑，再复制选定快照创建新的链尾；旧版本全部保留，不回滚 Source Annotation。
- [ ] 恢复后的内容 needs_review，关系重新分析；验证越界快照、损坏快照、非链尾动作、用户编辑保护及不得继承旧 Review。

**完成条件：** 可以看到 Recompile 对用户编辑的影响并恢复选定内容；无需 AI 解释差异或 Compiler 重生成旧正文。

### P2-C05 — 完整 Obsidian Review Note 工作流

**依赖：** P2-C02、P2-C03、P2-C04。**涉及：** 插件输入状态、动作控制、版本/关系展示。

- [ ] 在同一输入框提供 Idea、Recompile、Review Complete；只锁定本次动作输入，按 action_id 查询结果和 input_consumed 决定清空。
- [ ] 切换文件绑定正确 draft_id/revision_id，丢响应先查询；只清空提交时的输入版本，保留用户后来输入的内容。
- [ ] 接入版本、diff、rollback、显式重新分析和 prepared 继续/放弃；打开新版本不能抢走已经切换到其他 Note 的用户焦点。
- [ ] 验证重复点击、非空 Intent、提交中继续输入、Annotation 已保存但 AI 失败、Source 脏缓冲区、建议 stale/unavailable；无关系接受按钮和 Integration Planner 入口。

**完成条件：** 三种动作各有唯一长期落点，失败与已消费状态可分辨；原生编辑仍是正文修改的唯一 UI。

## 6. D — Desktop 与恢复

### P2-D01 — Desktop 最小配置与控制入口

**依赖：** P2-B06、P2-C05。**涉及：** `apps/desktop`、`src-tauri/src/routes.rs` 与 Core Host 配置传递。

- [ ] 扩展 Rust 路由白名单至实际使用的 P2 路由，仍禁止任意转发；共享同版 contracts。
- [ ] 提供一个 Provider 的地址/模型/凭据环境变量配置、可用状态与明确连接验证；不在前端保存或展示 API key。
- [ ] 在 Source 提供显式编译入口，在任务/Draft 状态提供正确结果和打开 Obsidian 的动作；展示 archived 默认排除和已有 Draft 引用。
- [ ] 验证已有运行 Core 与 Desktop 启动 Core 的配置一致性、环境变量可见性、编译中断/关系失败和旧扫描状态；不创建 Desktop 审阅编辑器。

**完成条件：** 用户可从控制中心配置和发起工作，在 Obsidian 完成 Review；失败能定位到正确任务，不误导为已归档。

### P2-D02 — 中断恢复、重新登记与安全清理边界

**依赖：** P2-C05。**涉及：** SQLite 启动恢复、Draft Store 文件发现、动作/元数据保留规则。

- [ ] Source/Knowledge 继续使用原 rebuild；实现有界的 `POST /v1/drafts/recover` 重新登记完整 Draft/revision/snapshot/Review，事务性更新投影；恢复唯一链尾与经 hash 验证的确认，缺失/多链尾明确报冲突。
- [ ] 保留不完整资产并报告，不自动删除；已完整发布但数据库未提交时从 manifest 恢复，不再调用 Compiler 或重复消费 Intent。
- [ ] 数据库丢失后不重放 Annotation prepare/commit 或模型任务；建议控制数据缺失显示 unavailable，已有 Source archived 可重新读取。
- [ ] 验证迁移后重启、发布各中断点、模拟数据库丢失、历史文件修改、Job 保留清理；清理不能删除 Annotation、Idea、用户快照或 Intent。

**完成条件：** 数据库仅影响可恢复状态与派生数据；知识文件可独立使用，恢复过程不生产知识、不归档、不重复追加。

## 7. E — 聚焦回归与 P2-G

### P2-E01 — 完成必要自动化回归

**依赖：** P2-C05、P2-D01、P2-D02。**涉及：** 现有测试组织、Core/Desktop/插件各自构建检查。

- [ ] 把前述契约、持久化、幂等、冲突和恢复测试放在职责所属 subsystem；复用已有 fixtures/helpers，避免重复大 E2E。
- [ ] 运行相应仓库的类型检查、构建与测试；Core 仓库按当前 scripts 执行 `npm run typecheck`、`npm test`，按改动需要完成 Desktop Rust/原生桥检查。
- [ ] 对 P1 重点回归：refresh/rebuild、archived 读取、Search/Document 范围、Capture 不覆盖、Origin/Host/token、严格 schema 和扫描生命周期。
- [ ] 确认常规自动化使用本地 stub，不依赖私人 Vault、真实密钥或外网；仅在新修改/失败/未解决风险出现时扩大复测。

**完成条件：** 所需检查通过，失败已处理；没有未验证的资产丢失、错误确认或重复动作问题。

### P2-E02 — 真实模型、Obsidian 与 P2-G 现场验收

**依赖：** P2-B08、P2-E01。**涉及：** 隔离真实样本、实际 Provider、目标 Obsidian 版本。

- [ ] **G1：** 复用两份真实 Web Clipper Source；人工检查来源忠实度、缺失内容、关键条件、Annotation 与事实分离、AI Notes 不进入正文。
- [ ] **G2：** 使用相关、不相关、矛盾 Knowledge 样本，验证有依据的语义建议、empty、失败、过时，以及 Recompile/rollback 后的版本对应。
- [ ] **G3：** 三种 Review Note 动作逐一验证长期落点、双回链、一次追加、Intent、清空时机及断线后结果查询。
- [ ] **G4：** 用户修改 → Recompile → diff → rollback；加入失败/中断和数据库恢复，核验用户内容与 Intent 仍可读取。
- [ ] **G5：** 默认排除 archived、显式重新编译可用；编译/确认/恢复均不改归档值，不写 `40_Knowledge`，不创建 Planner/ChangeSet。
- [ ] 将真实服务/模型、规则版本、目标 Obsidian 版本、样本副本来源与人工结论保存在 `.local/p2/`；不把现场结果或秘密嵌入维护性文档。

**完成条件：** 主计划 §13 的全部 P2-G 与附加条件通过。不能把“有效 JSON”“接口 200”或“生成了一个文件”当作真实知识质量通过。

### P2-E03 — 固化维护性说明与阶段出口

**依赖：** P2-E02。**涉及：** `docs/core.md`、`docs/desktop.md`、P2 契约说明、Obsidian 仓库 README。

- [ ] 维护实际支持的配置/环境变量、启动和插件安装、文件布局、Schema/API、Review 操作、错误及恢复方法，清楚区分已支持与以后阶段。
- [ ] 说明 Reviewed Snapshot 与 Integration Intent 的读取/验证合同，正文确认绑定 revision/hash，Source Changed 和关系状态必须可见；P3 只作为后续消费者说明。
- [ ] 清理不再准确的阶段说明，按真实验收状态更新 P2 清单；不将日志、检查点报告或当前机器路径写入稳定项目文档。

**完成条件：** Coding Agent 或后续维护者能按文档运行 P2、复现关键验收并识别阶段边界；交付止于 Reviewed Draft + Integration Intent。

## 8. 必须保持未实现的清单

以下不是 P2 待办，不应在上述任务中顺手实现：

- Integration Planner、ChangeSet 生成/审批/执行、最终目录决策与 Canonical 写入。
- `processing_status: archived` 的设置/清除，正式归档补偿和关系确认。
- Zotero、PDF/OCR、Research/Evidence、Embedding/向量检索、完整关系图谱。
- 自动编译/自动重试、批处理、Scheduler、多 Provider、模型路由、Agent/MCP。
- 第二个 Markdown 编辑器、通用 Source 覆盖 API、跨设备协作或全量永久操作日志。
