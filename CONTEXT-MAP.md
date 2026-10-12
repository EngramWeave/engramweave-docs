# EngramWeave Context Map

Shared semantics and cross-component decisions belong in `engramweave-docs/`. Component contexts and implementation decisions belong in their respective repositories. Temporary review records and validation evidence do not belong in maintained context documents.

## System context

- [CONTEXT.md](CONTEXT.md): shared knowledge production, integration, provenance, and research semantics.
- [GLOSSARY.md](GLOSSARY.md): system domain definitions.
- [docs/adr/](docs/adr/): significant cross-component decisions.
- [Overall design v0.4](个人知识编译系统总体设计方案_v0.4.md): architecture baseline.
- [Implementation plan](IMPLEMENTATION_PLAN.md) and [TODO](TODO.md): delivery scope and phase tracking; P1 is complete, with the first useful paper-to-knowledge loop delivered through P2 and P3.
- [P2 plan](P2_IMPLEMENTATION_PLAN.md) and [P2 tasks](P2_TODO.md): completed development contracts and remaining human acceptance records.
- [P3 plan](P3_IMPLEMENTATION_PLAN.md) and [P3 tasks](P3_TODO.md): current Agent/MCP integration, Vault-local candidate review, approved execution and scoped Git scope.

- [P3 A plan](.local/P3/P3_A_IMPLEMENTATION_PLAN.md) and [A tasks](.local/P3/P3_A_TODO.md): local phase delivery tracking; maintained component contracts live in [Core/Desktop](../engramweave/docs/planning.md) and [Obsidian](../engramweave-obsidian/README.md).

- [P3 BC plan](.local/P3/P3_BC_IMPLEMENTATION_PLAN.md) and [BC tasks](.local/P3/P3_BC_TODO.md): implemented combined review, approval, deterministic application, archival and recovery; human acceptance remains separately recorded. Internal slices do not constitute separate B/C releases; scoped Git remains D.

- [Desktop resource-page design](.local/P3/P3_DESKTOP_REVIEW_CHANGES_DESIGN.md): Draft and live ChangeSet browsing, existing safe management, the confirmed sibling-cleanup prompt, and standalone lifecycle scope deferred to D. Implementation is not yet delivered.

## Component ownership

| Context | Repository | Responsibility and existing documentation |
|---|---|---|
| Core / Desktop | [engramweave](../engramweave/) | [Component context](../engramweave/CONTEXT.md); Core contracts, task-scoped Tools/MCP, candidate workspace validation/lifecycle, approved execution, persistence, jobs/rounds, scheduling, Desktop hosting, and implementation decisions; [P1 contracts](../engramweave/docs/p1-contracts.md), [Processing](../engramweave/docs/processing.md) |
| Obsidian | [engramweave-obsidian](../engramweave-obsidian/) | Native Draft editing, workflow sidebar and Current/Candidate ChangeSet review; [component context](../engramweave-obsidian/CONTEXT.md) |
| Zotero | [engramweave-zotero](../engramweave-zotero/) | Reading, selected material submission, and original locations; [component context](../engramweave-zotero/CONTEXT.md) |
| Web Clipper | [engramweave-web-clipper](../engramweave-web-clipper/) | Web capture and Source protocol adaptation |
| Mobile | [engramweave-mobile](../engramweave-mobile/) | Mobile capture and client access |

Component documents reference system semantics rather than redefining Compiler, Source, Draft, or approval boundaries. Supported phase scope may differ from the product goal, but the distinction must be explicit.
