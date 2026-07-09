# Claude project pattern (personal)

How Pablo works with Claude across his personal GitHub projects. The
pattern defines the contract between Claude, project knowledge, and
GitHub repos — what Claude reads, where state lives, how changes are
made. Repository structure beyond the contract is left to each
project.

This document is the canonical source for Pablo's personal projects.
Each project's instructions should reference it by URL; they should
not copy its contents. When a project surfaces a useful refinement,
it is folded back here.

> Adapted from Eticas' `ai-ops` pattern
> (`Eticas-AI/ai-ops/claude-project-pattern.md`), generalised for
> solo personal use under Pablo's own GitHub account. Eticas projects
> keep pointing at the Eticas original — this file is for projects
> that live outside that org, such as personal or exploratory work.

---

## 1. The contract

Every project that follows this pattern guarantees five invariants:

**1. A known entry point.** Each project declares, in its Claude
project instructions, which file(s) Claude should read at the start
of each session. The list is short, ordered, and stable. There is
always at least one entry-point file at a known path.

**2. Source-of-truth hierarchy.** Three locations, with clear roles:

- **Project repo** — single source of truth for state, code, and
  project-specific synthesis. Accessed via GitHub MCP only, never via
  project knowledge GitHub sync.
- **Project knowledge** — artefacts that don't belong in version
  control (signed contracts, NDAs, architecture PDFs, large reference
  documents, email correspondence).
- **Other reference repos** — when a project depends on material you
  maintain elsewhere (a shared template, a personal knowledge base,
  a taxonomy), the canonical source is always the dedicated repo,
  read via MCP at the moment it is needed.

If the same content appears in more than one location, the canonical
location wins. Contradictions are flagged for cleanup.

**3. End-of-session protocol.** Claude proposes deltas to the
relevant files at session close. Pablo reviews and commits. Claude
does not commit autonomously.

**4. Commit and branching conventions.**

- Commit message convention for state files: `meta: <what changed>`.
  Other commits follow the project's own conventions.
- Default branching strategy: feature branch + PR review for changes
  that are large, risky, or benefit from a diff view. Routine small
  updates can go directly to `main` if the personal preferences file
  permits (currently: yes, for routine state-file updates; branch
  for repo structure, project instructions, or this pattern itself).

**5. MCP access only.** The project repo is reached through GitHub
MCP. The "Add content from GitHub" sync into project knowledge is
not used — it creates a parallel access path that drifts silently
from the repo.

These five points are non-negotiable. Everything else in this
document — file names, folder structures, which auxiliary files a
project keeps — is guidance.

## 2. The `meta/` folder

All projects under this pattern keep their operational meta-state in
a folder called `meta/` at the repo root. The folder is reserved for
files Claude (and Pablo) consult to coordinate work across sessions.
It is not for product documentation or anything that belongs in the
repo's own structure.

What goes inside `meta/` depends on the project. The components below
are common building blocks; each project picks the ones it needs.

### Components

**`STATE.md`** — *Live state.* The default entry point. ~1–2 screens.
Sections vary by project, but typically: TL;DR, current focus,
blockers, what can move now, open decisions, where to look for
context. **Edit in place; the file does not preserve history.** Update
the date in the header on each change. Recommended for any project
with ongoing work that spans multiple sessions. *Does not hold:*
session-by-session narrative (day-log), per-release changes
(`CHANGELOG.md` if the project versions), or decisions with
substantial rationale that deserve a dedicated record (an ADR, if
kept).

**`log/<YYYY-MM-DD>.md`** — *Session narrative, one file per day.* A
subdirectory inside `meta/` where each day's sessions are recorded.
Multiple sessions in the same day share the day's file, with one
`## session N — <short description>` H2 per session. Rollover by the
date the closeout commit lands. The directory listing is itself an
index — `ls meta/log/` gives a date-sorted view of session history.
Typical session entry covers what happened, decisions banked,
what didn't land and why, brief observations, and required reading
for next time. Each file stays small, MCP-safe by construction.

**`CONTEXT.md`** — *Static project context.* For information that is
stable across sessions but doesn't fit `STATE.md` because of length —
deliverables, scope, stakeholder maps, fixed timelines. Add when
`STATE.md` would otherwise bloat with non-changing context.

