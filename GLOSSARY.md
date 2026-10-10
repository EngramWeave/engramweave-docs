# EngramWeave 领域术语

本文件定义总体设计中的知识生产术语。行为约束与边界见 [CONTEXT.md](CONTEXT.md)，总体架构见 [总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)。

## 知识编译

**Raw Source**：用户一次明确投递的材料所构成的处理单位，包含 Source Asset、Source Record 及可选的派生表示。不同 Source 可以引用同一篇论文，但分别对应各自投递的内容与位置。

**投递内容（Submitted Content）**：用户主动交给 Compiler 整理、并已判断具有保留价值的来源内容。论文场景中通常是用户选择的段落、标注和批注，而不是整篇论文。

**参考上下文（Reference Context）**：帮助理解投递内容的相关材料，例如论文全文或既有笔记。作为上下文提供的材料不因此成为本次整理对象。

**Knowledge Compiler**：人工审阅前的整体知识加工能力，包含 Compiler 和 Draft Analyzer 两个过程。它输出待审正文及独立的侧边栏分析。

**Compiler**：Knowledge Compiler 中负责对投递内容去噪、适度提炼并尊重原意、生成标题和正文的过程。
_避免_：论文自动总结器、全文知识发现器。

**Draft**：由投递内容编译而成、可编辑且等待人工审阅的知识候选；一个未 archived 的 Source 可以有多份 Draft，最终只有一份用于正式入库。入库成功后，该 Source 的所有相关 Draft 标记 discarded。

**Source Record**：带 Properties 的用户可读来源记录，保存一次投递的材料描述、原材料引用及认知上下文。工作流中的 Source 主要指这一记录，不等同于所引用的整篇论文。

**Source Asset**：原始资料本体，包括保存的 Markdown 正文、本地原材料或外部引用的资料。它与 Source Record 逻辑分离，即使两者位于同一个物理文件中。

**Derived Representation**：围绕 Source 或 Asset 生成、可以重新生成的机器表示。它辅助理解和检索，但不能替代原始材料。

**Human Review**：Knowledge Compiler 整体完成后，用户阅读、编辑并确认 Draft 正文的阶段。它与 AI 执行的 Review Analyzer 是不同概念。
_避免_：用未限定的 Review 指代 Review Analyzer。

**Review Complete**：用户允许一份 Draft 进入整合规划的确认动作。它不冻结 Draft 正文，也不代表已经批准正式知识修改。

**Reviewed Draft**：用户已允许进入整合规划、仍可继续编辑的 Draft；它不是某个不可变正文版本的名称。

**Review Note**：用户审阅 Draft 时，为一次明确动作填写的临时输入；由新建灵感、Recompile 或 Review Complete 中的一个动作消费。

**Integration Intent**：用户随 Review Complete 提交、表达所选 Draft 应如何整合进知识库的意图，作为 Integration Planner 的输入。
_避免_：AI Integration Suggestions、Source Annotation、尚未提交的 Review Note。

**Review Metadata**：AI 在处理过程中提出的不确定性、潜在错误、待验证 Claim、关系和整合建议等审阅辅助信息。它们属于 Control Plane，不是 Draft 正文或用户已经认可的知识。

**Draft Analyzer**：围绕待入库 Draft 生成侧边栏分析的整体能力，由 Review Analyzer 和 Relation Analyzer 两个独立子任务组成。它不修改 Draft 正文。

**Review Analyzer**：按照分析模板审视 Draft 内容、投递原意及相关知识的分析能力。它产生内容疑问、错误提示、核对结果或总结等审阅辅助信息。

**Relation Analyzer**：按照分析模板发现 Draft 与相关知识、想法和研究材料之间的联系、冲突及整合线索的分析能力。其建议不等于用户认可的关系。

**Semantic Recall**：从 Knowledge、Ideas、Research 中找回含义相关、表达可以不同的材料候选的检索能力。召回候选不代表关系已成立，未召回也不能证明没有联系。

