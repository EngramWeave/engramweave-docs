# P3 实施清单

> 合同：[P3_IMPLEMENTATION_PLAN.md](P3_IMPLEMENTATION_PLAN.md)；上位范围：[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)。
> P1、P2 开发已完成。P3-A 开发已完成，原生宿主程序化验证与真实 Agent 验证分别留在本地记录；B/C 合并为一次交付，BC 开发已完成、原生程序化和真实 Agent 验证已记录；D 待实现，真人验收尚未回填。

实施遵守各仓库 AGENTS、文档规范和涉及测试的 tests/AGENTS。复用 P2；模型 API 扩展暂缓，保留现有配置与实现。本地 Core HTTP API、Embedding／Reranker 不在暂缓范围。

## A：Agent 工具与候选

详细任务按 [P3_A_TODO.md](.local/P3/P3_A_TODO.md) 跟踪，验收依照 [A 计划](.local/P3/P3_A_IMPLEMENTATION_PLAN.md)；以下保留上层范围。

- [x] 在 Obsidian ItemView 验证 CodeMirror 6 MergeView：左侧只读、右侧可编辑，真实 Markdown、键盘、滚动及关闭保存体验可用。
- [x] Core 管理 `70_ChangeSets/<id>/manifest.json` 与 `candidates/`，候选镜像正式相对目标，只保存涉及文件，不建立 `bases/` 或三方合并。
- [x] 明确排除 `70_ChangeSets` 的 Git、正式登记／全文检索、语义索引及 Source／Draft 发现；说明 Obsidian 搜索／Sync 排除需单独配置。
- [x] 从现有 Human Review 读取当前所选 Draft 和专属 Intent，固定本轮输入；工具上下文记录实际读取版本。
- [x] 复用 P2 Codex 适配、Agent loop、模板和独立模型配置，新增 Agent Planner；不强制新增模型 API 路径或更换运行器。
- [x] 盘点并补齐 Core Tools／MCP：任务输入读取、三类语义召回、当前笔记与目录读取、本轮候选创建／读取／编辑／校验。
- [x] Core 按任务强制权限：Compiler 限投递内容；Analyzer 只读、结果不改正文；Planner 只写本轮候选，无正式写入／批准／执行／删除工具。
- [x] 按真实缺口改善 Review／Relation 的 Agent 工具使用，保持独立任务和结果语义；保留 P2 API 能力。
- [x] 复用 Relation 模板与召回，重新验证关系，统一生成跨文件正文、来源、关系、目录和废弃候选。
- [x] 校验范围、候选完整性与可执行性；完整发布才进入 planned，失败保留原阶段。
- [x] 真实 Agent 通过工具读取／检索／写候选，正式文件保持原样，越权路径被拒绝。

## BC：候选审查、批准执行与恢复

详细任务按 [P3_BC_TODO.md](.local/P3/P3_BC_TODO.md) 跟踪，合同见 [BC 计划](.local/P3/P3_BC_IMPLEMENTATION_PLAN.md)。下列两部分属于一次交付，沿用 A 的侧栏和编辑能力。

### 候选审查

- [x] 同一工作台列出涉及文件及结构影响；Current ↔ Candidate 双栏，新增文件左侧为空。
- [x] 右侧编辑与 `Ctrl+S` 仅保存 Candidate；Refresh Current 保留候选修改。（已随 A 实现。）
- [x] 切换、关闭与重开时明确未保存状态并保护编辑；Draft 保持原生编辑，Desktop 不新增第二套可编辑 Diff。（已随 A 实现。）
- [x] 同一机制支持取舍、拒绝、整组处理、打回重新规划和明确批准；首版不依赖逐 hunk 操作。
- [x] 批准前保存本方案全部脏候选并校验有效集合，绑定最终批准字节和最后审查的 Current 前置条件；并发冲突可见。
- [x] 普通 Draft／本地编辑不自动撤销候选或启动规划；取消 Review Complete／取消或丢失候选沿既有阶段语义处理。（已随 A 实现。）
- [x] 展示来源、关系、目录、废弃及新一级领域影响；拒绝后的依赖校验，未批准过程不写正式内容。

### 批准执行与恢复

- [x] Executor 应用固定的最终批准内容并检查目标前置条件，后续 Candidate／Draft 编辑不改本次执行。
- [x] Knowledge／Research 与多文件重组支持，同份 Draft 可贡献多份正式笔记。
- [x] 快捷 Publish 保留人工新建体验，共用执行／归档／恢复，不强制 Planner 或额外批准。
- [x] 成功归档 Source，只自动 discarded 相关 Draft，撤销规划选择并保留 Intent。
- [x] AI 废弃走批准标记，物理清理保持独立明确授权。
- [x] 复用原生文件保护与必要的 Core 执行回执，不重复覆盖已应用内容；不承诺跨文件事务。
- [x] 升级后恢复未完成 P2 MVP 回执，不丢已接受审批。
- [x] 成功／明确取消后清理 ChangeSet 工作区；未完成批准操作保留，必要结果归 Job；备份包含候选与 Core 恢复事实。

## Desktop Review / Changes 增量

设计见 [本地页面方案](.local/P3/P3_DESKTOP_REVIEW_CHANGES_DESIGN.md)。采用日常管理产品范围并分步交付，正文编辑和批准留在 Obsidian。

- [ ] Review 全局 Draft 文件列表、筛选、详情、分析概况和关联跳转，包含旧稿 / discarded / 异常文件。
- [ ] Changes 存活工作区列表、全部影响、候选预览及任务归属，不复制 Jobs 历史。
- [ ] 接通已有 Core 安全取消 / 重新规划及当前选中稿的规划入口，已接受执行不可取消。
- [ ] Review Complete 对其他 active 同 Source 稿件默认提示废弃、支持保留，Core 统一接受 / 恢复且保护新选择。
- [ ] 原生页面、窄窗口、断线、固定宿主路由及受影响回归；真实 / 模拟 / 自动 / 真人证据区分。

## D：Git、生命周期与 P3-G

- [ ] 约定跟踪／排除范围，Source 只跟踪 Record Markdown，排除 `70_ChangeSets`。
- [ ] 成功应用只提交本次涉及文件，包含其原未提交编辑；不提交无关工作区／暂存内容。
- [ ] 快捷新建同样自动提交；用户手动／定时 commit 入口。
- [ ] Git 失败单独可见、可恢复，不能重复应用正式变更。
- [ ] 复用 Source／Draft 操作，补足正式文件标记／恢复／清理；Source Restore 已按 Q43 仅恢复 Source 为 active/pending。
- [ ] 补齐独立 Draft 废弃／恢复／清理；废弃当前所选稿统一撤销未批准流程、Source 返回 compiled，不按剩余 Draft 数量决定；未选稿不改阶段，archived 不回退，Restore 不授予审阅许可。
- [ ] 来源引用默认保护、用户明确覆盖、专属 Asset／Derived 与外部 Asset 边界。
- [ ] 应用后登记及基本找回可用，候选副本不混入召回，Embedding 保持显式更新。
- [ ] Agent Planner、论文／网页、跨文件重组、Research、快捷新建与 Git 真实验收，不要求新增 Planner API。
- [ ] 候选持久化、工具权限、并发本地编辑、拒绝／打回、中断、归档、Git 和数据库恢复的有意义回归。
- [ ] 更新组件合同、Context 与必要 ADR，临时资料放 `.local/p3/`。
- [ ] 达到总体 P3-G，交付受控整合及再次找回闭环。