**`README.md`** *(inside `meta/`)* — *Folder workflow.* Describes how
`meta/` is used in this specific project, including any deliberate
deviations from the canonical pattern. Add when the project diverges
from defaults in non-obvious ways.

A project that does not benefit from any of these is free to omit
`meta/` entirely, provided its instructions block points to whatever
serves as entry point.

### Additional files often kept at repo root

- **`README.md`** (root) — Project overview and pointers, including
  to `meta/STATE.md` if used.
- **`docs/decisions/`** *(opt-in per project; "ADRs")* — Architecture
  Decision Records, one decision per file, full ADR format
  (Context / Decision / Rationale / Consequences). Filename
  convention `NNNN-<slug>.md`. Opt-in: skip if there's nothing
  substantive enough to warrant it.
- **`CHANGELOG.md`** *(opt-in for versioned projects)* — Keep-a-
  Changelog format. **One entry per version bump, not per session.**
  Omit for unversioned projects.

## 3. Working with the repo efficiently

The GitHub MCP exposes a `create_or_update_file` operation that
requires sending the entire file content on every write. Beyond
roughly 30 KB of payload it becomes unreliable — silent truncation
producing partial commits with misleading commit messages. For files
under that threshold and single-line edits, the connector is fine.
For larger files, multi-edit sessions on the same file, or bulk
operations across several files, use this workflow instead:

1. **At the start of a session that will edit meta files**, fetch
   the relevant files from GitHub once and keep them in the local
   working filesystem (`/home/claude/`).
2. **During the session**, edit the local copies with surgical
   operations (`str_replace` or equivalent).
3. **At session end, or at meaningful checkpoints**, commit all
   changed files together. One commit per meaningful change set,
   not one per edit.

For sessions that touch only a single small file lightly, the full
download-edit-upload cycle via the connector is fine and not worth
optimising.

A trade-off to be aware of: the local working filesystem does not
persist across sessions. If a session ends without committing, the
local edits are lost. Propose deltas before the session ends so
Pablo commits them. For larger work that spans session boundaries,
commit at intermediate checkpoints rather than risk losing work.

## 4. Project knowledge — what belongs there

Project knowledge holds artefacts that:

- Are project-specific (so do not belong in transversal repos).
- Are not appropriate for version control (so do not belong in the
  project repo either).
- Are useful across sessions without re-uploading.

Typical contents: signed legal documents (NDAs, MSAs, SOWs),
architecture PDFs or system descriptions provided by counterparties,
email correspondence summaries or relevant exports, other reference
documents the project receives from outside.

What does not belong in project knowledge:

- The project repo, indexed via the GitHub sync feature. Use MCP only.
- Reference content from other repos you maintain. Read those via MCP.
- Material from other projects that has crept in.

For projects with no counterparty (personal tooling, exploratory
work), project knowledge can be empty.

## 5. Project instructions block

Each Claude project's "Instructions" field contains a block following
the template below. The block is kept short, since most of what
applies everywhere lives in the operational reference at
`PabloAccuosto/claude-projects/instructions-common.md`, loaded at
session start.

### Common section template

```
Project: <project name and one-line description>.
Working repo: PabloAccuosto/<repo-name> (<private|public>). Repo
access follows the rules in instructions-common.md (§ "GitHub
access: connector and PAT"). The GitHub sync feature in project
knowledge is not used.

The canonical pattern this project follows is documented at:
https://github.com/PabloAccuosto/claude-projects/blob/main/claude-project-pattern.md
The standing operational rules live at:
https://github.com/PabloAccuosto/claude-projects/blob/main/instructions-common.md
Read these if you are unfamiliar with the pattern. The rest of this
block is the project-specific application.

At session start, read in order:
1. https://github.com/PabloAccuosto/claude-projects/blob/main/instructions-common.md
   — standing workflow that applies to every personal project.
2. https://github.com/PabloAccuosto/claude-projects/blob/main/claude-personal-preferences.md
   — Pablo's working preferences (solo work, communication style,
   branching defaults, PAT protocol).
3. <project-specific entry point — typically meta/STATE.md, but
   adjust if the project has a different natural entry point>
4. <additional files only as needed for the task at hand>

Project-specific notes:
- <Anything specific to this project that is not covered by the
  standing workflow or personal preferences. Examples: counterparty
  contacts, contractual references, repo conventions that override
  the defaults. Keep it to what genuinely does not generalise.>
```

