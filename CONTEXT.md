# EngramWeave System Context

This document clarifies the product semantics of the [overall design v0.4](个人知识编译系统总体设计方案_v0.4.md). Definitions belong in [GLOSSARY.md](GLOSSARY.md); ownership is mapped in [CONTEXT-MAP.md](CONTEXT-MAP.md). Phase scope and implementation proposals are not automatically requirements of the overall design.

## Compilation scope

- Knowledge Compiler is the complete pre-human workflow: Compiler creates the body, then Draft Analyzer runs Review Analyzer and Relation Analyzer. Human Review starts after this workflow completes; Review Analyzer is AI analysis, not human approval.
- Users submit material they have already judged worth preserving. Compiler denoises and lightly refines it while respecting its content; it does not discover additional knowledge in unsubmitted material.
- Paper submissions consist primarily of selected passages, highlights, and comments. Full text may provide reference context but does not expand the material to be compiled.
- Source in this workflow primarily denotes the Source Record with Properties. An unarchived Source Record may have multiple Drafts. Only one Draft is selected for final integration; after successful archival, all related Drafts are marked discarded. Recompile preserves existing Drafts and user edits. Integration Planner may integrate that one candidate into multiple formal files, all referencing the same Source Record.
- Body compilation uses submitted content and Annotation, returns a title and body, and does not combine knowledge-library analysis or supplementary commentary into that generation. Existing note references in submitted material may be retained. Users can add explanations during Review.
- Uncertainty, suspected errors, statistical issues, claims to verify, relationships, conflicts, and integration suggestions are Review Metadata displayed in the Obsidian sidebar, not Draft body content.
- Draft is an intermediate before integration. Long-term reading and retrieval use the user's approved integrated content.

See [the compilation scope decision](docs/adr/0001-user-defined-compile-scope.md).

## Optional compilation intensity

Already organized material may need little processing; long AI conversations may need stronger denoising and extraction. Capture-time mode or intensity selection is an optional future direction, not a required current feature. Intensity must not expand the submitted scope.

## Relationship discovery

Relationship discovery should recognize meaningful links across different expressions and cover relevant Canonical Knowledge, Ideas, and Research. Phase coverage may be limited, but its limits must be explicit. Missing keyword matches or uncovered candidates do not establish that no relationship exists.

The first useful release includes basic semantic recall across Knowledge, Ideas, and Research. Limited recall coverage is acceptable; keyword and explicit-link candidates alone do not satisfy that release scope. Advanced relationship retrieval can follow later.

## Content placement and research

- `40_Knowledge` holds approved, independently useful concepts, principles, and methods.
- `50_Research` holds results, conditions, evidence, and analysis tied to particular papers, experiments, or research questions.
- Placement follows content purpose, not whether the original material is a paper.
- AI-generated research content uses Draft → Review → ChangeSet and receives the same protection against unapproved AI changes. Research placement alone does not establish user approval.
- Users generally create research topic folders. Integration Planner reuses that structure first and may propose Create New Folder when needed. Folder creation goes through ChangeSet; a new top-level Domain requires explicit approval.
- A Research Question is an ordinary Markdown note under a research topic. Other materials link to it and may use a `research_question` property containing its note link. No separate application-managed question entity is required.

## Submission identity and archival

- Each explicit submission creates an independent Source. Multiple Sources can share a paper reference while preserving their individual original locations.
- Markdown body remains a logical Source Asset even when stored in the same file as Source Record Properties. An inline capture has no separate Asset file/reference to delete; deleting the combined file also deletes that body.
- Capture-managed Assets under `20_Sources` belong to individual Source Records and are not shared between Records. Shared references are external Assets, outside EngramWeave's mutation/deletion authority. No managed-Asset reference-counting subsystem is required by this contract.
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
- Sending content back for Recompile returns it to `pending`. The next processing round may generate another Draft; existing Drafts, including user edits, remain recoverable. Recompile must not irrecoverably overwrite edited content.
- Recompile only returns the Source to `pending`; it does not immediately invoke a model. Processing waits for the next scheduled run or a manually started batch. Ordinary Source edits do not themselves request Recompile.
- Desktop may offer a recompile filter derived from the Core-held recompile count, allowing separate batch selection of recompile and first-compile material.

