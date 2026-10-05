# P2 — 编译与人工审阅：详细实施计划

> 设计状态：待实现。范围终点为 **Reviewed Draft + Integration Intent**。  
> 架构基线：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)。阶段依据：[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)、[TODO.md](TODO.md) 的 P2。  
> 配套任务：[P2_TODO.md](P2_TODO.md)。运行日志、真实资料副本、质量评估结果和临时验收证据放入实现仓库的 `.local/p2/`，不写入维护性文档。

## 1. P2 Scope / Non-goals

P2 在现有本地 Core 上增加一次明确的知识生产过程：用户选择 Source，Core 调用推理 API 生成可编辑 Draft，提供审阅前的关系建议，用户在 Obsidian 原生编辑器修改，最后明确提交一份正文快照及整合意图。最终产物仍属于 Draft，不因审阅完成而进入 Canonical Knowledge。

### 1.1 Scope

| 能力 | P2 交付范围 |
|---|---|
| Inference Provider | 一个 OpenAI-compatible HTTP API，支持自定义地址、模型与凭据引用；Compiler 与 Relation Analyzer 使用独立指令。 |
| 编译输入 | 当前 Source Record、用户 Annotation、inline Markdown 原文、有限的已存在 Knowledge 上下文；先支持 web/manual。 |
| Compiler | 生成 Markdown 知识候选及独立 AI Notes；保留来源、限定条件和原文的不确定性。 |
| 基础 Relation Analyzer | 在审阅前分析当前 Draft 与检索到的 Canonical 候选，返回有依据的语义关系和可能重复/更新等线索；不写关系。 |
| Draft 资产 | 普通 Markdown 工作文件、生成基线、动作前快照、revision/diff/rollback、可恢复的已审阅产物。 |
| Obsidian Review | 原生编辑 + 侧边栏，统一 Review Note 输入，执行新建灵感、Recompile、Review Complete。 |
| Core 工作流 | 显式编译、受限动作、任务状态、幂等请求、冲突检查和失败恢复。 |
| Desktop | 最小 Provider 配置/连接验证、编译入口、Job 与 Draft 状态、打开 Obsidian；不建设审阅编辑器。 |
| P3 交接 | 可定位、可验证的 Reviewed Draft 快照、Integration Intent、来源版本与关系建议版本信息。 |

首轮仍是 Windows、本地单 Vault、本机 Core、桌面版 Obsidian。推理方式已选择 OpenAI-compatible API；不以某个厂商账号、SDK 或订阅模式作为知识工作流基础。

### 1.2 Non-goals

- Integration Planner、ChangeSet、Diff 审批队列、正式知识创建/更新/移动/删除、Source 归档标记写入。
- 把 Review Complete 解释为归档；把 Draft 移到 `40_Knowledge`；在 Draft 中确认关系。
- Agent Worker、MCP、Scheduler、自动编译、自动重编译、批量无人值守处理、通用工作流引擎。
- 多 Provider 平台、模型市场、自动路由/降级、多 Agent、通用聊天、流式逐 token 编辑、提示词可视化管理平台。
- PDF/OCR/音视频提取、Zotero 接入、Research/Evidence 工作流、Embedding、向量库、完整知识图谱。
- 重写 P1 扫描、搜索、认证、Core Host 或 Capture；自动修复用户 Vault 的结构和链接。
- 每次键入保存一份永久版本、自动 Git 提交、通用文件编辑 API、完整编辑器或实时协同系统。

## 2. P1 复用基线与必要扩展

以下以当前实现的模块与 `docs/p1-contracts.md` 为约束。本阶段只扩展 P2 所需合同，不重新实现已有 P1 能力。

| P1 已有能力/限制 | P2 的复用或最小扩展 |
|---|---|
| `packages/contracts/src/index.ts` 使用 TypeBox，导出 JSON Schema 和类型 | 继续使用 TypeBox；按业务拆分契约文件并保留统一导出，不引入第二套 Schema 工具。 |
| `source/parse.ts` 分离 metadata/annotation/body，解析 archived 属性 | 复用 Source 读取；执行编译前重读当前文件，不只看 Registry。Draft 使用自己的 parser，不能把 30_Drafts 当成 Knowledge。 |
| `files/read.ts`、`files/paths.ts` 的受限路径、UTF-8 和字节 revision | 复用底层路径安全与稳定读取；分别声明 Source、Draft、Idea、配置读取范围，不全局扩大 P1 Scan Roots。 |
| `files/publication.ts` 只支持不覆盖的新文件发布 | 复用发布新的 Draft/revision/快照/Idea/Review 产物；不把硬链接发布冒充安全覆盖 API。 |
| `/v1/search` 与 `search/query.ts` 是字面检索 | 检索现有 Knowledge 候选，再从文件读取实际内容；不把短 snippet 当成完整知识，也不添加向量索引。 |
| `jobs/scans.ts` 是具体扫描执行器，active/recovery/list 当前针对整张 jobs 表 | 扩展 Job 类型前先隔离 scan_vault 的 active/recovery/retention 查询；增加一个具体 KnowledgeJobs 执行器，不做插件式任务框架。 |
| SQLite v1 只接受 documents/jobs/meta，且严格检查 DDL | 做一次有备份、可回退的 v1→v2 迁移；不能直接添加表后仍宣称 schema v1，也不能通过重建丢掉 P2 用户资产。 |
| Core 拒绝浏览器 Origin；Desktop 通过 Rust 原生桥接 | Obsidian 采用其原生请求能力连接 loopback，验证无浏览器 Origin；保持现有 Core 的 CORS/Host 防线。 |
| `processing_status: archived` 为只读长期属性 | 默认候选排除 archived；显式重新编译允许进入；P2 所有路径均保留归档值，不设置或清除它。 |

不改变总体架构，但必须明确三处阶段边界：

