# EngramWeave 项目实施规划

> 规划基线：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[CONTEXT.md](CONTEXT.md) 与已记录的领域决策。
> 更新日期：2026-10-12。
> 执行清单：[TODO.md](TODO.md)；当前 P3 范围：[P3_IMPLEMENTATION_PLAN.md](P3_IMPLEMENTATION_PLAN.md)、[P3_TODO.md](P3_TODO.md)；P2 历史范围：[P2_IMPLEMENTATION_PLAN.md](P2_IMPLEMENTATION_PLAN.md)、[P2_TODO.md](P2_TODO.md)。

## 1. 目标、当前基础与范围

P1、P2 开发已完成，作为现有实现基线，不重做已交付能力。P2 已实现状态、编译／分析、语义召回、调度、论文投递和人工审阅；P3-A 已实现 Agent Planner、Core Tools／MCP 与 Vault 候选编辑，接下来完成完整影响审查、多文件批准执行及自动 Git。模型 API 扩展暂缓，保留 P2 已有能力。MVP 已获批准的人工新建入口属于明确例外，不等同于完整 P3。

P1 历史范围保留在 [P1_IMPLEMENTATION_PLAN.md](P1_IMPLEMENTATION_PLAN.md) 和 [P1_TODO.md](P1_TODO.md)，现有受支持行为见 [P1 合同](../engramweave/docs/p1-contracts.md)。后续工作以本计划的 P3 起点为准，不将历史清单的未回填项视为待重做任务。

首个实用场景是 Zotero 阅读中明确选择的段落、高亮与批注，经 Source → Knowledge Compiler → Human Review → Integration Planner → ChangeSet 批准进入 Knowledge 或 Research，并能再次找回及返回原论文位置。已有网页／手动材料共用这条流水线。首个实用版本在 P3 完成，P2 是中间交付检查点。

开发投入为用户与 coding agent，期望一周左右开始使用，时间可让步。此目标不作为已估算的工期承诺，也不以裁掉知识重组、语义召回或 Codex 路径来满足日历目标。

### 1.1 不变的架构边界

- 总体设计和系统 CONTEXT 是架构及领域语义基线；阶段计划不能擅自新增产品语义。系统文档放 engramweave-docs/，组件上下文与实现决策放各自仓库。
- Core 独立运行；Desktop 是控制中心及 Core Host；Obsidian 负责原生编辑、导航和工作流侧边栏；Zotero 负责论文、PDF 和书目。
- 用户知识与来源保持开放文件或外部引用。数据库保存运行状态、可重建投影及派生数据，不独占正式知识。
- AI／Agent 只生成内容候选、分析和提案；正式修改由用户批准后交给确定性 Executor。标记废弃不同时授权物理删除。
- 实现选择由 coding agent 根据当前代码、成熟工具和验收合同决定。模块、数据库布局、协议细节、幂等和异常恢复不另设产品确认点；发现具体架构冲突再报告。

### 1.2 首个实用版本的能力合同

