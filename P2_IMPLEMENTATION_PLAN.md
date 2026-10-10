# P2：论文投递、编译与人工审阅实施计划

> 基线：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[CONTEXT.md](CONTEXT.md)、[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)。
> 更新日期：2026-10-09。P1 已完成；本阶段扩展现有合同，不重新验收 P1。
> 执行清单：[P2_TODO.md](P2_TODO.md)。首个实用闭环由 P2 和 P3 共同完成。

## 1. 阶段范围

P2 建立真实 Zotero 选段投递与普通网页／手动内容共用的编译审阅流程。完整 P2 的终点是用户允许进入 Integration Planner 的 Draft 与 Integration Intent；Planner、ChangeSet 审查、正式 Knowledge／Research 应用和 Git 提交在 P3 实现。提前交付的 Obsidian MVP 例外路径见 §1.1。

### 1.1 提前交付 Obsidian MVP

先串通现有 Compiler／Analyzer 与原生 Draft 编辑、侧边栏分析、人工直接入库。用户明确确认后由 Core 在 `40_Knowledge` 新建一份当前 Draft 的正式笔记；不覆盖已有文件，不执行 Planner 或 ChangeSet 审批。保留 Source 回链与 Draft 文件，完成后 Source archived、相关 Draft discarded。自动 Vault Git 本轮暂缓。完整 D、Zotero、Review Note 三动作及 Planner／ChangeSet 流程的交付状态仍独立维护，不能因 MVP 完成而宣布完整 P2／P3 完成。系统语义及限制见 Context 与 ADR-0015。

纳入 Source 阶段扩展、定时与手动轮次、Compiler、两项 Analyzer、基本语义召回、Analysis Profile、真实 API／Codex 适配、Desktop 配置和运行状态、Zotero 采集插件、Obsidian 侧边栏及 Draft 编辑保护。首期不自建 API 工具循环，不实现复杂研究／跨笔记维护平台，不要求整篇 PDF 解析或自动发现其他知识。

## 2. 领域合同与资产边界

### 2.1 Source 与 Draft

- 一次明确投递一个 Source Record，可引用同一篇论文并保存各自位置；一次操作只收所选材料，不同步投递后的 Zotero 高亮／批注。
- Markdown 正文是逻辑 Source Asset，可与 Record Properties 同文件；独立受管理 Asset 为 Record 专属，外部 Zotero Asset 仍由 Zotero 管理。
- Compiler 整理 Source 内容和 Annotation 中投递的用户理解，不标注来源事实／用户判断。全文上下文不扩大投递范围，正文不加入额外模型分析。
- 一个未 archived 的 Record 可以有多份 Draft；最终只有一份 Draft 用于正式入库，入库成功后所有相关 Draft 标记 discarded。Recompile 保留已有 Draft、用户修改及版本。P3 可能将一份 Draft 融入多份正式文件。
- Draft 与用户修改不能因数据库损坏丢失。Draft 不进 Git，重编译的 diff／恢复需有自身文件保留能力；无需绑定 Review Complete 的精确正文版本。
- Recompile 的后续轮次可以生成另一份 Draft；既有用户编辑必须保持可恢复，不增加正文替换审批点（总体设计 §8.2、§8.3）。

### 2.2 四类状态

| 状态 | 取值 | 权威位置 |
|---|---|---|
| processing_status | pending / compiled / reviewed / planned / archived | Source Record Properties，Core 可重建投影。 |
| lifecycle_status | active / discarded | Source、Draft、正式文件 Properties，Core 可重建投影。 |
| registration_status | ready / invalid / missing / unsupported | Core。 |
| job_status | queued / running / succeeded / failed / interrupted | Core。 |

Registry 缺失／空 processing_status 时补 pending，缺失／空 lifecycle_status 时补 active，保护正文、Annotation 和其他属性；历史材料同样适用。重建数据库读取完整文件属性，不自行重置阶段。错误、失败轮次、重试及重编译计数属于 Core。

P2 的阶段动作包括 pending→compiled、Review Complete 的 compiled→reviewed、规划尚未开始时取消的 reviewed→compiled，以及 Recompile 返回 pending。planned／archived 的正式推进和候选取消在 P3 完成。失败仅 Job 状态，discarded 不覆盖阶段。

## 3. 配置、模型路径与上下文

### 3.1 四类任务与执行入口

