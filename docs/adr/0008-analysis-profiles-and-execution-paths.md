# Analysis profiles compose templates and separate execution paths

An Analysis Profile combines Review/Relation templates, task models, and API or Agent routes. Capture can select presets. Core prepares template-driven context for separate API calls without implementing a dynamic API tool loop initially; Agent analysis reuses the existing runner loop with Core Tools/MCP. Core-built tool orchestration is deferred until needed.

This supports material-specific analysis and existing Agent capabilities without turning Compiler into a general-purpose analyzer or introducing another Agent loop. Analysis writes only sidebar results. Planner can reuse Relation templates and tools but plans from the final Reviewed Draft and user intent, with relationship revalidation when needed.
