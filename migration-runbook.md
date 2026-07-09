# Bootstrap runbook (personal)

Operational playbook for applying the personal Claude project pattern
(`claude-project-pattern.md`) to either:

- a brand-new Claude project being bootstrapped from scratch (the
  common case — most personal projects start here), or
- a project with existing material to bring in — scattered
  conversations, an existing repo without a dedicated Claude project,
  notes, or files sitting in a folder somewhere.

The canonical pattern is the contract (what and why). This runbook is
the playbook (how, in practice).

> Adapted from Eticas' `Eticas-AI/ai-ops/migration-runbook.md`,
> trimmed for solo use: no multi-contributor work-log tie-ins, no
> mandatory SharePoint/M365 step, generic shape names instead of
> Eticas project codenames.

---

## Invoking this runbook

From any conversation, say something like:

> Nuevo proyecto. Leé `https://github.com/PabloAccuosto/claude-projects/blob/main/migration-runbook.md`
> y guiame.

or, if starting from an existing repo or existing conversations:

> Migración. Leé `https://github.com/PabloAccuosto/claude-projects/blob/main/migration-runbook.md`
> y guiame.

That's enough. Everything else is collected through questions.

---

## Boot sequence (Claude reads this section first)

1. Confirm in one sentence that the runbook is loaded. Don't
   summarise it back.
2. Ask the first sondeo question (Phase 1 below). Ask **one thing at
   a time** when context doesn't make the answer obvious. Don't
   present a structured form for Pablo to fill in — the whole point
   of this runbook is to avoid that.
3. Proceed through the phases in order. Don't jump ahead.
4. Throughout, follow `claude-personal-preferences.md` — direct
   recommendations over option lists, state assumptions inline, ask
   only when a decision genuinely needs Pablo's input.

---

## Phase 1 — Sondeo

Things to establish, in roughly this order. Infer when you can, ask
when you can't.

- **Project name.** Often inferable from the first message; confirm
  if uncertain.
- **Bootstrap vs migration.** Is there existing material (an existing
  repo, scattered conversations, notes, files) to bring in, or is
  this a project starting from nothing? If unclear, ask.
- **Repo target.** Propose a short, lowercase repo name. Default
  owner is `PabloAccuosto` unless Pablo says otherwise (e.g. a repo
  under a client's own org, or under an Eticas project — in which
  case this personal pattern probably isn't the right one; check
  whether `Eticas-AI/ai-ops` applies instead).
- **Existing repo, or to be created?** If it doesn't exist yet,
  Claude can create it directly via `GitHub MCP:create_repository`
  (personal account, private by default) — confirm the name and
  visibility, then create it as part of Phase 5. No need for Pablo
  to create it manually first.