Compiler、Review Analyzer、Relation Analyzer 在 P2 均支持 API 或 Codex，并可独立配置模型；P3 的 Planner 复用适配并同样支持两路。Codex 入口使用已有运行器和用户可用认证，不将订阅路径替换为另行付费 API。

API 由 Core 按任务准备上下文并独立调用模型，不自建动态 loop。Agent 复用其运行器 loop，分析可按需读 Core Tools／MCP；只增加当期需要的工具，权限限制为任务所需的读取及侧边栏结果产出。模型不得通过文件权限绕过任务合同直接修改正式知识。

不为未来 Worker 建空包、通用调度框架或兼容层。复用现有工具后，实际调用方式、输出校验、认证存放和恢复机制由 coding agent 在组件中确定和记录。

### 3.2 Analysis Profile

Desktop 提前配置模板、模板内容、模型和路径，形成 Review／Relation 分析组合。Capture 只选用预设；pending 期间可改选择，执行按最终选择与当前配置处理，不保留采集时整套配置副本。

Review Analyzer 可针对知识理解、条件遗漏、原意或学术 Claim／统计等进行分析。Relation Analyzer 可面向知识关联或 Research 的观点、结论、冲突及整合建议。结果分别保存，只在 Obsidian 侧边栏展示。

默认各输出 0–3 项最有价值的短建议，每项 1–2 句，不重复总结 Draft；详情折叠。模型用本任务短证据 ID 引用材料，Core 补齐实际版本和片段范围，保留追溯及过期检查；分类和常规覆盖说明不要求模型填写。模板仍可个性化请求更详细内容。Compiler、Review 和 Relation 的 API 输出 token 预算独立可配置，保存不启动模型。

两项 Analyzer 绑定本轮相同 Source／Draft 输入，分别获得模板需要的上下文。Relation 三档复用：Review 全输入上下文；仅输出参考；上下文及输出均不复用。这里的 Review 指 AI Review Analyzer，不是之后的 Human Review。

是否参考本轮 Review 输出由用户在设置中按偏好选择，不是 Relation 的必需依赖。没有合法输出时直接独立执行 Relation，使用自己的输入和上下文；Review 自身失败按原机制处理，不增加提示或特殊降级状态，不引用旧轮输出替代。

### 3.3 基本语义召回

扩展 P1 的上下文读取和查找范围，覆盖 Knowledge、Ideas、Research，并具备基本语义召回。允许有界候选和有限覆盖，但不能将旧计划的 Knowledge-only、最多五份或仅关键词规则作为已确认合同。

候选召回不等于关系成立；模型根据材料提出有意义的关系、冲突与整合线索。单纯主题相似由检索呈现，不自动成为正式 Typed Relation。未召回的内容不被解释为没有联系。模型、索引、切分、检索策略及预算由实施选择，记录当期支持范围和质量限制。

Embedding 独立配置 endpoint、模型与凭据，本机优先，可使用用户明确配置的远端服务，不自动沿用 Compiler 配置。首次配置后明确执行“建立语义索引”；已有索引时，Refresh workspace 先完成本地登记，再仅为新增或变化笔记增量计算。启动和普通编辑不自动调用 Embedding。语义索引失败或过期独立反馈，不使已成功的登记失效，允许重试。C1 交付基本召回、必要配置与可检查入口，采用 BM25＋Embedding 混合召回，可选 Reranker 只在查询时重排；其默认值依据质量与耗时验证选择。Reranker 独立配置和凭据，失败保留混合排名并提示。Analysis Profile 和两项 Analyzer 属于 C2。

## 4. Knowledge Compiler 与处理轮次

```text
Capture -> Source saved -> Registry
  -> pending batch (scheduled or manual)
  -> Compiler -> Draft title and body
  -> Draft Analyzer: Review Analyzer -> Relation Analyzer
  -> Human Review in Obsidian
  -> reviewed + Integration Intent (P3 input)
```

每轮执行前重读 Source 阶段、登记和生命周期；编译只选有效、active、pending、无进行中任务的 Source。reviewed 在 P3 用于规划；compiled／planned／archived 不能因为没有完成某个辅助任务就再次自动编译正文。Desktop 的明确单项或多选 Compiler 可选择 active 的 pending／compiled Source，新增另一份 Draft；这不改变 Recompile 仅返回 pending 的动作语义。

