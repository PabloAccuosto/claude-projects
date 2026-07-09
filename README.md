# claude-projects

Pablo's personal Claude project pattern and standing preferences —
for projects that live under his own GitHub account rather than an
organisation with its own equivalent setup (e.g. `Eticas-AI/ai-ops`).

## What's here

- **`claude-personal-preferences.md`** — Pablo's working preferences,
  loaded at the start of every session where he's the contributor.
- **`claude-project-pattern.md`** — the conventions in detail: why
  projects are organised this way, what each piece does. Read when
  setting up a new project or migrating an existing one.
- **`instructions-common.md`** — what Claude reads at the start of
  every personal project session. Operational reference, imperative
  form.
- **`migration-runbook.md`** — the step-by-step playbook for
  bootstrapping a new project (the common case) or bringing in
  existing material. Just point Claude at it to start.

## Starting a new project

Say something like:

> Nuevo proyecto. Leé `https://github.com/PabloAccuosto/claude-projects/blob/main/migration-runbook.md`
> y guiame.

The runbook handles the rest through a few questions.

## Provenance

`claude-project-pattern.md`, `instructions-common.md`, and
`migration-runbook.md` are adapted from Eticas' `Eticas-AI/ai-ops`
pattern, generalised for solo personal use (no multi-contributor
resolution, no org-specific transversal repos, generic shape names).
When useful refinements land on either side, they're worth
cross-checking against the other.
