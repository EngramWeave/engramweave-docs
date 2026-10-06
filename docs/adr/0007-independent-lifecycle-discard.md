# Discard is a reversible lifecycle mark, not a processing stage

Source, Draft, and formal knowledge file Properties store `lifecycle_status: active | discarded`, with rebuildable Core projections. Discard preserves the previous processing stage and lets users restore or batch-clean marked files. Before archival, discarding a Source cascades to related Draft and ChangeSet; after archival, marking referenced formal knowledge is an explicit user choice.

Keeping discard independent avoids losing whether material was pending, compiled, reviewed, planned, or archived. Successful integration automatically marks related Drafts discarded rather than deleting user edits immediately. Physical cleanup, restoration cascades, and formal-content confirmation require their own explicit rules.
