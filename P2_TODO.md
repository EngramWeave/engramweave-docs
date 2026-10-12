# P2 实施清单

> 合同：[P2_IMPLEMENTATION_PLAN.md](P2_IMPLEMENTATION_PLAN.md)；上位范围：[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)。
> P1、P2 开发已完成。本清单保留历史实施合同及人工验收记录，当前施工见 [P3_TODO.md](P3_TODO.md)。

实施时遵守各仓库的 `AGENTS.md`、文档规范以及涉及测试的 `tests/AGENTS.md`，按 A–F 的实际依赖推进。有意义的资产、状态与恢复测试随相关代码交付，不以类型检查替代行为验收。

本文保留 P2 双路径的已交付范围。当前 P3 优先 Agent 与 Core Tools／MCP，保留已有模型 API、暂缓其扩展；不将历史双路径要求视为新增 Planner API 的前置条件。见 [ADR-0019](docs/adr/0019-agent-first-delivery-and-core-tools.md)。

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
- [x] Review Note 新建灵感，正确回链 Source 和 Draft。
- [x] Recompile 追加 Annotation、返回 pending，不立即执行模型；最小动作由 D 提供，E 完整 Review Note 集成继续复用。
- [x] Review Complete 保存当前 Draft 自己的 Intent、返回 reviewed；AI 分析可跳过，不要求 Analyzer 尝试；规划前可直接切换所选 Draft，其他 Draft／Intent 保留，无精确正文绑定。
- [x] 规划未开始时取消 Review Complete 返回 compiled。
- [x] 动作正确消费／清空输入；按 Draft 保留会话内未提交文字，送达不明请求以临时恢复记录和原 ID 查明，断线和重试不丢输入或重复追加。

## F：资产保护、文档与 P2-G

- [x] 整体性能测量与实证优化：登记、查询、Draft 读取、反链／批次及真实 Embedding；保留文件安全、文档原子发布和模型调用边界。
- [x] Recompile 与后续编译以独立 Draft 保留旧稿／用户修改，不依赖 Git；B／E 已实现，F 做相关保留回归，不新增跨 Draft diff 或 rollback 界面。
- [x] 数据库损坏后用户资产、Intent 和阶段仍可访问／重建。
- [x] 有意义的合同、状态、文件、Job 和失败恢复回归。
- [x] 真实论文／网页／个人理解／三类关联材料与分析失败场景检查；模型格式通过不代表语义质量，正文与建议仍需人工核对。
- [x] API／Codex 三任务及 Zotero／Obsidian 原生程序化验证；实际鼠标、浏览器扩展弹窗与本人材料另做人工检查。
- [x] 更新各组件长期合同、Context 和备份恢复说明；临时记录放 .local/p2-f/。未改变架构，不新增 ADR。
- [x] P2 开发及程序化验证完成，交付 Reviewed Draft + 各 Draft 自己的 Intent；正常流程不执行 Planner／ChangeSet，ADR-0015 的人工 MVP 新建／归档例外保留。
- [ ] 同步实际鼠标操作与本人材料的人工验收记录；程序化验证不代替该记录，已完成检查无需重做开发。
