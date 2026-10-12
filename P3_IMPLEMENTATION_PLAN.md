# P3：Agent 驱动整合、Vault 候选审查与批准执行

> 基线：[总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)、[CONTEXT.md](CONTEXT.md)、[总体计划](IMPLEMENTATION_PLAN.md)。
> 阶段状态：A 与 BC 开发已完成，B/C 为一次交付；D 待实现，真人验收单独记录；P1、P2 开发已完成。更新日期：2026-10-12。
> 配套清单：[P3_TODO.md](P3_TODO.md)；范围决定：[ADR-0019](docs/adr/0019-agent-first-delivery-and-core-tools.md)、[ADR-0020](docs/adr/0020-vault-local-changeset-review.md)。

## 1. 目标与边界

将已审阅内容融入当前知识体系，实现 Agent 驱动的统一 Planner、Obsidian 内候选审查、确定性批准执行及限定范围 Git 提交。论文选段与网页材料共用流程；Knowledge 和 Research 按内容用途落点，正式内容可以再次找回并溯源。

新建、修改、拆分、融合、重组和废弃描述可能结果，不是四种产品模式或四套 workflow。第一版保留完整知识重组能力，不将 Planner 收窄为仅创建新笔记。

当前优先改善 Agent 接入与供 Agent 使用的 Core Tools／MCP。复用 P2 Codex 运行适配和 Agent 自带 loop；新增 Planner 先走 Agent，不要求同期实现模型 API 路径。P2 已有 Compiler／Review／Relation 的 API 配置、实现和结果继续保留；模型 API 的扩展与效果优化暂缓，不删除或重做 P2。Embedding／Reranker 是已有独立召回能力，不因暂缓生成模型 API 而停用。Core 面向客户端的本地 HTTP API 同样继续维护。

保留人工 Publish to Knowledge 快捷新建入口，不强制调用 Planner 或再增加批准步骤；P3 将其接入共用 Executor、归档恢复和自动 Git。它仍不允许直接覆盖已有笔记，完整重组通过 Planner／ChangeSet。现有 P2 入口仍没有自动 Git，不能把规划当作已交付行为。见 [ADR-0018](docs/adr/0018-shared-execution-for-direct-publication.md)。

复杂研究、跨笔记 Maintenance、完整 Agent 调度平台、Core 自建工具循环及新的多模态 Asset 管理不在本阶段。用户已确定的领域语义不再逐项询问实现机制；接口、并发、幂等和异常恢复由 coding agent 按当前代码选择最小实现。

## 2. 已有能力与复用边界

| P2 基础 | P3 用法 |
|---|---|
| 当前 Reviewed Draft 选择及 Draft 自己的 Intent | Planner 从现有 Human Review 读取，不从 Source 阶段猜测目标，不建立共享 Source Intent 槽。 |
| Codex 执行、任务模型、模板、Core Tools／MCP | 优先补当前 Agent 任务所缺工具；新增 Planner 配置，复用执行适配，不强制更换 SDK 或搭建新平台。 |
| 已有模型 API 适配 | 保留现有选择和兼容行为；不列入本阶段新增功能及双路径验收矩阵。 |
| 三类语义召回与 Relation 结果 | 检索当前材料，参考旧关系并按本轮正文重新验证；不将过期分析当批准事实。 |
| 原生文件保护、独占发布、属性日志与回执恢复 | 扩展为当前具体多文件批准变更；不宣称文件系统具有跨文件事务。 |
| MVP 直接新建及已归档旧笔记 | 保留快捷体验和旧请求恢复；旧笔记可被新投递的 Planner 选为目标，不重跑其 archived Source。 |
| Source／Draft 生命周期和引用预览 | 补足正式文件清理和 Planner 废弃结果的实际缺口，不重新实现 Sources 页面。 |
| 显式 Build／Refresh 语义索引 | 应用后更新本地登记；Embedding 仍沿显式入口，不因批准内容偷偷调用模型。 |

独立 Source Asset 清理以明确归属为前提；当前 inline Capture 和外部 Zotero 引用已覆盖首个场景。没有当前使用对象时，不预建二进制资产平台或共享引用计数。

## 3. A：Agent 工具与候选工作区

详细施工顺序、代码复用与 A-G 验收见 [A 计划](.local/P3/P3_A_IMPLEMENTATION_PLAN.md) 和 [A TODO](.local/P3/P3_A_TODO.md)。A 包含候选编辑器验证及实际候选读写，B 完成最终审查／批准；A 的候选、Agent Planner 与 reviewed 分流已实现；B–D 尚未完成。

### 3.1 先验证审查体验