- **Available connectors in this session.** GitHub MCP is the
  baseline requirement for Phase 5. If a PAT is going to be used
  (check the personal preferences file's `pat` setting), ask for it
  now rather than mid-bootstrap.

Move to Phase 2 only when the above are settled. **Skip straight to
"Bootstrap from scratch" at the bottom if there's no existing
material** — Phases 2–4 exist for the migration case.

---

## Phase 2 — Inventory (skip if bootstrap from scratch)

The goal is a complete picture of what currently exists about this
project, classified by destination in Phase 3.

Sources to sweep, in this order:

### 2a. Existing project knowledge (PK)

If there's an existing Claude project with PK, ask Pablo to list its
contents, or search PK directly with broad queries if there's tool
access.

### 2b. Project memory

Ask Pablo to paste the project memory dump from the old project's
side panel, or use it directly if it's already visible in the
system prompt. Flag explicitly: items in memory may be outdated —
a starting point, not ground truth.

### 2c. Past chats not reflected in memory

Use `conversation_search` and `recent_chats` to surface threads that
covered topics not in the memory dump. Don't be exhaustive — aim for
topics with current operational consequence.

### 2d. Reading source documents

Depends on what's available:

- **Documents already in a connected Drive/SharePoint.** Read
  directly via the relevant MCP tool.
- **Documents in PK of the project the migration session runs in.**
  Use the project knowledge search tool directly.
- **Documents elsewhere.** Ask Pablo to upload them to the chat —
  chat-attached files don't persist after the session, which is
  fine for a one-off migration read.
- **Nothing of the above.** Ask Pablo to paste relevant excerpts.
  Last resort; only viable for short documents.

**Do not put PK-bound files in the repo to make them readable.** Git
never fully forgets — even after deletion, files remain in history.
For anything sensitive, use the paths above instead.

---

## Phase 3 — Classification

Classify every inventory item into one of:

- **External artefact** — PDFs, reference material produced outside
  this workflow. Candidate for project knowledge (decided in
  Phase 4).
- **Internal synthesis** — summaries, distilled context, structural
  understanding produced during the work itself. Goes to the repo as
  `meta/CONTEXT.md` or similar.
- **Working draft / live state** — open decisions, action items,
  things in motion. Goes to `meta/STATE.md`.
- **Stable working pattern** — preferences, tone, habits. Don't
  transfer manually; the new project's memory regenerates this from
  use.
- **Discard** — obsolete, redundant, or already covered elsewhere.

For borderline items, ask. Don't over-decide.

---

## Phase 4 — Shape decision

Three shapes, adapted from patterns that recur across projects. Pick
one explicitly and say why.

### Minimal shape

- PK: empty.
- External sources (if any): referenced by URL/pointer only, not
  mirrored.
- Repo: everything that matters lives in `meta/`.
- Applies when: there's little or no external reference material, or
  it's rarely read in detail. Most personal projects start here.

### Reference-heavy shape

- PK: operational copies of the reference material that gets read at
  clause/section level with some frequency (specs, contracts,
  architecture docs).
- External source (if any): same artefacts also live there as
  canonical record.
- Repo: `docs/references/README.md` as an index pointer.
- Applies when: a handful of reference artefacts are central and get
  read closely on a regular basis.

### Hybrid shape

- PK: a small subset of deliverables — the ones worked at
  clause/section level.
- External source: canonical for everything else.
- Repo: `docs/references/` with a `README.md` indexing everything
  (flagging which are also in PK) and a summary per item.
- Applies when: there are many reference documents, only a subset is
  worked deeply, and the index itself is a useful navigation aid.

### How to choose

- Zero or one artefacts ever read closely → Minimal.
- A handful of artefacts read often → Reference-heavy.
- Many artefacts, with a deep-work subset → Hybrid.

If unclear, default to Minimal (the simplest) and elevate later if a
need emerges. Adding PK items is cheap; removing them mid-flight is
annoying.

### Per-document PK decision (Reference-heavy and Hybrid shapes)

For each external artefact, ask: *does this get read at
clause/section level with some frequency, or just at summary level?*

- Clause level → operational copy in PK.
- Summary level → external canonical only; summary in repo covers
  the operational need.

Walk through the list explicitly before Phase 5 commits the
bootstrap.

---

## Phase 5 — Bootstrap (Claude does this)

Produce, in this order, in as few commits as practical:

1. **Create the repo** if it doesn't exist yet (`GitHub
   MCP:create_repository`, personal account, private by default) —
   confirm name/visibility with Pablo first if not already settled
   in Phase 1.
2. **Repo skeleton.** `README.md`, `meta/STATE.md`, `meta/CONTEXT.md`
   (only if there's enough stable context to justify it — smaller
   projects can skip it and put context inline in `README.md`).
3. **Shape-specific docs folder**, if the shape isn't Minimal.
4. **Initial summaries**, if applicable. Use inline `[TBC: ...]`
   markers for anything that needs Pablo's confirmation rather than
   asking upfront.
5. **Instructions block.** Use the template in
   `claude-project-pattern.md` §5. Customise the project name,
   description, PK bullet, and any project-specific notes.
6. **Inventory registration**, if Pablo keeps a personal inventory of
   projects on this pattern — add or update a row.

---

## Phase 6 — Pablo executes

After Claude has committed the bootstrap:

1. **Create the new Claude project** (if not done already).
2. **Paste the instructions block** into the project's instructions
   field.
3. **Upload to PK** only the items Phase 4 says go there.
4. **Validate.** Open a session in the new project with a minimal
   prompt like "session start" or "what's the current state?".
   Verify Claude enters via `meta/STATE.md` and reads in the order
   specified.
5. **Resolve `[TBC: ...]` markers** in a follow-up session.
6. **Delete or archive the old project**, if there was one and it's
   confirmed the new one is operational.

---

## Working without GitHub MCP

If GitHub MCP isn't available in the session, Phase 5 can't commit
directly. Path:

1. Generate a single Artifact with a bash script that, when run
   locally, creates the full repo structure (`mkdir`s,
   `cat > file <<'EOF'` blocks, `git init`, `git add`, `git commit`).
2. Pablo runs it locally: `bash bootstrap.sh`.
3. Pablo creates the empty repo on GitHub via the web UI (or Claude
   does it via a PAT in a follow-up session).
4. Pablo adds the remote and pushes:
   `git remote add origin <url> && git push -u origin main`.
5. Subsequent updates use the same pattern, or Pablo edits files
   manually and commits.

If Code Execution & File Creation is available instead, Claude
creates the files in `/mnt/user-data/outputs/` and Pablo downloads
them. Equivalent result.

---

## Decisions crystallised (don't re-discuss)

- **Memory is not transferred manually.** Stable preferences
  regenerate from use; operational state lives in the repo.
- **`[TBC: ...]` markers go inline** in meta files, not as Q&A
  upfront — faster than a question round-trip per item.
- **The PK criterion is clause-level reads.** If an artefact isn't
  read at clause/section level with some frequency, it doesn't go in
  PK — a summary in the repo is enough.
- **PK files never go through git.** Not even temporarily. Git
  history retains them indefinitely.
- **DOCX vs PDF in PK** is Pablo's choice. What matters is no
  duplicates of the same document at different stages.

---

## Things to flag every time

- **Duplicate artefacts in PK.** Same document in two versions —
  keep only the canonical one in PK.
- **Structural ambiguities** in source material. Don't resolve
  unilaterally — surface and let Pablo decide.
- **Items that look obsolete.** Deprecated references, lists that
  have almost certainly changed. Default to discarding with an
  explicit flag so Pablo can override if relevant.
- **Sensitive items** the memory or past chats reveal. These stay in
  the chat, not in the repo.
- **Timeline anchors that cross date boundaries.** Confirm the
  specific date before writing anything with a date reference if the
  bootstrap session and the source events happened close together.

---

## Bootstrap from scratch (no migration)

The common case. If there's no source material:

- Skip Phase 2 (inventory) and Phase 3 (classification) entirely.
- In Phase 4, choose the shape based on what the project is expected
  to look like over time, not what already exists — default to
  Minimal unless there's a clear reason not to.
- In Phase 5, the meta files start empty or with a minimal seed
  paragraph — ask Pablo for one or two sentences on current focus
  and what counts as "done" for the project's immediate horizon.
- All other steps apply identically.

---

## Lessons learned

This runbook is meant to evolve. When a project surfaces a case not
covered above, append a short note here describing what was new and
how it was handled.