### Personal preferences file

Kept at `PabloAccuosto/claude-projects/claude-personal-preferences.md`.
Since this is solo work, every project's instructions block points
directly at this URL — there is no contributor-resolution step to
run (unlike the Eticas version of this pattern, which resolves the
contributor per session because several people share the org). If a
personal project is ever shared with a collaborator, add a resolution
step at that point — ask who's in the session, look up their
preferences file — rather than building it in ahead of need.

Keep the preferences file short. Operational state of any specific
project belongs in that project's `meta/STATE.md`, not here.

## 6. External data (large or sensitive files outside the repo)

Some projects hold data that:

- Is too large for the repo (gigabytes of PDFs, traces, logs).
- Is too sensitive for git history (raw personal data).
- Is provided under conditions that limit replication.

Pattern: data lives outside the repo, on the local machine or a
controlled location. The repo holds a manifest describing what
exists, not the data itself.

Conventional location: a `data/` folder, gitignored except for two
files:

- **`data/README.md`** — where the actual data lives, why it is not
  in the repo, how to request access.
- **`data/MANIFEST.md`** — index of files outside the repo. Per file:
  filename, one-line description, date or version, source,
  approximate size, sensitivity classification.

When a session needs the actual content of a specific file, the file
is uploaded to `/mnt/user-data/uploads/` for that conversation only.
It does not persist to project knowledge or the repo.

For projects that repeatedly reference the same large source
documents, extracting their text once and committing the extracted
text (not the PDFs) to the repo can be worth the trade-off —
searchability and persistence at the cost of formatting and images.

## 7. Applying the pattern to a new or existing project

When setting up or migrating a project:

1. **Inventory what exists.** For existing projects, classify each
   file in project knowledge and in the repo: belongs here, belongs
   elsewhere, duplicate, stale, unclear (ask).
2. **Disconnect any GitHub sync** between the project knowledge and
   the repo. Use MCP only.
3. **Set up the repo skeleton.** At minimum, a root `README.md`
   pointing to wherever the entry point is. Add `meta/` if the
   project benefits from any of the components in §2. Add `data/`
   with manifest if external data applies (§6).
4. **Bootstrap entry-point content** by reading whatever existing
   state is scattered around and consolidating into `meta/STATE.md`
   or the chosen entry point.
5. **Write the instructions block** using the template in §5,
   referencing this pattern document by URL.
6. **Reset memory** of the Claude project to stable entries only.
   Operational state, key findings, and transient context belong in
   the repo now, not in memory.
7. **Update your own project inventory**, if you keep one, so the
   project is registered as following the pattern.

Order matters. Steps 1 and 2 first, so structure is built on a clean
inventory. Step 6 last, so context is not lost before the repo holds
it.

## 8. Maintenance

When a project surfaces a useful refinement to the pattern, it is
proposed as an update here and applied across active personal
projects when relevant. When a refinement does not generalise, it is
documented as a local exception in the project's `meta/README.md`.

## 9. GitHub access beyond the MCP connector

The MCP connector is the primary path for repo work and should
remain so — it is the lowest-friction, lowest-secret-exposure way to
interact with GitHub. But it is not capability-complete. The hosted
GitHub MCP server intentionally does not expose write operations on
tags, GitHub Releases, or `.github/workflows/` files, and is
one-call-per-file in a way that makes bulk operations impractical.

The pattern accommodates this with a session-scoped PAT protocol:
when an operation requires capabilities the connector does not have,
Pablo pastes a fine-grained PAT into the chat inside a code block,
Claude uses it via `git`/`gh`/REST API in `bash` for the duration of
that session, and the token is never written to any file or surface
that persists beyond the chat.

The default behaviour at session start, the security boundaries, and
recommended PAT scopes live in
[`instructions-common.md`](https://github.com/PabloAccuosto/claude-projects/blob/main/instructions-common.md)
under "GitHub access: connector and PAT". The personal preferences
file can override the default behaviour.

The PAT path is for the gaps. It does not replace the connector; it
extends it.
