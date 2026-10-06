# EngramWeave System Context

This document clarifies the product semantics of the [overall design v0.4](个人知识编译系统总体设计方案_v0.4.md). Definitions belong in [GLOSSARY.md](GLOSSARY.md); ownership is mapped in [CONTEXT-MAP.md](CONTEXT-MAP.md). Phase scope and implementation proposals are not automatically requirements of the overall design.

## Compilation scope

- Users submit material they have already judged worth preserving. Compiler denoises and lightly refines it while respecting its content; it does not discover additional knowledge in unsubmitted material.
- Paper submissions consist primarily of selected passages, highlights, and comments. Full text may provide reference context but does not expand the material to be compiled.
- One submission produces one Draft, even when it includes multiple themes. Integration Planner proposes final creation, update, splitting, or merging through ChangeSet.
- Body compilation uses submitted content and Annotation, returns a title and body, and does not combine knowledge-library analysis or supplementary commentary into that generation. Existing note references in submitted material may be retained. Users can add explanations during Review.
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
- Compilation selects the `pending` stage. A processing round may also handle `reviewed` Sources by creating integration proposals. Other stages are not treated as uncompiled merely because they lack an archival marker.
- Sending content back for Recompile returns it to `pending`. Existing Draft edits and versions must remain recoverable.
- Recompile only returns the Source to `pending`; it does not immediately invoke a model. Processing waits for the next scheduled run or a manually started batch. Ordinary Source edits do not themselves request Recompile.
- Desktop may offer a recompile filter derived from the Core-held recompile count, allowing separate batch selection of recompile and first-compile material.

## Independent stage, lifecycle, registration, and execution state

- `processing_status` describes the content processing stage: `pending / compiled / reviewed / planned / archived`. It is stored in Source Record Properties. Core stores a rebuildable projection. `planned` means an inspectable ChangeSet exists, awaiting approval and execution.
- `lifecycle_status` is independent: `active / discarded`. Source, Draft, and formal knowledge file Properties store it, with a rebuildable Core projection. Discarding preserves the previous processing stage; failure does not add a processing stage.
- `registration_status` describes whether the file exists, is valid, and is supported: `ready / invalid / missing / unsupported`. It is stored in Core.
- `job_status` describes one execution attempt: `queued / running / succeeded / failed / interrupted`. It is stored in Core independently of the content stage.
- Task error details, retry counts, and recompile counts are Core data, not Source Properties.
- A processing round re-reads the current stage. Eligible Sources are pending or reviewed, valid and supported, active, and without an ongoing task. Job ownership prevents duplicate processing.

Source Registry adds `processing_status: pending` when the property is absent or empty. This applies to old Sources as well: missing stage information makes them eligible for processing after registration. Users accept recompilation caused by a missing marker; damaged knowledge files should be restored from file history or backups rather than silently inferring an authoritative stage from the database.

Database reconstruction reads the stage from intact Source Properties; it does not itself erase the property or reset the stage to pending. This contract extends P1, whose implementation only supports the read-only archival value.

Failed tasks preserve the original Source stage. An inspectable plan does not imply approval; handling planned work must retain existing approval, conflict, and idempotent execution rules.

See [the status ownership decision](docs/adr/0005-source-processing-stage-and-core-runtime-state.md).

## Failure and retry behavior

- Temporary failures permit finite configurable retries within the current round.
- An explicit error or exhausted retries ends that round and records its error without overwriting the Source stage. Users can inspect failed round history.
- A subsequent round may retry a Source that still satisfies the eligibility rules. Rebuilding Core can likewise permit a new attempt based on the intact Source stage.
- A completed Draft is retained when relationship analysis fails. Retry only the failed relationship step, not body compilation.
- Failed Review or Relation jobs do not change `compiled` or block Review Complete. The sidebar shows their failure, and users may select a batch of Drafts with failed Draft Analyzer tasks for reanalysis.

## Lifecycle discard and Draft cleanup