**分析模板（Analysis Template）**：面向某类材料或分析目的的一组分析要求及上下文需求。它不等同于待分析材料本身。

**Analysis Profile**：组合 Review/Relation 分析模板、模型、执行路径及上下文／输出复用偏好的一套分析方案。

**Source Annotation**：用户围绕 Source 长期保留的认知上下文，包括保存理由、理解、疑问和后续整理指令。它可以与投递原文一起被整理进 Draft，不要求在正文中单独标示为用户判断。

**Integration Planner**：在正文审阅后，根据 Draft、关系分析、用户整合建议与当前知识库，统一规划知识重组并生成候选 ChangeSet 的能力。新建、修改、拆分、融合、重组和废弃是可能的整合结果，不是独立模式或工作流。它不直接执行正式知识修改。

**ChangeSet**：一次性、可消费的知识库修改提案及执行计划，供用户审阅批准后执行。它是临时工作流对象，不是正式知识资产或通用回收站中的文件。

## 整合后的内容

**Canonical Knowledge**：用户已经认可并整合的、可独立使用的概念、原理和方法等内容知识。它按内容用途区别于研究材料，不以原始来源类型划分。

**研究材料（Research Material）**：围绕特定论文、实验或研究问题的结果、条件、证据和分析。研究材料所在位置与其是否经过用户认可，是两个不同的概念。

**Source 归档**：一份投递内容经编译、审阅后产生的最终内容已经完成受控整合的事实。它不表示所引用的整篇论文或其他选段已经全部处理。

**Research Question**：归属于研究课题的一类普通 Markdown 笔记，表达该课题下的具体研究问题。其他材料通过链接关联它，也可以在 `research_question` 属性中保存对应笔记链接。

## 处理阶段

**processing_status（Processing）**：一份 Source 的内容处理阶段，取值为 `pending / compiled / reviewed / planned / archived`。它不等同于文件生命周期或某次任务是否运行成功。

**lifecycle_status（Lifecycle）**：文件的可恢复生命周期标记，取值为 `active / discarded`。discarded 表示待清理或停止使用，不表示文件已经物理删除，也不覆盖此前的内容处理阶段。

**Discard**：将材料标记为 discarded、停止当前使用但仍保留文件的可恢复动作。单独 Discard Draft 不表示 Discard 其来源 Source。
_避免_：用“删除”指代生命周期标记。

**Physical Delete**：对已 discarded 材料另行确认后，永久移除其文件的清理动作。
_避免_：Discard、把标记废弃当成物理删除授权。

**Source Health**：一份 Source 当前是否可用、存在、有效且受支持，取值为 `available / missing / invalid / unsupported`，独立于内容阶段及执行成败。
_避免_：Source State、用 Ready 表示可用性。

**job_status**：一次具体任务的执行状态，取值为 `queued / running / succeeded / failed / interrupted`。同一 Source 可以在不同时间拥有多次任务记录。

**待编译（pending）**：一份 Source 正在等待编译处理的阶段，包括新投递材料和被明确打回 Recompile 的材料。它是定时编译的候选阶段，不等同于归档属性缺失。

**Recompile**：将一份 Source 明确送回待编译阶段、供后续轮次生成另一份 Draft 的用户动作，已有 Draft 与用户修改保持可恢复。它本身不等同于立即执行模型调用。

**Recompile count**：用户成功提交的独立 Recompile 动作次数。它区别于 Compiler 执行次数、模型重试次数和 Draft 数量。

**已编译（compiled）**：一份 Source 已产出当前待审阅 Draft 的阶段。

**已审阅（reviewed）**：用户已允许 Draft 进入整合规划的阶段。它不表示正文被冻结，也不等同于知识归档完成。

**已规划（planned）**：Planner 已生成可审阅 ChangeSet、等待批准与执行的阶段。存在整合方案不等于已经批准或归档。