## Independent stage, lifecycle, registration, and execution state

- `processing_status` describes the content processing stage: `pending / compiled / reviewed / planned / archived`. It is stored in Source Record Properties. Core stores a rebuildable projection. `planned` means an inspectable ChangeSet exists, awaiting approval and execution.
- `lifecycle_status` is independent: `active / discarded`. Source, Draft, and formal knowledge file Properties store it, with a rebuildable Core projection. Discarding preserves the previous processing stage; failure does not add a processing stage.
- Rebuildable metadata and backlink projections may be updated per file for efficiency; they never replace fresh mutation preconditions. Windows file-operation protection belongs to Core and is available without Desktop through its compiled helper. Runtime file operations do not rely on PowerShell or dynamic compilation.
- `registration_status` describes whether the file exists, is valid, and is supported: `ready / invalid / missing / unsupported`. It is stored in Core.
- `job_status` describes one execution attempt: `queued / running / succeeded / failed / interrupted`. It is stored in Core independently of the content stage.
- Task error details, retry counts, and recompile counts are Core data, not Source Properties.
- A processing round re-reads the current stage. Eligible Sources are pending or reviewed, valid and supported, active, and without an ongoing task. Job ownership prevents duplicate processing.

Source Registry adds `processing_status: pending` and `lifecycle_status: active` when their respective properties are absent or empty. This applies to old Sources as well: missing stage information makes them eligible for processing after registration. Users accept recompilation caused by a missing marker; damaged knowledge files should be restored from file history or backups rather than silently inferring an authoritative stage from the database.

Database reconstruction reads the stage from intact Source Properties; it does not itself erase the property or reset the stage to pending. This contract extends P1's archival-only read boundary. Current registration support is documented in the [Core and Desktop context](../engramweave/CONTEXT.md).

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
- Discarding a Source stops subsequent processing while retaining its material and stage. Users can restore it to active. Users may separately discard explicitly selected active Drafts while keeping the Source and its stage unchanged; this does not physically delete any file.
- Before archival, discarding a Source also marks all its related Drafts discarded and stops further plan progress. ChangeSet is a temporary execution plan, not a lifecycle-marked file or a generic trash item; cancellation and unresolved execution follow the plan rules below.
- Whenever a user discards a Source, Core lists all Canonical Knowledge files referencing it. Users choose which, if any, to mark discarded. Marking does not authorize physical deletion, and integration success does not automatically discard Source or formal content.
- Old Drafts and user edits survive Recompile. Successful integration automatically marks related Drafts discarded for user-managed cleanup; Source, Annotation, and formal content remain unchanged apart from the required archival stage update.
- User-initiated file operations confirm an explicit target list. AI-proposed discard or deletion from Planner or Maintenance uses ChangeSet.
- Physical deletion is a separate user cleanup action for discarded material. Default batch cleanup skips Sources still referenced by active formal content. Users may explicitly select such a Source, review the reference warning/list, and confirm deletion; broken links are not automatically rewritten.
- Source Record deletion removes its associated Derived Representations and its Capture-owned local Asset under `20_Sources`. External references, including shared Zotero Assets, are not modified or deleted. Inline captures require no separate Asset deletion.
- Restoration defaults to Source only and lists related objects for user-selected restoration. Old Drafts automatically discarded after successful integration are not automatically reactivated. ChangeSets are not restored through this mechanism.

## Knowledge Compiler: Compiler and Draft Analyzer

Body compilation returns title and body. Desktop explicitly permits active pending or compiled Sources to run Compiler again, creating another Draft without changing the independent Recompile action. A failed repeat run retains the compiled stage. Draft retains the original captured_at and annotation property values. Compiler instructions are user-editable under 90_System/Prompts/Compiler.md and are frozen when a request is accepted. Draft Analyzer is Core orchestration of two independent sub-tasks, Review Analyzer followed by Relation Analyzer. Each has selectable analysis templates, models, and execution paths, stores its result separately, and is bound to the same Source and Draft version. All analysis outputs belong only in the sidebar and cannot modify body content.

