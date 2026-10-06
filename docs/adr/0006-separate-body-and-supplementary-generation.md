# Separate body compilation from supplementary analysis

Body compilation organizes submitted content with Annotation and returns only title and body. A separate, independently configured model reads Source, the generated Draft, and relevant Knowledge, Ideas, and Research to produce questions, suspected errors, and supplementary suggestions for the sidebar. That analysis cannot modify the body.

Separating the calls avoids asking one generation to both organize content and produce commentary that could leak into the body. Draft focuses analysis on the proposed content, Source permits fidelity checks, and existing library material permits connection, conflict, and duplicate discovery.
