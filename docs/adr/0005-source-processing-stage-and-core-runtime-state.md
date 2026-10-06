# Source owns processing stage; Core owns registration and execution state

Source Record Properties store `processing_status` as `pending / compiled / reviewed / planned / archived / failed / discarded`, and Core keeps a rebuildable projection. Core separately owns `registration_status` (`ready / invalid / missing / unsupported`) and `job_status` (`queued / running / succeeded / failed / interrupted`), plus errors, retry counts, and recompile counts.

This replaces the archival-only Source property boundary. Source Registry writes `pending` when the property is absent or empty, including for old material; missing markers may therefore cause recompilation. Intact Source stage properties survive database rebuilding, while damaged files are restored from file history or backups. Temporary execution details remain outside the Vault.
