# Plan Documents

LLM agents must create a Markdown plan document **in this directory** (`doc/plan/`) before porting any feature/bugfix from GTK, or performing a Windows-specific bugfix, named with this pattern:

* `2025-12-25-port-plaintext-indexing.md`
* `2025-12-26-fix-preferences-layout-bug.md`
* `2026-01-31-r-extract-behavior-from-controllers-to-models.md` ('r' for 'refactoring')
* `2025-12-25-1-zap-old-indexing.md` (ordered chronologically)
* `2025-12-25-2-add-gui-for-port-setting.md`

Before creating the plan, read these documents: 

* [@README.md](../../README.md)
* [@AGENTS.md](../../AGENTS.md)
* [doc/design/DESIGN.md](../design/DESIGN.md)
* Architecture notes from the [kaya-gtk](https://github.com/lofimx/kaya-gtk) repo's `doc/arch/*.md` (for porting context)

Then create the plan **in `doc/plan/`** and ask any clarifying questions you have. After I answer your questions to eliminate ambiguity, adjust the plan accordingly. Ask more clarifying questions, if required, and repeat the process until all your questions are answered.

**Important:** The plan file must be written to `doc/plan/` at the start of work, not to any tool-internal or temporary location. If your tooling uses a separate plan file path, copy or write the plan to `doc/plan/` as the first step. The plan document in this directory is the canonical record of what was done and why.