调度支持用户配置每天固定时刻或时间间隔，保存并展示时区，默认取本机当前时区；用户可在 Desktop 选取 Source 立即处理一轮。投递只保存 Source，不立即编译。Core 正常运行但忙时最多合并等待一轮，空闲后执行；停止、睡眠或重启期间错过的不补跑、不积压。保存设置不调用模型。

临时故障本轮有限重试，次数可设置。明确错误或耗尽结束本轮，保存错误及原阶段；下一轮符合资格可再试。没有 Draft 的正文失败不产生 failed 内容阶段。

Draft 已成功生成后，AI 分析是可选参考，Review Complete 不要求 Analyzer 尝试或结果；用户可跳过分析直接人工确认。默认完整轮次仍顺序执行分析，正在运行的轮次遵守互斥保护，不能误显示成已结束的失败。Analyzer 失败保留 compiled、显示失败且允许 Human Review。单独或批量重分析失败任务不重生成正文。

提供独立 Retry Review／Retry Relation 按钮，用户选择哪项就运行哪项，已有成功结果也可明确重试；不自动连带运行另一项或 Compiler。请求按当前输入／配置核验并冻结，其他结果保留其来源和版本；材料改变不强制重跑两项。Relation 只按配置复用同分析上下文内仍有效的 Review 材料或输出，没有合法输出就独立执行，不借不相关旧轮结果填缺口。

D 提前补最小 Recompile 动作及计数：反馈追加 Annotation、返回 pending、保留旧 Draft，不立即执行模型。每个成功接受的新用户 Recompile 请求计数一次，相同请求重放不重复计数；再次 Run Compiler 和模型自动重试独立记录，不计作 Recompile。其他 Review Note 动作和完整差异／恢复仍按 E／F 范围推进。

Core 完整轮次、时间／间隔调度、有限重试、独立／批量指定 Analyzer 和最小 Recompile 的已实现接口见 [Processing 合同](../engramweave/docs/processing.md)。当前阶段不执行 reviewed 的 Planner，只记录跳过原因；其他阶段不自动回退。最小反馈和原生 MVP 不等同于完整 E／F 或 P3。

## 5. 客户端职责和人工动作

### 5.1 Zotero

用户在阅读时明确选取段落或一组高亮／批注，执行投递，保存一个带论文及位置引用的 Source，并选择已配置的分析预设。其他材料不自动投递。投递后 Source 独立，Zotero 后续编辑不更新旧 Source；新材料再次投递。

插件只承担采集和必要反馈，不承担 Compiler、关系分析或整合逻辑。可复用两类 Capture 入口；实现路径根据真实 Zotero 能力决定，不建设文献管理器。

E1 的首批适配范围为 Windows Zotero 10.0.x PDF 阅读器，使用 Core API 保存 inline `source_type: paper`；评论保留在 Annotation，支持仅评论的投递。失败保留当前窗口冻结请求，明确重试或复制 Markdown，不添加离线后台队列。EPUB、OCR、全文提取和跨论文聚合不在首批范围。组件安装、选择和重试边界见 [Zotero README](../engramweave-zotero/README.md) 与 [Context](../engramweave-zotero/CONTEXT.md)。

### 5.2 Desktop

配置执行路径、模型、模板、Profile、调度与有限重试；显示 Source 阶段、生命周期、Job、Draft、分析失败。支持选定批处理、待编译筛选、重编译计数筛选和失败 Analyzer 的批量重分析。Desktop 不重建正文编辑器。

### 5.3 Obsidian

原生编辑 Draft，侧边栏显示 Source Annotation、Review Metadata、Relation／Integration Suggestions 和工作流动作。普通编辑不启动模型、不自动撤销 review，也不创建新审批点。

| 动作 | 语义 |
|---|---|
| 新建灵感 | 输入成为用户 Idea，回链 Source／Draft，直接创建，不经过 AI 审批。 |
| Recompile | 输入追加 Source Annotation；仅返回 pending，下一轮处理；已有 Draft 和用户编辑保留。 |
| Review Complete | 输入成为 Integration Intent，compiled→reviewed，允许规划，不冻结当前正文。 |
| 取消 Review Complete | Planner 未开始时 reviewed→compiled；P3 的生成候选取消按整体合同处理。 |

