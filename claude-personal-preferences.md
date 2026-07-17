# Pablo's personal preferences for Claude

> Loaded at the start of every session where Pablo is the contributor.
> This file holds Pablo's personal working
> preferences — things that are true regardless of which project the
> session is about, but that don't generalise to other contributors.
>
> Project-specific notes go in the project's own instructions block.
> Eticas-wide workflow lives in `Eticas-AI/ai-ops/instructions-common.md`.

---

## Working style

**Pablo works solo on his projects by default.** If a project's
instructions don't say otherwise, assume single-contributor.

**Routine updates to state files** (`STATE.md`, `meta/`) can go directly to `main`. Use feature
branches and PRs only for changes that are large, risky, or benefit
from a diff view — typically: changes to repo structure, project
instructions, or the canonical pattern in `ai-ops`.

When in doubt about whether a change is "routine", branch.

## Session start: confirm the goal before starting work

**Orient first, then confirm the session's focus with Pablo before
beginning any work.** After the start-of-session reads (project
instructions, this file, `STATE.md`, and `CONTEXT.md` when needed),
give a short summary of where things stand and the candidate next
steps — then ask Pablo what he wants to do this session. The pending
items in `STATE.md` are *candidates*, not a standing agenda: Pablo
sets the agenda each session. Don't start drafting, building, or
editing until he has confirmed the focus.

This is a deliberate session-opening exception to "state assumptions
inline rather than asking permission" below — it applies to *what to
work on at the start of a session*, not to execution once the focus
is agreed. Orientation reads and the situational summary proceed
without asking; the pause is before starting the actual work.

## Communication

**Direct recommendations over option lists.** When one option is
clearly better given what you know, recommend it — don't present a
menu and ask Pablo to pick. Pablo prefers a clear lead followed by
the rationale to a balanced presentation of alternatives that defers
the call.

**Clarifying questions only when a decision genuinely needs Pablo's
input** and you cannot recommend without it. If you have enough
context to make a defensible call, make it; if you're not sure but
the cost of being wrong is small, make it and explain. Use
clarification for genuine forks, not for hedging.

**State assumptions inline rather than asking permission.** If you
need to assume something to proceed (e.g., a default value, an
interpretation of an ambiguous request), state the assumption in
your response and continue — Pablo can correct in the next turn if
the assumption was wrong.

## Pace and approval

**Pause and seek approval before pushing changes to GitHub.** This
applies to every operation that writes to a repo —
`create_or_update_file`, `push_files`, branch creation, PRs, merges,
tag/release creation. Before any such operation, Claude states the
plan (what files, what commit message, which repo, which branch)
and waits for Pablo's go-ahead. The default is one explicit
approval per push.

**Exception: explicit autonomous task.** When Pablo specifically
asks for a multi-step task to be executed without intervention
("hacé X, Y, Z y avisame al final", "ejecutá todo el flujo", or
equivalent), Claude executes the full sequence and reports at the
end. The exception requires Pablo's explicit framing, not Claude's
inference from context.

**Pace inside the session.** When uncertain whether to keep moving
or pause, ask Pablo rather than deciding unilaterally. The right
pace depends on the type of task — some work benefits from
momentum, some from breaks to process. If Claude has produced a
lot of output in one turn (multiple decisions, several drafts,
dense technical content), check in with Pablo before continuing
rather than assuming the next step is welcome.

**Reading-only and analysis steps don't need approval.** Fetching
files, running searches, triaging comments, drafting content for
review in chat: all proceed normally without per-step confirmation.
The bar applies to *writes to GitHub*, not to thinking out loud.

## Don't assume the session is ending

**Completing a task is not the same as ending the session.** A
finished task — even a substantial one — may be one step in a
larger flow Pablo has in mind. Before triggering any end-of-session
action, ask Pablo whether the session is wrapping up or whether
more work is coming. The end-of-session protocol runs once Pablo
confirms, not on Claude's inference that "this seems like a good
stopping point".

End-of-session actions covered by this rule include:

- Updates to the project's `meta/STATE.md` (priorities, recent
  decisions, PK cache state).
- New entries in the day-log entry in `meta/log/<today>.md`, plus an ADR
or `CHANGELOG.md` entry as applicable per the project's conventions — see [`Eticas-AI/ai-ops/claude-project-pattern.md`](https://github.com/Eticas-AI/ai-ops/blob/main/claude-project-pattern.md)
- PK cache delivery (offering the updated cached file to be uploaded
  to project knowledge).
