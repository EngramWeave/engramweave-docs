# EngramWeave 上下文归属

总体语义与跨组件决策维护在 `doc/`；组件专属上下文和实现决策维护在各自仓库。术语定义与行为约束分别维护，临时审阅记录及验证证据不进入这些维护性文档。

## 总体上下文

- [CONTEXT.md](CONTEXT.md)：知识生产、整合、来源与科研组织的共同语义。
- [GLOSSARY.md](GLOSSARY.md)：总体领域术语。
- [adr/](adr/)：跨组件的重要取舍。
- [总体设计 v0.4](个人知识编译系统总体设计方案_v0.4.md)：系统架构基线。

## 组件归属

| 组件上下文 | 所属仓库 | 文档职责 |
|---|---|---|
| Core / Desktop | [engramweave](../engramweave/) | Core 合同、持久化、任务、Desktop 宿主与组件实现决策；已有 P1 合同见 [p1-contracts.md](../engramweave/docs/p1-contracts.md) |
| Obsidian | [engramweave-obsidian](../engramweave-obsidian/) | 原生编辑与工作流侧边栏集成 |
| Zotero | [engramweave-zotero](../engramweave-zotero/) | 论文阅读、选段投递与原材料定位集成 |
| Web Clipper | [engramweave-web-clipper](../engramweave-web-clipper/) | 网页采集和 Source 协议适配 |
| Mobile | [engramweave-mobile](../engramweave-mobile/) | 移动采集和客户端接入 |

组件文档引用总体上下文，不另行定义相互冲突的 Compiler、Source、Draft 或人工确认语义。阶段支持范围与实现限制可以不同，但应明确区别于总体产品目标。