| 能力 | 必须满足的行为 |
|---|---|
| Zotero Capture | 一次明确选择形成独立 Source，保留论文及各自位置；不自动投递其他高亮或全文，不同步后续 Zotero 编辑。 |
| Compiler | 只整理投递内容与 Annotation，返回标题和正文。可合入用户理解，无须区分来源事实／用户判断；不扩张整理范围。 |
| Draft Analyzer | Core 编排 Review Analyzer → Relation Analyzer，绑定本轮 Source／Draft 输入；分别保存侧边栏结果，不能修改正文。 |
| 配置和执行 | Desktop 预配置模板、Analysis Profile 与各任务模型；Capture 只选择预设。P3 优先 Agent 与 Core Tools／MCP，Planner 首先支持 Agent；P2 已有模型 API 保留，新增 API 能力暂缓。 |
| 关系召回 | 基本语义召回覆盖 Knowledge、Ideas、Research，支持不同表达下的联系；覆盖率可以有限，不能只用关键词替代。 |
| 人工审阅 | 原生编辑、统一 Review Note 三动作；Review Complete 允许规划，不冻结正文，不自动因编辑撤销。 |
| 统一整合 | Planner 自行提出跨文件新建、修改、重组、融合、关系和废弃等结果，统一形成候选 ChangeSet；不分成四种业务模式。 |
| 候选审查 | 候选位于 `70_ChangeSets`，在 Obsidian 内以 Current ↔ Candidate 双栏审查、编辑、保存及打回。未批准过程不修改正式文件；批准后立即应用最终内容。 |
| Git 和清理 | 成功应用后只提交本次涉及受跟踪文件，包括其中此前未提交用户编辑；入库只自动 discarded 相关 Draft，其他内容不自动废弃。 |
| 恢复和再次找回 | 保护用户文件及修改，正式执行中断可恢复且不重复应用；基本检索可返回正式内容和来源引用。 |

## 2. Roadmap 与里程碑

| 阶段 | 范围 | 完成标志 |
|---|---|---|
| P0 设计和语义基线 | 总体设计、Context、Glossary、关键 ADR 与阶段合同的维护 | 领域含义和组件边界一致；随实际决定维护。 |
| P1 本地资产基础 | 已完成的独立 Core、登记、Capture、读取查找、Desktop 与恢复基础 | 保留现有完成结论；后续变更做相关回归。 |
| P2 论文投递、编译与人工审阅 | 状态扩展、定时／手动轮次、API／Codex、基本语义召回、两项 Analyzer、Zotero 插件、Obsidian Review | 真实论文选段能形成可编辑 Draft 并允许进入 Planner；没有正式整合写入。 |
| P3 统一整合与首个实用闭环 | Agent Planner 与 Core MCP Tools、Vault 候选和 Obsidian MergeView 审查、Executor、Git、归档、清理及基本找回 | 论文／网页材料能融入已有 Knowledge 或 Research，并找回及溯源。 |
| P4 检索与科研增强 | 提升召回、关系检索和认知重载；研究问题、证据和冲突材料的使用体验 | 在真实研究材料上评估遗漏、证据及联系；不建立独立 Research 写入流程。 |
| P5 长期知识维护 | 重复、冲突、过时、结构问题等分析及提案 | 建议有依据，用户查看，正式变更复用 ChangeSet。 |
| P6 复杂 Agent 与外部工具扩展 | 复杂研究、跨笔记维护、长任务、扩展 MCP 与 Agent 管理 | 复用已有四任务执行入口，扩展能力保持读／提案权限边界。 |
| P7 多端和多模态扩展 | 有具体需要的 Mobile、Capture、派生处理及可选 Hybrid | 各入口仍按投递单位进入统一流水线，本地正式知识归属不变。 |
| P8 稳定化与发布 | 支持范围、安装更新、兼容、恢复和发行质量 | 已交付范围可持续使用；质量验证贯穿此前各阶段。 |

主线为 P1 → P2 → P3。Zotero 与 Obsidian 客户端可在相应 Core 合同可用后并行推进；不以客户端仓库名称为理由重做已有成熟工具。后续阶段顺序可按真实使用调整，不将高级能力作为首个闭环的前置条件。

M1 是已完成的 P1 基础；M2 是 P3 完成后的首个实用版本；M3 是 P4 的科研及检索增强；M4 是维护与复杂 Agent；M5 是扩展采集；M6 是稳定发行。

## 3. P2：论文投递、编译与人工审阅

### 3.1 实施顺序