先在 Obsidian 自定义 ItemView 中验证 CodeMirror 6 `@codemirror/merge` 的双栏 MergeView：左侧 Current 只读、右侧 Candidate 可编辑，支持保存、切换文件和刷新左侧。使用真实 Markdown 与插件构建验证依赖兼容、键盘操作、长文滚动和窗口关闭后的保存行为；成熟组件负责编辑及 Diff，不搬入完整 VS Code 或整套第三方插件。此切片只验证审查与候选保存，不提前写正式文件。

### 3.2 Core 管理候选，Agent 使用工具

ChangeSet 工作区放在 Vault 根目录 `70_ChangeSets/<id>/`：

```text
70_ChangeSets/<id>/
  manifest.json
  candidates/
    40_Knowledge/example.md
    50_Research/topic/result.md
```

`manifest.json` 保存提案说明、目标映射、来源／证据、所需目录或废弃标记以及执行前置条件等必要信息；候选文件按正式目标相对路径组织。只保存涉及的候选，不复制整个 Vault；不建立 `bases/` 或持久化完整 Base 副本，不做自动三方合并。目标版本检查和必要的临时执行备份仍由 Core 实现。

优先提供当前任务需要的 Core Tools／MCP：读取本轮 Source／Draft／Intent／分析结果，检索 Knowledge／Ideas／Research，读取并核对当前正式笔记与目录，以及创建、读取、修改、校验本轮候选。先盘点并复用 P2 工具，仅补实际缺口；具体工具名称和协议由实现确定。

权限在 Core 端按任务和工作区强制限制。Compiler 保持只整理投递内容与 Annotation、输出标题／正文；Review／Relation 只读证据、保存分析结果，不能修改 Draft 正文；Planner 可写本轮候选，不能写正式内容或其他 ChangeSet。Agent 不获得批准、执行或物理删除正式内容的工具，不能通过通用 shell／任意文件访问绕过边界。

`70_ChangeSets` 不进入 Git、正式登记／全文和语义召回，也不被识别为 Source 或 Draft。显式候选读取与审查不受此排除影响。说明 Obsidian 自己的搜索排除和 Sync 设置需单独配置，不能把 `.gitignore` 当作两者的排除规则。

### 3.3 Agent Planner 生成方案

在轮次启动时读取有效、active、reviewed Source 当前所选 Draft、其 Intent、关系结果及当前知识库，固定本轮输入；Agent 工具获取的正式文件上下文记录实际读取版本，同一文件的冲突读取应报告，不建立全库快照。

Planner 独立配置模板、模型和 Agent 入口，复用 Relation 模板、既有运行器 loop 与必要工具，重新验证关系。只读 Analyzer 的任务约束保持独立；根据真实材料暴露的缺口改善其 Agent 工具能力，不改为多任务混合正文生成。

统一生成跨文件正文、关系、来源、目录和废弃候选。必要新目录属于提案，新一级领域须明确批准。Core 校验目标范围、候选完整性和计划可执行性，只有完整可审方案发布后 Source 才进入 planned；失败保留原阶段，Job 记录失败。A 的出口是真实可读取的候选和可用 Agent 工具，不是固定返回样例的 Planner。

## 4. BC：候选审查与重新规划

B 与 C 合并为一次交付，详细施工顺序见 [BC 计划](.local/P3/P3_BC_IMPLEMENTATION_PLAN.md) 与 [BC TODO](.local/P3/P3_BC_TODO.md)。本节与 §5 同属 BC；内部先补合同、执行与恢复，再接通真实批准，不分别交付仅有审批 UI 或缺少恢复的 Executor。A 已完成的侧栏、候选保存/预览和原 Draft 跳转直接复用。

审查以 Current ↔ Candidate 两栏为主。左侧读取审查时的当前正式文件，只读；右侧编辑候选，用户自行取舍两边内容，形成最终批准版本。新建文件的左侧为空；目录、来源、关系和废弃影响在同一 ChangeSet 中说明。用户可拒绝条目或整组、打回重规划，最后明确批准；不要求逐个 Diff hunk 操作作为首版前置条件。

`Ctrl+S` 只保存 Candidate，不应用正式内容。Refresh Current 更新左侧与对比，保留右侧修改；切换、关闭和重新打开时明确展示未保存状态，避免候选编辑丢失。批准前保存并校验最终候选，绑定本次批准内容与目标前置条件；已有外部编辑冲突必须可见，不能静默覆盖。Desktop 管配置、任务和状态，首版不再建设另一套可编辑 Diff 界面；Draft 继续使用 Obsidian 原生编辑器。

Draft 普通编辑不撤销 Review Complete、不触发重新规划，也不自动废除候选。本轮固定输入不随编辑变化；取消 Review Complete 废除未批准候选并返回 compiled，之后重新确认。明确取消或丢失未批准候选时，在所选 Reviewed Draft 仍可用的情况下返回 reviewed，等待下一轮；新候选重新批准。