The ordinary workflow exposes Human Review after the compilation/analysis round, with analysis failures shown when allowed. Human edits are not a normal interleaving between Review Analyzer and Relation Analyzer. Concurrent external file changes still require version checks rather than silent overwrites.

| Analysis input | Purpose |
|---|---|
| Draft | Focus suggestions on the actual content being prepared for integration, reducing original noise. |
| Source including Annotation | Check omitted conditions, changed meaning, or conclusions absent from the submitted material. |
| Relevant knowledge-library content | Discover connections, conflicts, and duplication involving existing Knowledge, Ideas, and Research. |

- Review Analyzer uses material-specific templates. Knowledge review may check omissions or changed meaning, question understanding, identify errors, or summarize content. Academic review may examine suspected paper errors, statistical scope, and claims to verify. It also retrieves relevant library context.
- Relation Analyzer uses templates to suggest existing knowledge links, conflicts, integration, and merging. Knowledge-oriented analysis may emphasize the library; academic analysis may emphasize Research, new viewpoints, and connections to earlier conclusions.
- Each sub-task obtains the context its templates need. Relation has three reuse choices: complete Review Analyzer input context, Review Analyzer output as reference, or neither its context nor output. This refers to AI analysis, not Human Review. Output reuse does not turn suggestions into approved knowledge.
- Context/result reuse remains bound to the Source and Draft versions. Rules for changed templates, models, and other retrieved files must be explicit rather than silently treating older analysis as current.
- An Analysis Profile combines selected Review/Relation templates, models, and execution paths into an analysis scheme. Users configure template definitions, models, and routes in Desktop beforehand. Capture only chooses which Review/Relation template presets to use, and the selection may be changed before processing. Execution uses the final selection and current configuration rather than a capture-time copy of the configuration.
- API execution: Core prepares context from templates and calls the two analysis models separately. Initial API execution has no dynamic tool loop.
- Agent execution: reuse the existing Agent runner loop to call Core Tools/MCP as needed. A Core-built tool loop is deferred until a concrete need arises.

See [the separate generation decision](docs/adr/0006-separate-body-and-supplementary-generation.md).

## Integration Planner reuse

Planner can reuse Relation templates and tool capabilities. At the start of each planning round, it reads the current reviewed Draft, user Integration Intent, and current local knowledge-library content, and fixes those inputs for that round. It may run another Relation analysis to revalidate relationships. Planner remains responsible for the integration proposal and ChangeSet; analysis reuse does not authorize direct writes.

The first release supports full knowledge reorganization through this single Planner. Creating, changing, splitting, merging, reorganizing, and marking files discarded describe possible outcomes, not separate product modes or workflows. Planner decides the required changes and produces one candidate ChangeSet for the common inspect/edit/reject/replan/approve flow. Low-level execution details do not constrain planning to four fixed business operations; physical deletion remains separately authorized cleanup after discard.

See [the unified integration decision](docs/adr/0012-unified-knowledge-reorganization.md).

## User-controlled review and planning

- Review Complete changes `compiled` to `reviewed` and permits planning. It does not bind approval to an exact Draft version. Draft remains editable; edits do not automatically revoke review, trigger replanning, or invalidate a generated ChangeSet.
- Before Planner starts, users may cancel Review Complete to return `reviewed` to `compiled`. Once a ChangeSet exists, canceling Review Complete cancels that unapproved candidate and returns `planned` to `compiled`; users can edit and confirm again to request a new plan.
- Later Draft edits do not change the inputs of a running planning round or the contents of an existing candidate. No real-time Draft watcher or persistent review-version binding is required by this workflow.
- During ChangeSet review, users compare candidate file contents against current local files and choose or edit both sides to form the final approved contents. Rejecting a candidate and requesting replanning makes Planner read current local content again.
- Generating, inspecting, editing, or rejecting an unapproved candidate does not execute it or modify formal knowledge files. Approval causes Core to apply the final approved contents immediately. A started execution follows that approval independently of later Draft edits; execution checks target-file preconditions.
- If a canceled or missing ChangeSet leaves a Source at `planned`, return it to `reviewed` when the reviewed Draft remains usable and wait for a new planning round. A new candidate requires new approval. Recovery after approved execution has begun is an implementation responsibility, not another user confirmation workflow.