1. 协调扩展 Source 解析、登记、属性补充、投影和客户端合同，不重开 P1。确认五个 processing 阶段及独立 lifecycle／registration／job 状态。
2. 建立最小真实 API 与 Codex 执行适配、按任务模型配置、分析模板和 Profile 配置。先打通一条真实编译切片，再扩展完整首期矩阵。
3. 为 Knowledge、Ideas、Research 提供基本语义召回和引用上下文，供 Analyzer 及后续 Planner 使用。
4. 打通 Compiler → Review Analyzer → Relation Analyzer；按配置定时／手动轮次执行，提供失败可见性和单独重分析。
5. 在 Source 投递与预设选择合同稳定后接入 Zotero 选定材料投递，同时实现 Obsidian 侧边栏、原生审阅和 Review Note 动作。
6. 验证用户编辑后 Recompile 的独立 Draft 保留，以及数据库损坏／操作中断后的资产恢复；不要求 Draft 之间的 diff 或独立 rollback 界面。完成真实论文／网页材料的审阅检查点。

### 3.2 P2-G 验收

- 论文投递选定材料形成独立 Source，链接回正确原位置；网页／手动输入同样可编译。全文只可作上下文，不自动扩张知识范围。
- Compiler 仅生成标题／正文；Review／Relation 两项独立分析、可配置、只进侧边栏。真实 API 与 Codex 均可用于这三类任务。
- 基本语义召回覆盖三类正式上下文，并有表达不同的代表性材料验证；未召回不能宣称不存在联系。
- 每轮读取当前阶段；pending 编译，reviewed 供后续规划；仅处理有效、active、无进行中任务的材料。无启动补跑。
- 辅助分析失败保留 compiled 并可继续审阅或单独重分析；有限本轮重试，失败不覆盖 Source 阶段。
- Review Note 正确分流：Idea 用户直接创建；Recompile 追加 Annotation 并只返回 pending；Review Complete 保存 Intent 并返回 reviewed，不绑定精确正文版本。
- 修改、Recompile、数据库重建和模型失败不丢 Source、Annotation 或用户 Draft 编辑。P2 正常审阅流程不执行 Planner／ChangeSet；ADR-0015 授权的 MVP 可人工新建 Knowledge 并归档，不能覆盖已有内容或写 Research。

## 4. P3：统一整合与首个实用闭环

### 4.1 实施范围

- Planner 读本轮启动时的当前 Reviewed Draft、关系分析和 Intent，通过 Core Tools／MCP 获取当前知识库及分类上下文，记录实际读取版本；优先 Agent 路径，复用既有运行器 loop、Relation 模板与召回，重新验证关系。Compiler／Analyzer 的独立语义不变。
- 第一版支持完整知识重组结果，使用一套候选 ChangeSet 和审查机制，不以 Create／Update／Split／Merge 独立模式缩减能力。
- ChangeSet 放在 Vault 根目录 `70_ChangeSets/<id>`，包含 manifest 与候选目标文件，不建立 Base 目录或自动三方合并。Agent 只通过受限工具写本轮候选，不获得正式写入或批准／执行能力。
- Obsidian 使用 CodeMirror 6 MergeView：左侧当前文件只读，右侧候选可编辑；保存只写候选，刷新左侧保留右侧。用户编辑最终版本、拒绝或打回；未批准过程不修改正式内容。Desktop 不新增第二套可编辑 Diff。
- 批准后 Executor 立即应用最终内容并检查目标前置条件；Job、归档标记和 Git 提交可恢复，不能因中断再次套用同一变更。
- Knowledge／Research 按用途归档；已有课题文件夹优先，必要新建文件夹随方案审查；新一级领域需明确批准。
- 实现 Source／Draft／正式文件的完整 discarded 筛选、恢复和批量清理合同。Sources 的 inline Record 显式物理清理已前移至 P2 B 修订；独立 Asset 归属、Derived Representation 与完整跨类型清理仍在相应能力实现时交付。AI 提案走 ChangeSet；用户操作按明确清单；物理删除在标记后另行确认。

### 4.2 当前施工顺序与快捷入口

