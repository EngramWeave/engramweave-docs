# EngramWeave Context Map

Shared semantics and cross-component decisions belong in `doc/`. Component contexts and implementation decisions belong in their respective repositories. Temporary review records and validation evidence do not belong in maintained context documents.

## System context

- [CONTEXT.md](CONTEXT.md): shared knowledge production, integration, provenance, and research semantics.
- [GLOSSARY.md](GLOSSARY.md): system domain definitions.
- [docs/adr/](docs/adr/): significant cross-component decisions.
- [Overall design v0.4](个人知识编译系统总体设计方案_v0.4.md): architecture baseline.

## Component ownership

| Context | Repository | Responsibility and existing documentation |
|---|---|---|
| Core / Desktop | [engramweave](../engramweave/) | Core contracts, persistence, jobs, Desktop hosting, and implementation decisions; [P1 contracts](../engramweave/docs/p1-contracts.md) |
| Obsidian | [engramweave-obsidian](../engramweave-obsidian/) | Native editing and workflow sidebar integration |
| Zotero | [engramweave-zotero](../engramweave-zotero/) | Reading, selected material submission, and original locations; [component context](../engramweave-zotero/CONTEXT.md) |
| Web Clipper | [engramweave-web-clipper](../engramweave-web-clipper/) | Web capture and Source protocol adaptation |
| Mobile | [engramweave-mobile](../engramweave-mobile/) | Mobile capture and client access |

Component documents reference system semantics rather than redefining Compiler, Source, Draft, or approval boundaries. Supported phase scope may differ from the product goal, but the distinction must be explicit.