- Discard is a reversible, trash-like mark. Desktop can mark selected Sources discarded and let users filter discarded files for batch physical deletion.
- Discarding a Source stops subsequent processing while retaining its material and stage. Users can restore it to active.
- Before archival, discarding a Source also marks its related Draft and ChangeSet discarded. ChangeSet is a control-plane record; this does not imply adding a knowledge file for it.
- For an archived Source, Core checks for formal knowledge references and prompts the user to choose whether those knowledge files should also be marked discarded. This is not an automatic cascade to formal content.
- Old Drafts and user edits survive Recompile. Successful integration automatically marks related Drafts discarded for user-managed cleanup; Source, Annotation, and formal content remain unchanged apart from the required archival stage update.
- The confirmation mechanism for formal-content discard/deletion, dependency-sensitive deletion, and restoration cascades needs explicit rules. A discard mark is not an implicit authorization to delete referenced files.

## Compiler and Draft Analyzer

Body compilation returns title and body. Draft Analyzer is Core orchestration of two independent sub-tasks, Review Analyzer followed by Relation Analyzer. Each has selectable analysis templates, models, and execution paths, stores its result separately, and is bound to the same Source and Draft version. All analysis outputs belong only in the sidebar and cannot modify body content.

| Analysis input | Purpose |
|---|---|
| Draft | Focus suggestions on the actual content being prepared for integration, reducing original noise. |
| Source including Annotation | Check omitted conditions, changed meaning, or conclusions absent from the submitted material. |
| Relevant knowledge-library content | Discover connections, conflicts, and duplication involving existing Knowledge, Ideas, and Research. |

- Review Analyzer uses material-specific templates. Knowledge review may check omissions or changed meaning, question understanding, identify errors, or summarize content. Academic review may examine suspected paper errors, statistical scope, and claims to verify. It also retrieves relevant library context.
- Relation Analyzer uses templates to suggest existing knowledge links, conflicts, integration, and merging. Knowledge-oriented analysis may emphasize the library; academic analysis may emphasize Research, new viewpoints, and connections to earlier conclusions.
- Each sub-task obtains the context its templates need. Relation may reuse the complete Review input context, reuse Review output as a reference, or reuse neither. Context reuse aims to improve possible prompt-cache reuse; output reuse does not turn Review suggestions into approved knowledge.
- Context/result reuse remains bound to the Source and Draft versions. Rules for changed templates, models, and other retrieved files must be explicit rather than silently treating older analysis as current.
- An Analysis Profile combines selected Review/Relation templates, models, and execution paths into an analysis scheme. Capture can offer convenient selection of template presets.
- API execution: Core prepares context from templates and calls the two analysis models separately. Initial API execution has no dynamic tool loop.
- Agent execution: reuse the existing Agent runner loop to call Core Tools/MCP as needed. A Core-built tool loop is deferred until a concrete need arises.

See [the separate generation decision](docs/adr/0006-separate-body-and-supplementary-generation.md).

## Integration Planner reuse

Planner can reuse Relation templates and tool capabilities, but its input is the final Reviewed Draft and user Integration Intent. It may run another Relation analysis to revalidate relationships. Planner remains responsible for the final integration proposal and ChangeSet; reuse does not authorize stale relationships or direct analysis writes.

## AI execution and task models

- Direct inference and an early Codex execution entry are both in the intended initial scope. Codex access should use the user's available Codex entitlement; it must not be assumed to require a separately billed API key.
- Agent analysis can use the existing runner's tool loop through Core Tools/MCP. Complex research, cross-note maintenance, and broader orchestration remain separate extensions; an early analysis integration does not authorize direct Agent changes to approved content.
- Model selection is configurable by task, including compilation, AI supplementary information, relationships, integration planning, complex research, and cross-note maintenance. Separate instructions and task semantics remain intact.
- Provider and Worker execution protocols remain distinct even if both present a simple input/result interface to the workflow.

## Documentation ownership

Cross-component semantics, architecture, and system decisions belong in `doc/`. Core/Desktop, Obsidian, Zotero, Web Clipper, and Mobile implementation contexts and decisions belong in their respective repositories. System context should not freeze component-only implementation details.