复用 P2 已有能力，先验证 Obsidian MergeView 审查体验，再补齐 Core 候选工作区及任务工具，推进真实 Agent Planner → 人工审查 → 批准执行 → Git 与真实闭环，详细任务见 [P3 计划](P3_IMPLEMENTATION_PLAN.md) 和 [P3 TODO](P3_TODO.md)。

保留 Publish to Knowledge 人工快捷新建，不强制调用 Planner 或增加批准步骤；P3 将其接入与 ChangeSet 相同的 Executor、归档恢复及自动 Git。当前 MVP 尚无自动 Git，升级必须保留已有未完成审批回执的恢复。见 [ADR-0018](docs/adr/0018-shared-execution-for-direct-publication.md)。

新增模型 API 路径、API 效果优化与 Core 自建工具循环暂缓；这不删除 P2 API，也不暂停客户端本地 HTTP API 或已有 Embedding／Reranker。模型 API 后续根据真实需要恢复，不绑定到一个虚定阶段。范围与候选审查决定见 [ADR-0019](docs/adr/0019-agent-first-delivery-and-core-tools.md)、[ADR-0020](docs/adr/0020-vault-local-changeset-review.md)。

### 4.3 P3-G 验收

1. 从 Zotero 选段到正式 Knowledge／Research 再次找回及回到原位置，全链路真实可用；网页材料共用同一流程。
2. Agent Planner 实际使用读取、检索及候选写入工具，统一生成多文件重组、关系、来源和废弃方案。Core 拒绝越权；不要求新增 Planner 模型 API 路径。
3. Planner 期间 Draft／本地 Note 改动不自动撤销候选；Obsidian 对比 Current 与 Candidate，候选保存及 Refresh Current 不写正式文件或丢右侧编辑。取消 Review Complete 返回 compiled，新候选重新批准。
4. 正式应用满足批准内容与目标前置条件；失败及异常恢复不会静默丢用户编辑或重复应用。
5. Source 在整合成功并保存归档标记后进入 archived；相关 Draft 自动 discarded；Source、Annotation、正式内容不自动 discarded。快捷新建共用批准执行与恢复，不额外调用 Planner。
6. ChangeSet 与快捷新建成功应用后均自动提交 Git；仅跟踪约定目录与配置，提交包含涉及文件先前未提交编辑，不带入无关文件，不跟踪独立 Asset／Derived／Draft／Inbox／ChangeSet；`70_ChangeSets` 同时排除正式登记与召回。
7. 默认清理跳过被 active 正式内容引用的 Source；用户可经引用提示覆盖。专属受管理 Asset／Derived 随 Record 删除，外部 Zotero 文件不动。
8. 数据库重建读取文件阶段和生命周期；已取消／丢失未批准候选在 Draft 可用时返回 reviewed。正式执行和 Git 失败有明确恢复结果。

## 5. 后续阶段边界

P4 提升已经存在的语义召回和科研材料体验，而不是首次加入论文入口或首次允许 Research 入库。Research Question 继续是普通 Markdown 笔记和链接，不新增应用实体。

P5 复用统一候选审查和受控执行，不静默修改内容或主动推送维护提醒。P6 在已有 Codex 适配和必要 Core Tools／MCP 上扩展复杂能力，不重新接入首期四任务。Core 自建工具循环仅在具体需要出现时增加。

P7 依据实际采集需求选择入口和多模态 Worker，避免空包和未来框架。P8 整理已完成能力的发行，不将安全、资产保留或恢复验证推迟到发布阶段。

## 6. 文档、验证和实施责任

验收采用真实代表性材料与有意义的自动测试，保护状态、内容持久化、候选批准边界、执行恢复与 Git 范围；模型输出格式通过不等于内容质量通过。普通测试不自动调用付费模型，真实调用在明确验收环境中进行。

维护性文档说明长期合同和复现方法；现场样本、模型原始输出、临时诊断与阶段报告放各仓库 gitignored 的 .local/<phase>/。coding agent 按当前能力选择最小实现并维护组件 Context／ADR；不要把本计划扩写成无当前用途的基础设施。
