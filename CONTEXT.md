# EngramWeave System Context

This document clarifies the product semantics of the [overall design v0.4](个人知识编译系统总体设计方案_v0.4.md). Definitions belong in [GLOSSARY.md](GLOSSARY.md); ownership is mapped in [CONTEXT-MAP.md](CONTEXT-MAP.md). Phase scope and implementation proposals are not automatically requirements of the overall design.

## Compilation scope

- Users submit material they have already judged worth preserving. Compiler denoises and lightly refines it while respecting its content; it does not discover additional knowledge in unsubmitted material.
- Paper submissions consist primarily of selected passages, highlights, and comments. Full text may provide reference context but does not expand the material to be compiled.
- One submission produces one Draft, even when it includes multiple themes. Integration Planner proposes final creation, update, splitting, or merging through ChangeSet.
- Existing notes may be referenced. Compiler does not add unsolicited explanations, teaching, or knowledge outside the submitted scope. Users can add explanations during Review.
- Uncertainty, suspected errors, statistical issues, claims to verify, relationships, conflicts, and integration suggestions are Review Metadata displayed in the Obsidian sidebar, not Draft body content.
- Draft is an intermediate before integration. Long-term reading and retrieval use the user's approved integrated content.

See [the compilation scope decision](docs/adr/0001-user-defined-compile-scope.md).

## Optional compilation intensity

Already organized material may need little processing; long AI conversations may need stronger denoising and extraction. Capture-time mode or intensity selection is an optional future direction, not a required current feature. Intensity must not expand the submitted scope.

## Relationship discovery

Relationship discovery should recognize meaningful links across different expressions and cover relevant Canonical Knowledge, Ideas, and Research. Phase coverage may be limited, but its limits must be explicit. Missing keyword matches or uncovered candidates do not establish that no relationship exists.

## Content placement and research

- `40_Knowledge` holds approved, independently useful concepts, principles, and methods.
- `50_Research` holds results, conditions, evidence, and analysis tied to particular papers, experiments, or research questions.
- Placement follows content purpose, not whether the original material is a paper.
- AI-generated research content uses Draft → Review → ChangeSet and receives the same protection against unapproved AI changes. Research placement alone does not establish user approval.
- Users generally create research topic folders. Integration Planner reuses that structure first and may propose Create New Folder when needed. Folder creation goes through ChangeSet; a new top-level Domain requires explicit approval.
- A Research Question is an ordinary Markdown note under a research topic. Other materials link to it and may use a `research_question` property containing its note link. No separate application-managed question entity is required.

## Submission identity and archival

- Each explicit submission creates an independent Source. Multiple Sources can share a paper reference while preserving their individual original locations.
- Archival records successful controlled integration of the final content produced from that submission, not completion of the entire paper or other passages.
- Existing completion conditions apply, including successful approved operations and successful archival marker persistence.
- Retried delivery and deliberate new submissions must be distinguished; a shared paper locator is insufficient for deduplication.

## Zotero capture and Source independence

- The planned Zotero plugin lets users explicitly select passages or a group of highlights and comments and submit them to EngramWeave. One operation creates one Source with paper and location references. Other highlights and full text are not automatically submitted.
- A submitted Source is independent of later Zotero highlight or comment edits. Its connection to the original material is the locator link, not synchronization.
- Users submit new material as a new Source. Old Sources remain unchanged unless users explicitly edit or delete them.
- Component-specific semantics are maintained in [the Zotero context](../engramweave-zotero/CONTEXT.md).

## Comments and personal understanding

- Comments are retained in Source Annotation as long-term context.
- Personal understanding explicitly submitted for preservation can also be merged directly into Draft. The body does not require a separate user-opinion section or attribution label; the library represents the user's understanding.
- Keeping Annotation and including that understanding in Draft coexist. Compilation does not remove or relocate the original comments.
- Compiler need not classify original material and Annotation into source facts versus user judgments. Integrated content links to Source for verification.
- Suspected errors in the user's understanding belong in AI supplementary information. Compiler does not silently replace that understanding or substitute its judgment for Review.

See [the user understanding decision](docs/adr/0004-user-understanding-in-compiled-content.md).

## Compilation triggers

- Capture saves Source first; submission does not immediately compile it.
- EngramWeave compiles unprocessed Sources together on a configured schedule. Users can also select Sources in Desktop for immediate compilation.
- Scheduled compilation belongs to the ordinary knowledge workflow and does not inherently depend on an Agent Worker.
- Schedules support a specific execution time or an interval. Missed runs are not replayed on startup; users may manually start a batch after launch.
- Scheduled compilation selects only the `pending` (awaiting compilation) stage. It does not select compiled, reviewed, archived, invalid, or deleted/unavailable Sources merely because they lack an archival marker.
- Sending content back for Recompile returns it to `pending`. Existing Draft edits and versions must remain recoverable.
- Ordinary Source edits do not themselves request Recompile. Whether a Recompile action immediately executes or only returns the Source to the scheduled queue is a separate behavior decision.

## Workflow stage visibility

Each processing phase must have an explicit visible stage. `pending` names the stage eligible for scheduled compilation; `compiled`, `reviewed`, and `archived` distinguish Draft production, accepted body, and completed integration. Invalid or deleted/unavailable Sources are excluded from scheduled compilation.

The persistence location for these stages is not yet fixed. The existing design reserves Source Record `processing_status` for the durable `archived` attribute and keeps workflow state in Core; extending that file property to include intermediate stages would change this boundary. A visible stage model must not be mistaken for an already approved file schema change.

## Failure and retry behavior

- Temporary failures allow a finite configurable number of retries.
- Explicit errors or exhausted retries wait for manual action.
- A completed Draft is retained when relationship analysis fails. Retry only the failed relationship step, not body compilation.

## AI execution and task models

- Direct inference and an early Codex execution entry are both in the intended initial scope. Codex access should use the user's available Codex entitlement; it must not be assumed to require a separately billed API key.
- Early Codex integration can serve bounded request/result tasks. Complex research, cross-note maintenance, and broader Agent orchestration can be added later; an early integration does not authorize direct Agent changes to approved content.
- Model selection is configurable by task, including compilation, AI supplementary information, relationships, integration planning, complex research, and cross-note maintenance. Separate instructions and task semantics remain intact.
- Provider and Worker execution protocols remain distinct even if both present a simple input/result interface to the workflow.

## Documentation ownership

Cross-component semantics, architecture, and system decisions belong in `doc/`. Core/Desktop, Obsidian, Zotero, Web Clipper, and Mobile implementation contexts and decisions belong in their respective repositories. System context should not freeze component-only implementation details.