- Handover documents.

These are checkpoints, not chronological closures — they should be
proposed deliberately, after confirmation, rather than emitted
automatically when a piece of work appears finished.

## Script execution

**Pablo runs scripts locally.** The working model is: Claude writes
and edits code and commits it to the repo; Pablo runs it on his own
machine, where the `.env` with real credentials lives. Three reasons:
Claude should not have access to client API keys (Wiselook or
otherwise — also an NDA consideration); running locally keeps Pablo
close to what each step actually does; and Pablo's environment is
already set up.

**Claude does not run scripts that require real client credentials,
and does not ask for those keys to be loaded into its sandbox.**
Claude's sandbox use is limited to credential-free dry checks —
compiling, importing, linting, offline logic, URL/shape resolution
that needs no secret. Anything that needs a real key is handed to
Pablo to run locally; Pablo brings back the output to iterate on.

## GitHub access (PAT)

**Behaviour: `PAT-preferred` for all repos.** At the start of every
session that may write to any GitHub repo, Claude assumes a PAT will
be used and asks Pablo to provide one in a code block. The PAT Pablo
provides is expected to cover `PabloAccuosto/*` repos under a single token.
If Pablo declines or skips, the
session falls back to the MCP connector — no second prompt, **except
for private repos under `PabloAccuosto/*` (see note below), where
there is no connector fallback at all.**
Rationale: pushes via MCP send file content as Claude-generated
tokens (linear in file size), which is slow and expensive on
multi-file or large-file updates; PAT bypasses this. Empirically,
the MCP write tools also become unreliable beyond roughly 30 KB of
payload — silent truncation, partial commits with misleading commit
messages (the empirical threshold and failure mode are documented
in [`Eticas-AI/ai-ops/claude-project-pattern.md`](https://github.com/Eticas-AI/ai-ops/blob/main/claude-project-pattern.md)
§3). For any file near or above that size, the git CLI path is
required, not optional. The full protocol is in
[`Eticas-AI/ai-ops/instructions-common.md`](https://github.com/Eticas-AI/ai-ops/blob/main/instructions-common.md)
under "GitHub access beyond the MCP connector".

**Note on connector identity (found 2026-07-12).** The GitHub MCP
connector is authenticated as `PabloAccuosto-ETICAS`, not as the
personal account `PabloAccuosto`. This means the "falls back to the
MCP connector" behaviour described above **does not exist** for
private repos under `PabloAccuosto/*`: without a PAT, reads and
writes to those repos return a 404 (indistinguishable from
"repo doesn't exist"), not a slower-but-working alternative. For
those repos the PAT isn't merely *preferred* — it's the only path.
The connector works without a PAT only for (a) public repos
(read-only), or (b) repos where `PabloAccuosto-ETICAS` itself has
access (e.g. Eticas org repos). If a session opens against a private
personal repo and the connector 404s, treat it as this identity gap,
not a broken connection — ask for the PAT rather than concluding the
repo or file is missing.

**Prefer git CLI for writes when the PAT is loaded.** Use `git`
command-line operations (`clone`, `commit`, `push`) for file-level
writes, and the GitHub REST API (via `curl` or equivalent) for
repo-level operations git can't do — opening PRs, merging PRs,
deleting branches, querying check runs. MCP write tools
(`create_or_update_file`, `push_files`, etc.) are the fallback when
neither git nor the REST API is available (e.g., PAT didn't
authenticate). Reads via MCP remain fine when convenient.

**Token rotation is Pablo's responsibility, done manually, ~daily —
not per session.** Claude does not need to remind Pablo to revoke
tokens at session end. Pablo handles rotation himself on his own
schedule (typically once per day on days he uses Claude). The only
exception: if a PAT is exposed in a way that warrants immediate
revocation (e.g., pasted as plain text outside a code block),
Claude flags it explicitly per the protocol.

**Authorship**
All GitHub operations must be authored by *Pablo's GitHub user*. This is because Pablo assumes full responsibility for the repository content, which cannot be delegated to Claude.
For transparency, commit messages can include a line indicating that the work was done with Claude's assistance during Pablo's session.
If project-specific instructions contradict this, Claude should flag it at the beginning of the session.

This is a per-contributor scope refinement of the protocol's
default; it does not change the protocol itself.

## Reading this file

This file is referenced from the instructions block of every project
where Pablo is the contributor. If the project's own instructions
contradict something here, the project's instructions win for that
session — but flag the discrepancy so the divergence is intentional,
not accidental.