1. 总体侧边栏含 Integration Suggestions；P2 不实现该栏目背后的 Planner，也不返回伪造的整合方案。Relation Analyzer 的可能重复/融合线索只是审阅参考。
2. 总体 Relation Analyzer 可使用 Ideas/Research 等上下文；P2 初版只承诺当前 Source 与检索到的 Canonical Knowledge。Idea 创建会交付，但全库 Ideas/Research 检索及关系覆盖延后，不把基础实现描述成完整关系图谱。
3. Draft revision 和 Integration Intent 含用户资产，不能仅放入 SQLite/可清理缓存。其普通文件保存在既有 `30_Drafts` 的明确子目录；Job、AI Notes、当前建议与错误仍留在 Control Plane。

## 3. 分阶段实施步骤与依赖

| 子阶段 | 目标与交付 | 依赖 | 完成条件 |
|---|---|---|---|
| A 契约与持久化基础 | 最小字段、Provider 能力范围、Draft 文件布局、v2 迁移及独立任务类型 | P1 | 路径/权限未放宽，旧 API 和扫描行为保持可用 |
| B 首条编译审阅切片 | 一种真实 API → Draft → 一次关系分析 → Obsidian 编辑 → Review Complete | A | 产生持久 Reviewed Draft 与 Integration Intent，无正式知识写入 |
| C 审阅动作与版本保护 | 新建灵感、Recompile、Annotation 追加、diff、rollback、失效检测 | B | 用户修改可恢复，动作消费与重试一致 |
| D 控制中心与恢复 | Desktop 配置/状态、迁移/恢复、失败与中断处理 | C | 无重复追加、无静默覆盖，数据库丢失不丢长期产物 |
| E P2-G | 聚焦回归、真实模型质量评估和少量跨组件验收 | A—D | P2-G 全部通过，产物边界可供 P3 独立使用 |

建议先完成 A→B 的纵向路径，不先搭建完整 Provider 页面或全部侧边栏样式。B 的首条路径使用空 Review Note 完成正文确认；C 再补齐非空输入、重新编译和失败窗口。真实模型必须在 B 验证，不能等 UI 完成后才检查 API 是否兼容。

## 4. 第一条可运行的 Vertical Slice

**用户故事：** 从 P1 已登记的真实 Web Clipper Source 发起编译，在 Obsidian 中直接编辑生成的 Draft，看到来源和审阅信息，再提交已认可正文与整合意图。提交后仍在 `30_Drafts`，可以停止 Core 后直接打开文件查看。

```text
Current Source + Annotation + Existing Knowledge
  -> Explicit Compile -> OpenAI-compatible API
  -> Draft Markdown + AI Notes
  -> Relation Analysis
  -> Obsidian Native Editor + Review Sidebar
  -> Review Complete
  -> Reviewed Snapshot + Integration Intent
```

切片步骤：

1. 复用 P1 的真实 R1 样本准备方式，在隔离 Vault 中登记 Source；另准备一个与其中概念有关的已存在 Knowledge 样本。样本路径、hash 与评估记录只放 `.local/p2/`。
2. 配置一个 OpenAI-compatible 地址、模型与 API key 的本地引用，运行一次连接验证。远端调用只由明确的连接验证或编译动作触发。
3. 用户点击编译；Core 重读 Source，检查未归档及支持的 inline 类型，检索并读取至多 5 个 Knowledge 候选。
4. Compiler 生成正文与 AI Notes；Core 校验结构并添加受控来源属性，发布一个新 Draft 文件和生成基线。
5. Relation Analyzer 对这个 Draft revision 执行一次分析，侧边栏显示建议或明确的失败/无建议状态。
6. 用户在原生编辑器改正文；插件确认当前编辑内容已保存，Review Complete 绑定其确切字节版本。
7. 保存 Reviewed Draft 快照及 Integration Intent，返回 `awaiting_integration`；既不调用 Planner，也不创建 ChangeSet，不改变 Source 归档值。

**切片验收：** 至少一次真实模型生成完成且质量可人工审阅；确认后的快照包含用户修改；Source 原文和 `40_Knowledge` 字节不变；数据库删除后 Draft 和已确认产物仍可读。此切片通过不代表 Recompile、rollback 等完整 P2-G 已完成。

## 5. 模块和职责边界

| 模块 | 负责 | 不负责 |
|---|---|---|
| Inference Adapter | 一个兼容协议、超时、响应解析和安全错误；接收已组装的任务输入 | 读取整个 Vault、选写入路径、调用工具或自动重试业务动作 |
| Context Builder | 当前 Source 分区、受限 Knowledge 检索与重读、输入预算和截断诊断 | 推断正文事实、构造知识图谱、联网补全原文 |
| Compiler | 正式知识风格的候选正文、独立 AI Notes | 更新 Canonical、认可关系、选择最终知识目录 |
| Relation Analyzer | 针对特定内容 hash 的关系建议和证据说明 | 写入链接、接受关系、合并文件 |
| Draft Store | 新 revision、不可变快照、普通文件引用、diff/恢复和审阅产物 | 编辑器、自动清理用户历史、依赖数据库独占保存正文 |
| Review Actions | 动作消费、版本前置条件、幂等键、Annotation 追加计划、Reviewed 交接 | Integration Planning 或第二次正文确认 |
| KnowledgeJobs | 串行运行编译/关系分析、终止未完成任务、记录近期执行信息 | Scheduler、自动批处理、通用 Agent Harness |
| Obsidian 插件 | 当前 Note 上下文、保存一致性检查、侧边栏、原生文件打开及受限 Annotation 编辑适配 | AI 请求、规则判断、SQLite、Core 的检索和编译实现 |
| Desktop | 配置、状态、显式发起初次编译、打开审阅位置 | 审阅正文、修改 Annotation、替代 Obsidian |

Core 仍独立运行。初次编译、查询、版本资产和已审阅交接不依赖插件常驻；互动审阅由客户端发起。Annotation 的 prepare/commit 协议不依赖 Obsidian 类型，P2 的具体客户端通过 Obsidian 原生文件 API 完成受限写入，Core 保留语义校验和执行决定。

## 6. Inference、输入预算与输出契约

### 6.1 一个明确的兼容子集

