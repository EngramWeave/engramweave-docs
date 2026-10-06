# EngramWeave 总体实施清单

> 基线：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[CONTEXT.md](CONTEXT.md)、[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)。
> P1 已完成；其后的未勾选项表示尚需实现和验收，不否定既有成果。

## P0：设计与语义基线维护

- [x] 明确 Source 投递范围、Compiler 与 Draft Analyzer 层级、人工审阅和候选批准边界。
- [x] 明确状态所有权、Recompile、重试、生命周期和清理规则。
- [x] 明确统一知识重组、首期语义召回、四任务双路径和 Git 范围。
- [ ] 随实际架构决定维护总体设计、Context、Glossary 与必要 ADR。

## P1：已完成的本地基础

- [x] 独立 Core、Desktop 基础与资产／运行数据分离。
- [x] Source 登记、Capture、原材料引用、读取和基础查找。
- [x] 数据库投影恢复基础与 P1 合同验收。

## P2：论文投递、编译与人工审阅

详细任务见 [P2_TODO.md](P2_TODO.md)，验收范围见 [P2_IMPLEMENTATION_PLAN.md](P2_IMPLEMENTATION_PLAN.md)。

- [ ] 协调扩展处理阶段、生命周期、登记与 Job 合同；Registry 缺失／空阶段补 pending。
- [ ] 实现 API 与 Codex 最小执行适配，以及按任务配置模型和路径。
- [ ] Desktop 配置 Review／Relation 模板、Analysis Profile 和预设选择。
- [ ] 为 Knowledge、Ideas、Research 建立基本语义召回。
- [ ] Compiler 根据投递内容及 Annotation 只生成标题／正文。
- [ ] Core 编排独立 Review Analyzer、Relation Analyzer，结果只进侧边栏。
- [ ] Relation 三档上下文／结果复用，分析绑定本轮输入。
- [ ] 定时具体时间／间隔和 Desktop 手动批处理，无启动补跑。
- [ ] 有限本轮重试、错误历史、单独及批量重分析，失败保留阶段与正文。
- [ ] Zotero 插件投递明确选择的段落／高亮／批注，保留论文及位置引用和预设。
- [ ] Obsidian 原生编辑与工作流侧边栏，Review Note 三动作与取消确认。
- [ ] Recompile 仅返回 pending、保留旧 Draft 编辑、提供 diff／恢复。
- [ ] 完成 P2-G；三类模型任务的 API／Codex 和真实论文场景可用。

## P3：统一整合与首个实用版本

- [ ] Integration Planner 根据当前输入统一规划跨文件知识重组，可选 API／Codex。
- [ ] 一个候选 ChangeSet 支持完整重组结果，不分四种独立业务模式。
- [ ] 与当前本地文件 Diff，候选编辑／取舍／拒绝／重新规划／批准。
- [ ] 未批准候选不写正式文件；批准后立即由 Executor 应用最终内容。
- [ ] 前置条件、异常中断、归档属性、避免重复应用和 Git 失败恢复。
- [ ] Knowledge／Research 按用途整合，来源回链、关系及目录调整受批准控制。
- [ ] 入库 Source archived；只自动 discarded 相关 Draft。
- [ ] Git 约定跟踪范围、仅提交本次涉及文件，用户编辑主动／定时提交。
- [ ] 生命周期管理、引用列表、默认跳过仍被引用 Source、恢复与统一清理。
- [ ] 删除专属受管理 Asset／Derived，不操作外部 Zotero 文件。
- [ ] 完成真实 Zotero 和网页共同闭环、基本检索与来源追溯的 P3-G。

## P4：检索与科研增强

- [ ] 改进已有语义召回、关系检索和认知重载，验证遗漏与来源正确性。
- [ ] Research Question 普通笔记、链接与 research_question 属性使用体验。
- [ ] 有来源的支持／反对／冲突／未解决证据表达与跨材料整合。
- [ ] 研究内容继续复用审阅与 ChangeSet，不建设独立无审批写入路径。

## P5：知识维护

- [ ] 按真实需要增加重复、冲突、过时、结构及引用问题分析。
- [ ] 有依据的 Maintenance Proposal，用户查看，不主动推送。
- [ ] 维护变更复用 ChangeSet、生命周期及执行恢复机制。

## P6：复杂 Agent 与外部工具扩展

- [ ] 在首期适配之上增加复杂研究、跨笔记维护和长任务能力。
- [ ] 按需要扩展 Core Tools／MCP、任务控制与 Agent 配置。
- [ ] 保持 READ + PROPOSE 默认边界和正式写入批准机制。
- [ ] 仅在当前具体需要成立时增加 Core 自建工具循环。

## P7：多端和多模态

- [ ] 按真实需求选择 Mobile／Capture／Worker 能力。
- [ ] 验证统一 Source 投递单位、原材料保护和可重建派生数据。
- [ ] 需要时单独验证 Hybrid，不让远程成为正式知识唯一载体。

## P8：稳定化与发布

- [ ] 整理实际支持范围、安装／升级、兼容和恢复说明。
- [ ] 验证发行物和关键真实使用流程。
- [ ] 跨阶段持续维护有意义回归、资产保留和故障恢复测试。