See [the review and planning decision](docs/adr/0010-user-controlled-review-and-planning.md).

## Temporary ChangeSet consumption

- A generated ChangeSet is temporarily persisted for inspection and approval. Execution checks versions and preconditions and applies only approved operations.
- ChangeSet is a one-use, consumable plan, not a permanent knowledge asset or a generic recycle-bin object.
- Successfully consumed or explicitly canceled plans are cleaned up. Unfinished operations remain until their execution outcome is clear, preventing duplicate application.
- Successful execution cleans the plan and marks related Drafts discarded for user cleanup. Necessary Job results and error information remain; full successful ChangeSet copies need not be retained permanently.
- Formal knowledge version history belongs to Git. Cleanup must not discard the unfinished operation needed to recover a failed archival-marker write or an ambiguous execution result.

See [the temporary plan decision](docs/adr/0009-temporary-changeset-consumption.md).

## Vault Git scope and commits

- Git tracks `10_Ideas`, `40_Knowledge`, `50_Research`, and `60_Projects`; under `90_System`, only selected user configuration files such as Review/Relation analysis templates are tracked.
- Under `20_Sources`, Git tracks Source Record Markdown files only. Independent Source Assets, Derived Representations, `30_Drafts`, and `00_Inbox` are excluded to keep history bounded.
- After an approved ChangeSet is successfully applied, Core automatically commits the final versions of the tracked files involved in that integration. Earlier uncommitted user edits in those files are included; unrelated file changes are not committed.
- Users may commit their own edits manually or configure periodic automatic commits. This does not require a Git commit before every planning round or integration; planning and candidate comparison use current local files, not the last committed versions.
- Git commit failures and interrupted execution require recoverable implementation behavior without silently repeating approved content changes. Git is version history, not a second approval gate.

See [the scoped Git decision](docs/adr/0011-scoped-vault-git-history.md).

## AI execution and task models

- Direct inference and an early Codex execution entry are both in the intended initial scope. Codex access should use the user's available Codex entitlement; it must not be assumed to require a separately billed API key.
- Compiler, Review Analyzer, Relation Analyzer, and Integration Planner each support selection of API or Codex execution in the first useful release, with independently configured task models. Reuse minimal execution adapters rather than requiring the complete future Agent platform.
- Agent analysis can use the existing runner's tool loop through Core Tools/MCP. Complex research, cross-note maintenance, and broader orchestration remain separate extensions; an early analysis integration does not authorize direct Agent changes to approved content.
- Model selection is configurable by task, including compilation, AI supplementary information, relationships, integration planning, complex research, and cross-note maintenance. Separate instructions and task semantics remain intact.
- Provider and Worker execution protocols remain distinct even if both present a simple input/result interface to the workflow.

## Documentation ownership

Cross-component semantics, architecture, and system decisions belong in `engramweave-docs/`. Core/Desktop, Obsidian, Zotero, Web Clipper, and Mobile implementation contexts and decisions belong in their respective repositories. System context should not freeze component-only implementation details.

## Sources browsing

Source Health is displayed as available / missing / invalid / unsupported; the existing registration wire value ready means available. Sources shows six views: the first five exclude discarded Sources, while Discarded exclusively contains them. All Sources contains the remaining registered Sources, Pending selects pending, Processing selects compiled/reviewed/planned, Archived selects archived, and Issues selects missing/invalid/unsupported. The first five may overlap. Processing directly displays processing_status (pending/compiled/reviewed/planned/archived); Lifecycle directly displays lifecycle_status (active/discarded). Detailed processes and logs belong in their own pages. Failed executions do not add a Health issue. Missing or unreadable files may retain their last readable stage and lifecycle projection; these are last-known values, not proof that the current file is valid. Sources Inspector and Discard previews show active Drafts only. Batch controls occupy a stable bottom position, target confirmation uses a separate dialog, and execution feedback uses a transient toast with accessible per-item results.

Source filters combine dimensions with AND and categories within a dimension with OR. Tags come from Registry. Captured Time supports day and bounded date ranges. Desktop batch operations apply the existing individual action contract sequentially to explicitly selected Sources, with per-item outcomes and fresh eligibility checks.