使用配置的 API 根地址调用 `POST /chat/completions`，非流式，使用 `model/messages/stream=false` 和经能力验证的输出 token 限制参数。默认只要求普通文本响应中包含 JSON；不假设所有兼容服务支持 JSON Schema response_format、工具调用或同一组扩展参数。基准协议参考 [Chat Completions API](https://developers.openai.com/api/reference/resources/chat)，实际服务兼容性必须单独验证。

配置保存在 Vault 外的本机 `inference.json`：`config_version/base_url/model/api_key_env/timeout_ms/max_output_tokens/output_limit_parameter`。`output_limit_parameter` 明确选 `max_tokens` 或 `max_completion_tokens`；不在失败后偷偷切换参数、模型或服务。默认 timeout 120 秒，每任务一次调用，无自动业务重试；关系分析是第二个独立任务步骤。

API key 从指定环境变量读取，缺失即给出配置错误。Desktop 只显示配置与凭据是否可用，不回传 key；Obsidian 不保存推理服务 key。非 loopback 地址要求 HTTPS；显式配置的本地兼容服务可用 HTTP。没有默认远端地址和自动外发。

Compiler 与 Relation 各有独立默认指令，允许读取 `90_System/Prompts/compiler.md`、`90_System/Prompts/relations.md` 覆盖。不存在时使用仓库内默认文件，不自动覆盖用户配置。每次执行记录所用 prompt hash、输出 Schema 版本、模型与来源 revision；不把完整请求/响应长期写入日志。

### 6.2 输入分区与最小检索

- Source content、Source metadata、用户 Annotation、当前 Review Instruction、Existing Knowledge 使用不同字段，不用拼接后无法区分来源的长文本。
- 当前文件必须由 Core 直接读取。Registry 用于发现候选，不能决定当前归档状态或替代模型输入的原始内容。
- 初版只编译非空 web/manual inline Markdown。独立 Asset、未知类型或内容过大明确报 `UNSUPPORTED_COMPILE_INPUT`，不把 `.source.md` 说明当作 PDF 原文。
- 默认由标题生成最多 5 个短字面查询，复用 P1 Search，去重后最多读取 5 个 `40_Knowledge` 文件；允许一次请求指定不超过 5 个 Knowledge 路径作为补充，不提供复杂检索配置。
- 初始预算建议：Source 正文 40,000 字符、Annotation/Instruction 合计 8,000 字符、Knowledge 合计 20,000 字符、模型 JSON 响应 256 KiB。Source 超限不默默截断；Knowledge 可以按完整候选边界减少，并在 AI Notes 中说明上下文限制。字符上限不是通用 token 精确计量，实际服务上下文上限须在能力验证中记录。
- Source 和 Knowledge 内的命令式内容作为待分析资料，不能改变系统指令、服务地址、文件目标或权限。模型不获得文件、网络或工具执行能力。

### 6.3 需要冻结的 AI 输出

| Schema | 最小内容与校验 |
|---|---|
| `CompilerOutput` | `title/body_markdown/ai_notes[]`；正文非空，长度有界，Core 添加来源 Frontmatter；模型不返回目标文件路径、归档值或任意 metadata 更新。 |
| `AINote` | `kind/message/basis`；kind 为 uncertainty/missing_condition/unsupported_claim/context_limit，basis 区分 source/annotation/knowledge；不把用户猜测标成来源证据。 |
| `RelationSuggestion` | `id/type/target_path/reason/draft_evidence/target_evidence`；目标必须来自传入的候选集，证据片段必须存在于对应输入；语义成立与否仍由人判断。 |
| `RelationSet` | `draft_id/revision_id/content_hash/source_revision/knowledge_revisions/status/items`；完成状态为 ready/empty/failed/stale，不能将旧建议直接用于新内容。未执行完用 pending/running，数据库恢复后没有对应控制数据用 unavailable，不能伪装成 empty。 |

关系类型先限定为 supports、contradicts、extends、derived_from、possible_duplicate、possible_update、possible_merge。单纯主题接近不伪装成语义关系；没有足够依据时返回空列表。模型输出无效、截断或拒绝时产生可解释失败，不通过无限修复 Prompt 或正则猜测输出结构来伪造成功。

## 7. Draft 文件、revision、diff 与 rollback

### 7.1 文件布局与长期资产

以下目录均位于既定顶层目录内，不新增顶层知识平面；所有路径由 Core 生成并校验，模型不能指定路径。

```text
30_Drafts/<draft_id>/
  r-<revision_id>.md
  revisions/<revision_id>.json
  snapshots/<snapshot_id>.md
  snapshots/<snapshot_id>.json
  reviews/<review_id>/draft.md
  reviews/<review_id>/review.json
10_Ideas/<idea_id>.md
```

- `r-*.md` 是 Obsidian 原生可编辑文件，Frontmatter 最小为 `type: draft`、`title`、`sources`。稳定身份来自目录与 revision 文件名，不要求 Source 获得永久机器 ID。
- `revisions/*.json` 是用户版本资产的描述：`schema_version/draft_id/revision_id/parent_revision_id/origin/working_path/baseline_snapshot/source_paths/source_revisions/created_at/action_id`。origin 为 compile/recompile/rollback。不存 Job 状态、AI Notes 或建议。
- `snapshots/*.md` 保存生成基线及明确动作前的已保存正文；同名 JSON 最后发布，记录 `schema_version/snapshot_id/draft_id/revision_id/path/content_hash/source_revisions/reason/created_at/action_id`，reason 为 generation/before_action/comparison。即使动作在新 revision 发布前失败，仍能独立找回用户快照；不监听每次键入创建历史。
- 用户修改保存在当前工作 Markdown；动作前快照保护已修改版本。历史文件默认保留，P2 不提供自动清理策略。
- Idea 的最小 Frontmatter 为 `type: idea/title/sources/drafts`，sources 与 drafts 分别包含 Source Record 和当次 Draft 的有效 Vault 内引用；正文只使用用户输入。idea_id 使用该动作 UUID，重复动作定位同一文件并校验预期内容，不覆盖碰撞文件。
- 版本描述文件在其引用的文件均发布成功后最后发布；只有完整描述文件代表一个完成的 revision。中断残留保留并报告，不自动删除或假装成功。

SQLite 只保存当前选中 revision、工作流状态和这些文件的可重建引用。删除数据库不能删除工作文件、快照、Idea、Source Annotation 或 Integration Intent。用户手工改动历史文件时，hash 不匹配必须提示受损，不能继续称其为原确认快照。

P2 每个 Draft 只建立线性 revision 链，修改动作必须以当前链尾为目标；旧工作文件仍可打开、编辑和比较，但不能悄悄从旧 revision 分叉。恢复时以经过完整性验证的唯一链尾恢复 active revision；多链尾、循环或缺失父节点报告冲突，不按文件修改时间猜测。可从旧快照 rollback 生成新的链尾。

### 7.2 三种不同的“版本”

| 名称 | 含义 |
|---|---|
| `revision_id` | 一次初编译、Recompile 或 rollback 创建的逻辑版本身份。 |
| `content_hash` | 当前 Markdown 原始字节的 SHA-256；用户普通编辑会改变它，但不自动创建新 revision 或 AI 任务。 |
| `snapshot_id` | 保存某一时刻确切字节的不可变资产，用于比较、恢复或正文确认。 |

所有用户动作提交 `revision_id + expected_content_hash`。Core 读取文件确认对应关系；插件同时检查编辑缓冲区与磁盘文本一致。未保存输入不可被旧磁盘版本冒充；不调用未公开的 Obsidian 私有保存接口，必要时提示保存后再执行。

### 7.3 Recompile

1. 客户端固定当前动作输入，提交唯一 `action_id`、当前 revision、expected_content_hash 和 Review Instruction。重复使用同一 ID/参数返回同一动作，不再次追加或计费；同 ID 不同参数返回冲突。
2. Core 校验当前 Draft、Source、归档属性和权限，保存当前已编辑 Draft 快照。已归档 Source 允许用户显式 Recompile，但不清除 archived。
3. 非空 Instruction 按第 8 节追加到 Source Annotation；成功保存后才允许实际推理。空输入表示不追加 Annotation，但仍是明确重新编译动作。
4. 根据 Source、更新后的 Annotation、该次 Instruction 与 Knowledge 运行 Compiler；旧 Draft 可作为标明“用户编辑历史”的辅助输入，但不能把它当作来源事实。
5. 发布一个新的 `r-*.md` 与基线/版本描述文件，绝不覆盖旧工作文件；用户在模型运行期间继续编辑旧文件也不会丢失。客户端只在仍聚焦原工作流时打开新版本，否则提供打开动作。
6. 对新 revision 重新运行 Relation Analyzer。旧关系集合保留为过时参考，不能贴到新 revision 下。

编译失败时旧 Draft 与已追加 Annotation 都保留。重试同一已消费动作不再追加 Annotation；若当前 Source/规则/输入已经改变，原重试拒绝并要求创建新的显式动作，不静默换输入重试。

### 7.4 Diff 与 rollback

- Diff 比较两个明确 hash 对应的快照，或将当前文件先快照再比较；采用成熟文本 diff 库，展示行级增删，不让模型生成差异。
- UI 优先比较“动作前用户编辑快照”和“新生成基线”，另可比较当前工作文件与生成基线。
- Rollback 选择已有快照，先保护当前已保存文件，再将选定内容复制为一个新 revision；旧版本、较新版本和 Source Annotation 都保留。
- 恢复后的版本是未审阅 Draft，重新计算关系或标为待分析；不能因内容与旧 Reviewed Snapshot 相同就自动恢复用户确认。
- Diff/rollback 不修改 Source、不触发 Compiler；关系重算是该明确动作的后续步骤，失败有独立状态。

## 8. Source Annotation 追加：受限客户端编辑与 Core 校验

P1 的无覆盖发布不能安全更新现有 Source。P2 不引入通用文件覆盖接口；采用针对 Recompile 的两步受限动作：Core 生成 Annotation 编辑计划，客户端使用原生文件机制执行，Core 重读确认后提交编译任务。

**Prepare：** Core 保存动作前 Draft 快照，读取当前 Source 字节与 Annotation，生成只能修改 Annotation 节点的确定性文本改动，返回 `source_path/expected_source_revision/expected_text_hash/edit/result_text_hash`。保留原始正文、其他 metadata、原有 Annotation、已有 archived 以及 BOM/换行；不把整份 YAML 重新格式化后覆盖。无法无损修改的特殊结构明确拒绝，不能直接丢弃别的字段。

**Apply：** Obsidian 插件仅应用 Core 返回的受限编辑计划，使用 `Vault.process()` 在回调内比较当前文本与计划基线；不一致就中止。插件不重新实现 YAML/业务拼接算法，不调用 `read` 后无条件 `modify`。若 Source 在其他编辑视图存在未保存修改，先阻止动作并保留输入。官方建议用 process 完成读取后修改，并要求异步计算后的修改再次比较当前文本。[Obsidian Vault 文档](https://docs.obsidian.md/Plugins/Vault)

**Commit：** 客户端报告完成；Core 重新读取并验证计划结果、Source 正文和其他属性未变，才创建 Recompile Job。使用实际磁盘 byte revision 绑定输入，文本 hash 仅服务于插件编辑一致性。插件编辑机制不能保持正文原始字节时，验收失败，不能降低为静默换行转换。

这是客户端执行用户输入的文件适配，不是把 Compiler 或 Source 语义移入插件。Core 不导入 Obsidian API，也不要求插件维持后台连接；其他客户端可以遵守同一 prepare/commit 合同，P2 不为它们建立额外框架。首轮并发保证针对单 Obsidian 实例与 Core 的受控写入；跨设备同步和多个任意外部写入者不在支持范围。

| 中断点 | 行为 |
|---|---|
| Prepare 前置条件失败 | 没有文件变化，输入保留。 |
| Prepare 后、Source 未写入 | 动作保持 prepared；同 action_id 可查询/继续，不能启动 AI。 |
| Source 写入后、Commit 响应丢失 | 重读结果 hash 与计划一致则完成原动作，不再次追加。 |
| Source 在此期间又发生变化 | 报冲突；保存用户已有内容，不猜测完成状态，不再次自动应用 patch。 |
| Annotation 已保存、Provider 失败 | 输入已消费，展示“指令已保存，编译失败”；可显式重试原动作。 |
| 数据库丢失 | Annotation 仍在 Source，Draft 快照仍在 Vault；不自动重放未知的 Prepare/Commit 或重新调用模型。 |

临时命令参数可以在应用数据中用于幂等恢复，但未提交的 Review Note 不自动保存，不建设独立 Review Note 历史；其长期落点只允许 Annotation、Idea 或 Integration Intent。

## 9. Obsidian Review 工作流

### 9.1 连接与当前文档

插件位于 `engramweave-obsidian` 仓库；使用 TypeScript 和 Obsidian 原生 UI/ItemView，不引入 React 编辑器或通用聊天框架。共享 `@engramweave/contracts` 的同版构建包；本地通过明确的包构建/安装步骤共享，不复制 Schema，也不使用跨仓库源码相对路径。

插件配置 loopback 地址与 Core 本地 token，启动后校验 Core API 能力及所连 Vault 与当前 Vault 一致。只调用所需 P2/P1 路由；推理凭据留在 Core。采用 Obsidian 原生 `requestUrl`，其 API 定义及目标版本行为需要连接冒烟验证，不通过开放浏览器 CORS 解决接入问题。[Obsidian API 类型定义](https://github.com/obsidianmd/obsidian-api)

### 9.2 侧边栏与保存一致性

显示当前 Draft 的 Source Annotation、AI Notes、Relation Suggestions、当前 revision/建议新鲜度及必要错误。来源通过链接打开，不重新显示完整 Raw Source。没有 Integration Suggestions 功能入口，没有逐项“接受关系”。

普通编辑完全使用原生编辑器。插件在当前文件变化时刷新状态/显示过时，不自动调用 AI。输入和 UI 绑定 draft_id/revision_id；切换 Note 不能把旧输入提交到新 Draft。同一次动作提交中锁定三种按钮，避免一段文字同时流向多个目标。

执行动作前，插件检查当前编辑文本与最新磁盘读取一致，再提交对应的 byte hash；Core 再次校验。保存未完成、缓冲区不一致或提交期间文件变动返回可重试冲突，输入仍留在原 Draft 上。Review Complete 固定的是点击时保存的快照，之后的新编辑不改变既有确认。

### 9.3 三个 Review Note 动作

| 动作 | 长期落点 | 成功与输入清空时机 |
|---|---|---|
| 新建灵感 | `10_Ideas/<idea_id>.md`，正文为用户输入，来源和当前 Draft 以回链引用 | Idea 完整发布后清空；输入全空白拒绝。不调用 AI、不追加 Annotation。 |
| Recompile | 非空输入追加 Source Annotation；产生新 revision | Annotation 持久保存且原动作已接受后消费；AI 失败不撤销用户输入，也不再追加。空输入合法。 |
| Review Complete | Reviewed Snapshot + `review.json` 中的 Integration Intent | 产物及完成描述文件发布后清空；无输入时 Intent 为空字符串。 |

网络超时不等于动作未执行：客户端保留 action_id 并先查询结果。只清理本次提交的输入版本，不能清空用户后来新输入的内容。Core 的服务响应明确区分 `input_consumed/action_status/job_status`，不把 Job succeeded 当成所有动作的消费依据。

Relation Analyzer 在首次生成和每次新 revision 后执行。pending/running 时先完成该尝试再开放 Review Complete；没有候选时返回 empty；模型失败时显示 failed，并允许明确重试。失败不伪造建议，也不销毁可审阅 Draft；用户仍可完成正文审阅，交接中写明关系状态。用户改正文后建议标 stale，P2 不自动花费模型调用，P3 必须以最终正文再判断其适用性。恢复后只有文件而没有建议控制数据时显示 unavailable，允许用户在明确看到该状态后审阅；这不能替代首次正常链路的关系分析验收。

## 10. Reviewed Draft 与 Integration Intent 的终点契约

Review Complete 只确认当前 Draft 正文。在提交前重读工作文件、Source 引用和 hash，校验 `type: draft`、有效 sources 与当前工作流一致；用户如果破坏必要来源属性，报告具体问题，不静默修复后代替其确认。

先发布 `reviews/<review_id>/draft.md`，再发布 `review.json`。后者是最终完成标记，包含：

| 字段 | 语义 |
|---|---|
| `schema_version/review_id/draft_id/revision_id` | 交接版本和资产身份，不依赖数据库内部 document ID。 |
| `action_id/request_hash` | 找回同一次确认并核对参数，数据库恢复后也不能将不同 Intent 当成同一动作。 |
| `reviewed_snapshot_path/reviewed_content_hash/reviewed_at` | 用户当次认可的确切正文及时间。 |
| `sources[]` | `path/compiled_revision/current_revision/source_changed/original_locator`；分别保留编译输入和确认时实际读取的版本，不宣称 Source 本身已被用户认可。Source 缺失或无法读取时阻止新确认，已完成的历史确认不因此被删除。 |
| `integration_intent` | 该次 Review Note 被动作消费后的语义落点；不进入 Draft 正文。 |
| `relation_basis` | 建议对应的 revision/hash、ready/empty/failed/stale/unavailable 状态及可选集合 ID；临时建议丢失时不得伪称存在。 |

数据库状态为 awaiting_integration，UI 显示“正文已审阅，等待后续整合”。不把这句话写进知识正文，不写 archived，不调用未来 Planner。Review 之后继续编辑、Recompile 或 rollback 会形成未审阅内容；旧 Review 仍只证明旧快照被认可，不能传递到新内容。

幂等提交使用 action_id；同一动作的响应丢失后，凭已完成描述文件可找回产物，不能重复创建或丢失 Intent。Review 完成文件是用户确认及意图的资产；AI Notes/关系建议可重建，不能把它们混入交接正文来求持久化。

## 11. Core Schema / API 的冻结范围

### 11.1 公共 Schema

复用现有 snake_case、路径规范、SHA-256 RevisionSchema、统一错误结构和分页。需要在跨组件实现前冻结：

| 对象 | 最小字段 |
|---|---|
| `InferenceSettings` | 第 6.1 节配置，不包含可读回的 key；能力测试结果是状态，不是默认可用承诺。 |
| `CompileRequest` | `request_id/source_path/expected_source_revision/intent/context_paths?`；intent 为 initial 或 explicit_recompile，后者允许已归档 Source。 |
| `DraftSummary` | `draft_id/active_revision_id/working_path/source_paths/status/content_hash/relation_status/latest_review_id/source_changed`；status 为 needs_review/awaiting_integration/artifact_error，与 Job 进度分开。只有 active revision 与当前 hash 同时匹配已完成 Review 才是 awaiting_integration。 |
| `DraftRevision/DraftSnapshot` | 第 7.1 节文件描述字段；当前 content_hash 与 baseline hash 分开。 |
| `DraftAction` | `action_id/type/revision_id/expected_content_hash`，按 type 附 instruction、idea_text、integration_intent 或 snapshot_id；snapshot/reanalyze 无 Review Note 字段，拒绝其他分支字段。 |
| `ActionResult` | `action_id/status/input_consumed/job_id/artifact_paths/conflict`；prepared/committed/completed/failed/interrupted/cancelled 状态明确，Job 进度另行查询。 |
| `ReviewHandoff` | 第 10 节持久交接字段；无 canonical_path、ChangeSet 或自动归档参数。 |

`request_id/action_id` 由客户端生成，重复参数采用服务端规范化请求 hash 判断。新动作可产生新请求；重试只能继续同一输入。进行中的同一 Draft 动作互斥，另一个动作返回 `DRAFT_BUSY`，不悄悄创建第二份并发处理。

资产和动作 ID 使用 UUID，时间使用 UTC RFC 3339，路径统一为 Vault 相对路径，hash 使用 P1 SHA-256。Review Note 三种文本字段均限定最多 8,000 字符；Idea 拒绝全空白，Instruction/Intent 允许空字符串。Draft 的 sources 首轮必须恰好一个有效 Source Record；结构保留数组形式，但不实现多 Source 合并编译。未知字段和不匹配的动作分支字段拒绝，不宽松吞掉输入。

### 11.2 最小 API 增量

保留九个 P1 路由及原行为；新增路由仍在 `/v1`，更新 Core 与共享包版本。插件根据既有 health 的 core_version 检查受支持的 P2 版本，再访问必要路由；不新增通用能力协商平台。`/v1/status.active_job` 继续只表示扫描任务；P2 客户端通过扩展的 Job 列表查询推理任务。

| 路由 | 用途 |
|---|---|
| `GET/PUT /v1/inference/settings` | 读取脱敏配置/设置单一 Provider，不保存客户端提交的明文 key。 |
| `POST /v1/inference/check` | 明确发起最小连接验证；返回认证、协议及模型可用性，不读取 Vault 资料。 |
| `GET /v1/compile-candidates` | 从 P1 ready Source 枚举后重读，筛选未归档且类型受支持的候选，标明已有 Draft/进行中任务，不自动处理。 |
| `POST /v1/compilations` | 当前 Source 的初编译或显式重编译；202 返回 Job 与 action_id（等于 request_id），归档默认拒绝，重复请求收敛。已有 Draft 的重编译使用 Draft actions；本路由只创建新的 Draft 工作流。 |
| `GET /v1/drafts` | 当前 Draft 与状态列表。 |
| `POST /v1/drafts/recover` | 显式从受限 Draft 目录重建文件资产投影；只读 Vault 文件，不编译、不写归档属性。 |
| `GET /v1/drafts/{id}` | 当前 revision、来源、AI Notes、关系状态及最近 Review；读取文件验证新鲜度。 |
| `GET /v1/drafts/{id}/revisions` | 版本和可恢复快照列表。 |
| `GET /v1/drafts/{id}/diff` | 两个受限快照 ID 的文本差异；不接受任意磁盘路径。 |
| `POST /v1/drafts/{id}/actions` | idea/recompile/review_complete/rollback/reanalyze/snapshot 的判别联合请求；只有前三者消费 Review Note，snapshot 只保存当前已确认 hash 对应的比较快照。 |
| `GET /v1/draft-actions/{action_id}` | 查询动作是否消费输入、已落盘产物和 Job，解决响应丢失。 |
| `POST /v1/draft-actions/{action_id}/commit` | 仅提交准备好的 Annotation 编辑回执；Core 验证后启动原 Recompile。 |
| `POST /v1/draft-actions/{action_id}/retry` | 只重试已保存输入的失败/中断动作；不重做 Annotation 追加；输入已变化则冲突。 |
| `POST /v1/draft-actions/{action_id}/cancel` | 只放弃尚未提交给推理执行器的 prepared 动作；不实现远端请求取消或撤销 Annotation。 |
| `GET /v1/reviews/{review_id}` | 读取并验证 Reviewed 快照及 Intent，作为 P3 的读取边界。 |

初编译也在同一 `draft_actions` 表登记为 compile 动作，复用 action 查询与 retry，不另建请求去重系统。编译请求的 intent 只控制是否明确处理已归档 Source，不代表可以覆盖已有 Draft。同一 Source 存在进行中的编译时拒绝第二个不同 ID 的请求；已有有效 Draft 时返回其引用，由用户在该 Draft 内执行 Recompile。

prepared 动作可由同一客户端继续或显式放弃，避免长期占用 Draft。放弃时重读 Source：仍是基线则 input_consumed=false；已匹配追加结果则 input_consumed=true、保留 Annotation，但不启动 AI；无法证明哪一种时返回冲突并保留输入，提供终止该动作的冲突结果，不猜测追加或撤销。已经提交给执行器的动作不接受此操作。遇到无法判断远端是否已调用的中断，仅显式 retry，不自动续跑。

`GET /v1/jobs` 增加可选 `kind` 过滤，默认仍只返回 scan_vault；P2 客户端明确请求 `kind=all` 或具体推理类型，使用判别联合 DTO。已有扫描 Job 的详情、扫描响应和列表字段保持原样；新增 Job 详情按其 kind 返回，不能把 mode 填成虚构的扫描模式。

新增错误至少包括 `INFERENCE_NOT_CONFIGURED/INFERENCE_FAILED/INVALID_MODEL_OUTPUT/UNSUPPORTED_COMPILE_INPUT/SOURCE_ARCHIVED/REVISION_CONFLICT/DRAFT_BUSY/IDEMPOTENCY_CONFLICT/ARTIFACT_INCOMPLETE`。HTTP 对输入用 400/422、状态冲突用 409、未知资源用 404、外部服务失败用 502/504；错误不包含秘密和完整来源正文。

P1 `/v1/documents` 的 20/40 目录边界不变；Draft 从专属 API 读取。P1 `/v1/search` 默认仍不索引 Draft、快照与 AI Notes，不把候选当作已认可知识。现有 Capture 也不能写 10_Ideas 或 30_Drafts，新增动作走各自受限路径。

### 11.3 SQLite 与 Job 的最小演进

在现有库中迁移到 v2，保留 documents/meta 和 P1 数据，扩展 jobs，增加 `drafts`、`draft_actions`、`review_metadata` 三张工作流投影表；文件 revision/Review 清单仍是长期资产，不再为它们建立独占正文表。

- `drafts`：逻辑 Draft、当前选中 revision、工作流状态与最近 Review 引用，可从文件重新登记。
- `draft_actions`：幂等键、请求 hash、前置条件、执行阶段和结果路径；临时动作参数用于重试，不能当成永久 Review Note 存储。
- `review_metadata`：特定 revision/hash 的 AI Notes、关系集合及 prompt/model/source/context 运行说明；允许按保留策略重建或丢失，不删除用户资产。
- jobs 采用 scan_vault/compile_source/recompile_draft/analyze_relations 判别类型；scan 的 mode/summary 保持现状，非扫描任务不能伪造 refresh/rebuild。GET jobs 按上述 kind 约定返回同一分页包络。
- 扫描执行器所有 active/restart/cleanup 查询加 kind 范围。新增一个串行推理执行器，一次最多一个远端调用；扫描可以继续运行，不能被编译 Job 冒认为正在扫描。仅复用必要的 Job 读写/状态函数。
- 旧扫描保留策略不删除未完成动作或用户快照。已有迁移/恢复工具同步识别 v2；迁移失败保持旧库可用，不自动清空再建。
- 重启标记未完成的远端任务 interrupted，不自动重付费。关系失败与编译成功分别呈现，不能回滚已发布的有效 Draft。

compile/recompile Job 内按顺序执行 Compiler、发布 Draft、Relation Analyzer；最后一步是同一 Job 的步骤，不在串行队列中嵌套等待另一个 Job。只有独立重新分析或 rollback 后的分析创建 analyze_relations Job。Draft 已完整发布而关系失败时编译 Job 可 succeeded，但结果必须携带 relation_status=failed 和警告，侧边栏不能显示“关系已就绪”。排队后实际开始执行时重新检查 Source 版本和归档属性，不能只在 HTTP 接收时检查。

相同请求的重复提交不重复启动远端调用；如果进程在远端已执行但尚未持久保存结果时中断，本地无法保证服务方不重复计费。标记 interrupted，用户显式 retry 才重新调用，并说明可能产生新的调用费用；不宣称跨服务 exactly-once。已发布的 revision 则从完成描述文件找回，不能因 Job 状态未更新再次运行 Compiler。

## 12. 失败、并发与恢复

| 风险 | 必须具备的行为 |
|---|---|
| 模型响应慢/失败/非 JSON/截断 | 明确失败，无半成品工作文件；可显式重试，不自动切服务或重复计费。 |
| Source 在提交后变化 | 输入快照记录实际版本；发布前发现变化则保留候选但标 source_changed，不能冒充针对当前 Source 的结果；重新提交 Review 时明确当前/编译时版本。 |
| 用户在模型运行时继续改旧 Draft | 旧文件不被替换；新版本另存，用户修改可继续读取。 |
| 重复点击/HTTP 响应丢失 | 根据 action/request ID 查询或继续原操作，不重复写 Idea、Review 或 Annotation。 |
| 部分 revision/Review 文件已写，完成描述未写 | 标 incomplete，保留文件，恢复原操作；不把 incomplete 产物当 Reviewed 或删掉用户编辑。 |
| 当前关系建议已过时 | 明确 stale；允许显式重新分析，不自动确认，也不把它当最终整合输入。 |
| SQLite 损坏/丢失 | 重建 P1 Source/Knowledge 投影，并显式重新登记 Draft/revision/Review 文件；恢复可读内容与 Intent，不承诺复原全部 Job/AI Notes。 |
| 用户手工移动/修改历史文件 | 缺失/篡改引用报错；不猜测匹配、不重写 Source 或 Canonical。 |

Source Changed 的 Review 规则：保留编译时输入版本，显示当前 Source 已变化；用户对当前 Draft 的正文确认仍可以成立，但交接必须携带版本差异标志，不能声称它覆盖了最新 Source。归档标记始终只由未来 P3 的成功整合维护。

数据库恢复后不能依赖旧内部 ID 定位 Draft，不能因为找不到旧 Job 就重新编译。已有完整 Review 文件经 hash 校验可重新发现；新编辑的当前 Draft 不会自动变成 reviewed。所有恢复操作与测试在隔离副本中执行。

恢复入口是 `POST /v1/drafts/recover`：没有进行中的 Draft 动作时执行一次有界本地恢复，复用 P1 的文件数/总读取字节预算，验证完成后事务性更新投影，返回登记数量和缺失/冲突诊断。超限明确失败，保留原投影，不创建后台调度系统或隐式续跑。它不扩大 `/v1/scans` 的 Source/Knowledge 根目录；Source/Knowledge 仍用原 rebuild 恢复。恢复后的未知动作不自动重放。

## 13. 测试与 P2-G 验收

### 13.1 复用测试栈

继续使用现有 Vitest、真实 SQLite/临时 Vault/独立 Core 进程 helpers；按 subsystem 拆分测试，不创建新的测试框架、临时验证应用或大型 E2E 平台。纯协议与失败测试使用本机可控的最小 HTTP stub；这不替代真实 Provider 验收。

Core 在当前仓库测试，插件纯逻辑可在其仓库复用 Vitest；用户可见编辑、保存、版本打开和原生请求以少量手工冒烟为主。源码注释、代码块和测试标识用英文；维护性文档中文。实际报告、模型原始输出、现场检查记录放 `.local/p2/`，`docs/` 只保留长期行为与复现方法。

### 13.2 重点用例与分层

| 层级 | 应保护的行为 |
|---|---|
| 契约/纯函数 | Source/Annotation 分区、预算、模型输出校验、关系目标/证据校验、字段分支、路径和输入消费状态。 |
| 文件/SQLite 集成 | revision 只新建、动作前快照、Annotation 无损编辑计划、Review 完成标记、幂等、v1→v2 迁移、Job 类型隔离和恢复。 |
| 有限 HTTP 集成 | Provider 超时/失败/错误输出、Source 过期、相同 ID 不同参数、commit 丢响应、重试不追加、旧 P1 API 不退化。 |
| 薄 E2E | 一条真实剪藏到 Reviewed 的路径；一条用户修改→Recompile→diff→rollback；一条数据库恢复保护内容与 Intent。 |
| Obsidian 手工 | 接错 Vault、未保存缓冲区、切换 Note、重复点击、断线、Source 同时打开、CRLF 原文保持、侧边栏 stale 状态与原生编辑共存。 |
| 真实模型质量 | 来源忠实度、关键条件保留、Annotation 不当成事实、无依据结论、不将 AI Notes 混入正文、关系建议有明确证据。 |

真实样本优先复用 P1 的两份 Web Clipper 文件，在隔离副本中增加非空 Annotation、预置 archived、相关/不相关/冲突 Knowledge、长输入和注入式资料片段。资料片段可能不完整，Compiler 必须承认缺失，不能联网补全或编造文章未保存部分。

真实调用验收需使用用户配置的服务和测试资料；测试不在普通 `npm test` 中自动向远端发送数据。一般回归用本地 stub；现场真实调用记录服务/模型、prompt/schema 版本、耗时与有界用量，不在日志公开密钥或私有全文。

### 13.3 P2-G 对照表

| 上位标准 | 验收条件 |
|---|---|
| G1 可直接阅读的 Draft + 来源/条件 | 真实模型为代表性 Source 生成可编辑正文；含正确来源，关键限制不被省略，Annotation 与原文事实分开；AI Notes 只在控制信息中。 |
| G2 Review 前关系建议与 revision 对应 | 首次/重新生成/恢复新 revision 均分析或明确失败；有效建议基于明确证据，新旧正文不混用；纯相似或无候选允许 empty，不写关系。 |
| G3 Review Note 三种动作 | Idea 带双回链；Recompile 指令只追加一次；Review Complete 保存 Intent；成功消费后清空，失败/超时不误清其他输入。 |
| G4 用户编辑与恢复 | 已编辑 Draft 在 Recompile/rollback/中断后可恢复，Source 正文与既有 Annotation 不丢；数据库丢失后所有用户文件和 Intent 仍可读。 |
| G5 归档边界与 P2 终点 | 默认排除 archived、显式重新编译可用且不改归档值；最终只有 Reviewed Draft + Intent，无 Planner/ChangeSet/Canonical 写入。 |

附加通过条件：旧 P1 聚焦回归通过；迁移/任务边界验证通过；真实 Provider 与 Obsidian 保存/Annotation 路径得到现场验证；没有未解决的资产丢失、错误正文确认或重复动作问题。模型质量采用代表性样本人工核验，不以 JSON 校验通过代替知识正确性，也不承诺模型永不出错。

## 14. 实现前必须决定的问题

| 编号 | 问题 | 本计划的决定或默认值 | 实现前需要的证据 |
|---|---|---|---|
| D01 | 第一条推理链路 | 已选择 OpenAI-compatible，自定义 base_url/model；仅一个协议适配 | 具体地址、模型、凭据环境变量及 token 参数支持，在本机配置中落实，不在计划里记录秘密 |
| D02 | 初版支持的 Source | web/manual inline Markdown；不处理外部二进制 | 真实 P1 样本和拒绝路径能复现 |
| D03 | 版本物理布局 | 新 revision 新文件、不可变快照、普通 JSON 描述；不覆盖旧工作文件 | Obsidian 打开/切换体验与恢复路径验证 |
| D04 | Annotation 的并发写入 | Core prepare → 客户端原生受限 edit → Core commit；不引入通用 Source 编辑 API | `Vault.process`、缓冲区检查、BOM/CRLF/其他字段保留在目标 Obsidian 版本实测；不成立时先报告该实现阻塞，不静默改成无条件覆盖 |
| D05 | Review 完成落点 | snapshot + review.json，Intent 独立于正文，发布完成描述文件后才成功 | 重复提交、丢响应与部分写入恢复验证 |
| D06 | Relation 覆盖与失败 | 只对最多 5 个已存在 Knowledge 候选分析；失败可继续正文审核但显式标注 | 正/负/空/过时建议样本及人工检查 |
| D07 | 配置与指令 | 本机 inference.json + 环境变量 key；两类独立 Prompt，可从 90_System 覆盖 | Headless 与 Desktop Host 环境变量传递及配置不可泄密 |
| D08 | Job 和 DB 演进 | v1→v2，一种串行推理执行器，旧扫描语义保持 | 严格 DDL 校验、active/recovery/retention 不跨类型误处理 |
| D09 | 插件分发/契约共享 | 独立 Obsidian 仓库，同版 contracts 构建包；桌面版 Obsidian | 锁定 Obsidian API/minAppVersion 和包构建安装方式，不依赖私有接口 |
| D10 | 输入/输出预算与模型质量 | 使用第 6 节有界预算，真实样本人工质量门槛 | 目标模型上下文限制与真实输出，超限不能悄悄截断 Source |

这些决定控制实现范围，不要求重新讨论总体架构。遇到与当前代码或外部工具的具体冲突，先记录冲突和最小修正，更新对应契约后再实现；不以“方便 P3”为理由扩充 P2。

## 15. 阶段交付物

P2 交付 Core 的编译/关系分析/版本/审阅接口、最小 Obsidian 插件、Desktop 必需配置与状态、稳定文件与接口说明、可复现验收方法，以及满足上述门槛的 Reviewed Draft + Integration Intent。详细实施项和依赖见 [P2_TODO.md](P2_TODO.md)。P3 接手这些产物后再独立实现 Integration Planner 与 ChangeSet。