Review Note 每次被一个动作消费，成功消费后清空，不单独永久存储，也不插入 Draft 正文。要保留的 Integration Intent／Annotation／Idea 按各自语义保存。客户端断线或响应丢失不能丢输入或重复追加。

Integration Intent 归属于各自 Draft，Source 的当前规划选择与 Intent 内容分别维护。规划开始前对另一份 Draft 执行 Review Complete 可直接切换选择并保存该 Draft 的本次 Intent，其他 Draft 及其 Intent 保留；取消或 Recompile 撤销许可，不删除或转移这些内容，新 Draft 不继承其他 Draft 的 Intent。Obsidian 未提交的 Review Note 按 Draft 在当前插件会话内保留，文件切换和关开侧边栏不丢；插件重载／退出不保存未提交文字。已发送但送达不明的动作另留临时恢复记录和原请求 ID，查明结果后清理。见 Context 与 ADR-0016。

## 6. 实施切片与依赖

| 切片 | 内容 | 依赖与检查点 |
|---|---|---|
| A 合同和登记 | 协调扩展状态、属性写入、投影和恢复 | 现有 P1；新阶段可登记，缺失属性可补。 |
| B 执行和正文 | API／Codex 最小适配、任务配置、Source→Draft | A；各入口真实产出并守正文范围。 |
| C 召回和分析 | 三类语义上下文、两项 Analyzer、Profile 与复用 | B；同轮输入、独立结果、双路径。 |
| D 调度与状态 | 具体时间／间隔、手动轮次、有限重试和重分析 | A–C；不重复正文生成、不补跑。 |
| E Capture 和 Review | Zotero 选定材料、Obsidian 侧边栏及三动作 | 对应 Core 合同可用后可并行；不复制业务逻辑。 |
| F 编辑保护与验收 | Recompile／diff／恢复、真实场景、文档 | B–E；满足 P2-G，交给 P3。 |

## 7. 验证与交付

自动测试保护 Source 属性写入、阶段和 Job 隔离、正文／分析边界、轮次重试、输入消费、用户编辑保留和投影重建；使用现有测试栈。具体契约和布局在当前实现需要时确定，不冻结旧 AI 计划中的自造路由、数据表或字节绑定审阅协议。

真实验收包含论文选段／批注、网页材料、可合入正文的个人理解、不同表达的关联 Knowledge／Ideas／Research、分析失败后继续审阅、Recompile 保留用户修改。API／Codex 三任务分别有真实成功证据；客户端交互在 Zotero 和 Obsidian 中验证。

数据库重建不丢用户文件、Intent 和阶段；模型输出合法不代表忠实度通过，正文及分析质量需人工核验。测试、现场输出、临时证据放 .local/p2/；稳定文档记录合同及复现方法。

交付物为 Core 编译／分析／状态／召回／调度能力、Desktop 必要配置和管理、Zotero Capture 插件、Obsidian Review 侧边栏、可恢复 Draft 与 Integration Intent，以及满足整体计划 P2-G 的验证。正式知识写入留给 P3，不提前宣布实用闭环完成。

### Sources 展示与显式操作

Sources 按 All Sources／Pending／Processing／Archived／Issues／Discarded 六个 View 浏览。discarded Source 只出现在 Discarded，其余五个 View 可以重叠。Health 使用 available／missing／invalid／unsupported；Processing 直接展示 processing_status 的 pending／compiled／reviewed／planned／archived；Lifecycle 直接展示 lifecycle_status 的 active／discarded。详细任务与日志放对应页面。Processing View 精确选择 compiled／reviewed／planned；Issues 只按 Health 选择，不接收 Job 失败。筛选维度间 AND、同维度类别 OR，采用可删除 Filter Chips、Clear all 和排序。底部固定多选工具栏、独立目标确认弹窗和 Toast 避免选择导致列表跳动。

Core 按单项合同依次执行 Compile／Discard／Restore，并支持只 Discard 勾选 Draft 而保留 Source。Inspector 与操作预览不重复显示 discarded Draft。Source 的显式物理清理前移至此修订：只允许已 discarded 的 Source，明确清单确认，active 正式引用默认跳过、用户可明确覆盖；不自动删除 Draft 或改写坏链。当前 Capture 均为 inline，因此清理 Record 与内联正文；完整 Capture 专属独立 Asset／Derived Representation 清理仍随相关能力实施，不能按目录猜测资产归属。
