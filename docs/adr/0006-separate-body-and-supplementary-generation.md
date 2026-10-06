# Separate body compilation from supplementary analysis

Knowledge Compiler comprises Compiler and Draft Analyzer. Compiler organizes submitted content with Annotation and returns only title and body. Draft Analyzer then orchestrates Review Analyzer followed by Relation Analyzer, each configured through templates, a model, and an API or Agent path. Both analyze the same Source and Draft versions and store independent sidebar-only results. Human Review is the subsequent user editing and approval stage, distinct from Review Analyzer.

Separating tasks avoids asking body generation to produce commentary that could leak into the body. Review templates focus content checking; Relation templates focus connections and integration clues. An Analysis Profile combines the two. Context or output reuse is selectable, and Planner can reuse Relation capabilities while retaining responsibility for its final ChangeSet.