生成、预览、修改候选及打回操作都不写正式内容。条目依赖及拒绝后的可执行性由实现校验，不让用户设计执行协议。

## 5. BC：统一批准执行、归档与恢复

批准后由 Executor 立即应用最终批准内容，检查目标前置条件。前置条件来自最终审查的 Current，不直接沿用 Planner 的旧读取版本；全部条目须已明确纳入或拒绝，批准固定有效集合和字节。AI 不参与最终写入。快捷新建使用同一执行保障，沿现有人工确认体验，不额外调用 Planner。

支持 Knowledge／Research 与跨文件知识重组，单份 Draft 可融入多份正式笔记。保护已有正文、Properties、来源链及当前未提交修改。AI 废弃提案只产生批准后的 lifecycle 标记，物理删除仍是标记后的另一次用户清理。

本次条目均已作出决定、批准整合成功且 Source 归档属性保存成功后，Source archived；只自动 discarded 相关 Draft，Source Annotation、原材料和其他正式文件不因此废弃。既有 Draft 与 Intent 按合同保留，Source 的规划许可随归档撤销。

未批准候选可取消重建；已批准执行的异常保留必要恢复事实于 Core 应用数据，不能以重跑 Planner 或再次覆盖正文代替恢复。批准内容在执行期间固定，不受后续候选／Draft 编辑影响。Git 失败和内容应用失败分开，不因提交未完成再次应用已批准内容；升级须恢复 P2 MVP 已接受回执。

成功消费或明确取消后清理 `70_ChangeSets/<id>`；必要 Job 结果保留，未明确结束的批准操作保留至结果清楚。备份应覆盖 Vault 的未完成候选与 Core 执行回执；仅重建数据库不能代替两者。正式历史交给 Git，不建立永久完整 ChangeSet 档案。

## 6. D：Git、清理与真实使用闭环

Git 跟踪 `10_Ideas`、`40_Knowledge`、`50_Research`、`60_Projects`，`20_Sources` 只跟踪 Record Markdown，`90_System` 只跟踪选定用户配置。独立 Asset、Derived Representation、`30_Drafts`、`70_ChangeSets` 和 `00_Inbox` 不跟踪。

批准成功应用后，Core 只提交本次涉及的受跟踪文件最终版本，包括其中此前未提交的用户修改；保留其他文件及暂存区中无关改动。快捷新建也自动提交。规划与审查使用当前本地内容，不强制先 commit，不以最近 Git 版本替代未提交文件。用户自己的编辑可手动或按设置定时提交。

复用 Source／Draft 标记、恢复及引用预览，补足正式文件清理缺口。默认跳过被 active 正式内容引用的 Source，用户可经引用提示明确覆盖；外部 Zotero Asset 永不删除。不能仅按目录推断独立 Asset 归属。

真实场景覆盖论文选段融入已有研究结论、同一 Draft 贡献多份笔记、MVP 旧笔记参与重组、规划期间本地编辑后人工合成、打回重规划、快捷新建与限定范围 Git。检索能找回已应用内容和 Source，候选副本不会混入正式召回；模型关系仍由用户决定。

## 7. 验证、施工顺序与交付

按 A 中的 MergeView 可行性切片 → Core 候选与工具权限 → 真实 Agent Planner → BC 候选审查／统一批准执行／恢复 → D Git／真实闭环推进。B/C 合并交付，工程内部仍分小切片和检查点；BC 不提前包含 D 的自动 Git。各切片随当前代码保护持久化与权限行为，不先搭完整 Agent 平台。早期切片成功不等于完整 P3 完成。

使用现有测试栈，覆盖候选路径隔离与越权拒绝、当前选择／Intent、Candidate 保存和刷新、未批准不写正式文件、执行前置条件、归档、临时计划清理、旧回执升级、Git 文件／暂存范围及异常恢复。模型输出合法不等于重组合理；用真实材料核对遗漏条件、原意、用户理解和过时关系。Agent 工具循环须有实际读取、检索和候选写入证据，不能只验证 JSON 格式。

验收以 Agent Planner 为必需路径，覆盖 Knowledge／Research、统一重组及快捷新建；不要求新增 Planner 模型 API 路径。P2 现有 API 仅做受影响兼容回归，不重新做完整双路径效果验收。隔离测试和现场记录放各仓库 `.local/p3/`，维护性文档只保存长期合同及复现方法。

阶段出口为总体 P3-G：已有知识可受控重组，论文／网页能入库并再次找回，Obsidian 内完成候选编辑与批准，两类入口共用执行和 Git，恢复不丢修改、不重复应用。高级检索、维护、模型 API 扩展与复杂 Agent 继续按真实需要后续分期。
