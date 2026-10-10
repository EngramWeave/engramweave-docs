# P2 实施清单

> 合同：[P2_IMPLEMENTATION_PLAN.md](P2_IMPLEMENTATION_PLAN.md)；上位范围：[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)。
> P1 已完成。按切片推进；尚未勾选不表示重做既有基础。

实施时遵守各仓库的 `AGENTS.md`、文档规范以及涉及测试的 `tests/AGENTS.md`，按 A–F 的实际依赖推进。有意义的资产、状态与恢复测试随相关代码交付，不以类型检查替代行为验收。

## A：合同、登记和投影

- [x] 扩展共享处理／生命周期／登记／Job 状态定义及受影响客户端。
- [x] Registry 对缺失或空 processing_status 补 pending、lifecycle_status 补 active，保护其他内容和属性。
- [x] 支持五阶段登记与可重建投影，错误和次数保留在 Core。
- [x] 验证文件阶段读取、重建不自行重置、现有资产合同相关回归。

## B：真实执行和正文

- [x] 实现最小 API、Codex 适配，复用现有成熟执行能力。
- [x] Desktop 配置任务模型／路径，凭据和配置遵守现有资产边界。
- [x] Compiler 用投递内容及 Annotation 只返回标题和正文。
- [x] 支持未 archived Source 的多份 Draft，完整保留已有 Draft、revision 与用户修改；最终一份正式入库后统一 discarded 由 P3 实现。
- [x] 真实 API／Codex 各验证正文范围、用户理解合入与来源回链。
- [x] pending／compiled 的显式重复 Compiler、Draft captured_at／annotation 保留及 90_System 用户模板。
- [x] Sources 六个 View、五阶段／生命周期标签、Health、组合 Filter Chips、时间与排序。
- [x] Discarded 独占 View、底部稳定工具栏、独立确认弹窗、Toast 和 active Draft 反链索引。
- [x] 只 Discard 勾选 Draft 并保留 Source；显式清理 discarded inline Source，校验文件身份与正式引用。

## C：召回、模板和两项分析

- [x] 扩展 Knowledge／Ideas／Research 读取上下文，建立基本语义召回。
- [x] 用不同表达的相关材料验证召回，并记录有限覆盖。
- [x] Desktop 配置 Review／Relation 模板、内容、模型、路径和 Profile。
- [x] 预设选择可在 Source pending 期间修改，执行读取最终选择和当前配置。
- [x] Review Analyzer、Relation Analyzer 同轮输入、独立模型和独立结果。
- [x] Relation 三档复用与侧边栏只读结果合同，不改正文和正式知识；原生侧边栏展示由 E2 接入。
- [x] 两项任务分别验证 API／Codex 路径，必要 Core Tools／MCP 只承接当前需求。

## D：轮次、失败和批处理

- [x] Desktop 手动选定 Source 的 Compiler 批处理，逐项重读与报告，无启动补跑。
- [x] 每天固定时刻／间隔与时区配置，以及 Core 编译／分析完整处理轮次；忙时合并一轮，停止／睡眠／重启不补跑。
- [x] 每轮重读阶段／登记／生命周期，排除无资格和进行中材料。
- [x] 本轮有限可设置重试，结束后保留原内容阶段和失败历史。
- [x] 已有 Draft 时只重试失败分析，不重新编译正文。
- [x] 显示分析失败，可继续审阅；独立 Retry Review／Retry Relation 与单独／批量重分析，只运行用户选择的任务。
- [x] 最小 Recompile 反馈动作及幂等计数：追加 Annotation、返回 pending，不立即执行模型；不把重复 Compiler／自动重试计作 Recompile。
- [x] Desktop 展示 Source／Draft／Job，提供待处理和重编译计数筛选。

## E：Zotero Capture 与 Obsidian Review

提前交付的 Obsidian MVP 使用 Context／ADR-0015 的人工直接入库路径；不等同于以下完整 E 的全部动作或 P3。自动 Vault Git 暂缓。

- [x] Obsidian MVP：现有 Source→Compiler→Analyzer、原生 Draft 编辑、侧边栏结果及直接入库 `40_Knowledge`，保留 Source 和 Draft、可恢复归档。

- [x] Zotero 投递明确选择段落／高亮／批注，一次独立 Source；Windows Zotero 10.0.x PDF 入口，Core API 保存，失败保留冻结请求供明确重试。
- [x] 保留论文和位置链接、分析预设；不投其他材料、不同步后续修改。仅评论投递保留 Annotation，不伪造论文正文。
- [x] 原生 Obsidian 编辑与侧边栏 Source Annotation／AI 分析／关系建议。
- [ ] Review Note 新建灵感，正确回链 Source 和 Draft。
- [x] Recompile 追加 Annotation、返回 pending，不立即执行模型；最小动作由 D 提供，E 完整 Review Note 集成继续复用。
- [ ] Review Complete 保存当前 Draft 自己的 Intent、返回 reviewed；规划前可直接切换所选 Draft，其他 Draft／Intent 保留，无精确正文绑定。
- [ ] 规划未开始时取消 Review Complete 返回 compiled。
- [ ] 动作正确消费／清空输入；按 Draft 保留会话内未提交文字，送达不明请求以临时恢复记录和原 ID 查明，断线和重试不丢输入或重复追加。

## F：编辑保护、文档与 P2-G

- [ ] Recompile 保留旧 Draft／用户修改，有 diff 和恢复能力，不依赖 Git。
- [ ] 数据库损坏后用户资产、Intent 和阶段仍可访问／重建。
- [ ] 有意义的合同、状态、文件、Job 和失败恢复回归。
- [ ] 真实论文／网页／个人理解／三类关联材料与分析失败场景验收。
- [ ] API／Codex 三任务及 Zotero／Obsidian 原生操作真实验证。
- [ ] 更新各组件长期合同、Context 和必要 ADR；临时记录放 .local/p2/。
- [ ] 达到整体 P2-G，交付 Reviewed Draft + Intent，不写正式知识或归档。
