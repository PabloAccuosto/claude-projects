# Pablo's common instructions for Claude

> Operational reference loaded at the start of every session in any
> of Pablo's personal Claude projects. This file tells you *what to
> do*. The conceptual rationale lives in
> [`claude-project-pattern.md`](https://github.com/PabloAccuosto/claude-projects/blob/main/claude-project-pattern.md) —
> read that if anything here is unclear or seems to conflict with
> itself.
>
> Adapted from Eticas' `Eticas-AI/ai-ops/instructions-common.md`,
> generalised for solo use under Pablo's own GitHub account.

---

## Source of truth

The locations below have clear roles. If the same content appears in
more than one location, the canonical location wins.

- **Project repo** — single source of truth for state, code, and
  project-specific synthesis. Reached through GitHub MCP only, never
  through the project knowledge GitHub sync feature.
- **Project knowledge** — counterparty artefacts not appropriate for
  version control (signed contracts, NDAs, architecture PDFs,
  reference documents, email correspondence).
- **Reference repos** — if a project depends on other repos Pablo
  maintains (a shared template, a personal knowledge base), the
  project's instructions declare them explicitly and Claude reads
  them via MCP at the moment they're needed.
- **Project-declared additional sources** — some projects depend on
  systems beyond the two above (SharePoint, Google Drive, a
  third-party issue tracker). When this applies, the project's
  instructions block declares the source explicitly and states its
  operating rules: when to read it directly versus when a digest in
  the repo suffices, what is and is not mirrored, and any access
  prerequisites.

If you find content in project knowledge that contradicts the
canonical repo, the repo wins — flag the contradiction so Pablo can
clean it up.

## Workflow

**Commits to state files** (`STATE.md`, anything in `meta/`) use the
message convention `meta: <what changed>`. Other commits follow the
project's own conventions.

**Branching default.** Use a feature branch + PR for any change that
is large, risky, or benefits from a diff view. Small routine updates
to state files may go directly to `main` — see the personal
preferences file for the current default. When in doubt, branch.

**End-of-session protocol.** Propose deltas to the relevant `meta/`
and root files at session close. Pablo reviews and commits. Do not
commit autonomously to `main` unless the personal preferences file
explicitly authorises direct commits for the kind of change you are
making.

**Before modifying repo structure, project instructions, or
meta-state files**, verify that the canonical pattern in
`claude-project-pattern.md` is consistent with what you are about to
do. If you find drift, flag it before proceeding — do not silently
work around it.

## Working with meta files efficiently

Meta files grow over time, and the GitHub MCP `create_or_update_file`
tool requires sending the full file on every write. For sessions that
touch meta files multiple times:

1. Fetch the relevant files from GitHub once at the start, into the
   local working filesystem (`/home/claude/`).
2. Edit the local copies with surgical operations (`str_replace`),
   not by regenerating full content.
3. Commit all changes together at session end (or at meaningful
   checkpoints) in a single push.

For sessions that touch only a single file lightly, the
download-edit-upload cycle is fine and not worth optimising.

## GitHub access: connector and PAT

Two paths exist for talking to GitHub from a session: the **MCP
connector** and a **session-scoped Personal Access Token (PAT)** used
directly via `git`, `gh`, or the GitHub REST API in `bash`. They are
not interchangeable, nor is one strictly "primary" — the right choice
depends on the operation.

### Choosing between the connector and the PAT

**(1) Connector only — operations the PAT does not improve.**

Prefer the connector even when a PAT is loaded in the session. Using
the connector keeps the PAT's effective surface narrow and avoids the
small risk of malformed bash. Includes:

- Single commits to small or medium-sized files (under ~500 lines as
  a rough guide).
- Reading files (`get_file_contents`).
- Listing branches, commits, tags, releases, PRs, issues.
- Opening or merging pull requests.
- Creating or updating one issue or PR comment.
- Searching code, issues, PRs, repositories.

**(2) PAT preferred — operations where the connector technically
works but is technically worse.**

If a PAT is already loaded in the session, prefer it for these. The
exposure of the secret has already happened; the remaining choice is
technical, and the technical choice favours `git`/`gh`/`curl`.
Includes:

- **Edits to large files.** The connector requires re-sending the
  entire file on every write — costly in tokens, fragile against
  truncation, awkward when multiple edits are needed in sequence.
  With a PAT, `clone → str_replace → commit → push` does surgical
  edits and one push.
- **Multiple sequential edits to the same file in one session.**
- **Bulk operations across many files** (delete a directory, rename
  a folder, refactor that touches a dozen paths). The connector is
  one-call-per-file, which becomes impractical fast.

**(3) PAT only — operations the connector cannot perform.**

- Creating, editing, or deleting **GitHub Releases**.
- Creating annotated tags or pushing tags to the remote.
- Editing files under `.github/workflows/`.
- A few other endpoints — when the connector returns "not found" or
  "no such tool" on a known-valid endpoint, treat it as a (3) case.

### When in doubt

Default to the connector. The worst case is paying some extra tokens
or running into a one-shot inefficiency; correct on the next
operation.

### Personal override of the category defaults

The three categories above describe the default reasoning when the
personal preferences file doesn't specify further. A blanket
override is valid and takes precedence: if the preferences file
states something like "prefer git CLI/PAT for all writes when a PAT
is loaded, regardless of category," that applies even to category
(1) operations — single commits to small files, one-off file
creation, etc. Pablo's current preference is exactly this: PAT
preferred for all repos, git CLI preferred for writes whenever a PAT
is loaded. Check the personal preferences file before defaulting to
the connector for category (1) work.

### Default protocol — ask for the PAT at session start

By default, at the start of every session Claude announces that a
PAT may be needed and lets Pablo decide whether to provide it. The
opening line is short and non-blocking: "If this session will
involve tags, GitHub Releases, workflow files, or non-trivial edits
to large files, paste a PAT inside a code block. Otherwise we
proceed with the connector only."

If Pablo declines and later an operation falls into category (3),
Claude flags the gap, explains what permission is needed, and waits
for Pablo to decide whether to provide a PAT or do the operation
manually. If an operation falls into category (2) and no PAT is
loaded, Claude proceeds with the connector and notes the
inefficiency rather than blocking.

This default can be overridden by the personal preferences file.
Recognised values for the `pat` preference:

- **`ask-on-start`** — announce the option at session start.
- **`ask-when-needed`** — do not announce at start; only ask when an
  operation actually requires it.
- **`never`** — do not ask. If an operation requires a PAT, flag it
  and stop; Pablo will handle that operation outside the session.
- **`pat-preferred`** — assume a PAT will be used for any session
  that may write to GitHub, and ask for it upfront regardless of
  which category the work turns out to fall into.

### Handling a PAT during a session

When a PAT is provided:

1. The PAT must arrive **inside a triple-backtick code block**. If
   Pablo pastes it as plain text, Claude immediately asks him to
   revoke that token in GitHub UI, generate a fresh one, and paste
   the new one inside a code block. The compromised token is not
   used for anything.

2. Claude persists the token to a session-local file in the working
   filesystem with restrictive permissions, then reads it from that
   file in each `bash` invocation that needs it:

   ```
   # First time the token is received:
   umask 077
   cat > /home/claude/.gh_token << 'EOF'
   <token-value>
   EOF
   chmod 600 /home/claude/.gh_token

   # In every subsequent bash_tool call that uses the token:
   GH_TOKEN=$(cat /home/claude/.gh_token)
   curl -H "Authorization: Bearer $GH_TOKEN" ...
   # or
   git push "https://x-access-token:${GH_TOKEN}@github.com/<owner>/<repo>.git" <branch>
   ```

   The reason for the file is not security but persistence: each
   `bash_tool` call starts a fresh shell, so an `export GH_TOKEN=…`
   in one call does not survive into the next. The file in
   `/home/claude/` does persist across `bash_tool` calls within a
   session and is wiped automatically when the session ends.

3. Claude does not echo the token back, does not write it to any
   file outside `/home/claude/.gh_token`, does not include it in
   commit messages, PR bodies, or remote URLs persisted to
   `.git/config` (after `git clone`, sanitise with
   `git remote set-url origin <https-url-without-token>`), and does
   not reference its value in subsequent turns.

4. At session end, before the working filesystem is wiped, Claude
   removes the token file explicitly: `shred -u /home/claude/.gh_token`
   (or `rm -f` if shred is unavailable).

### Where a PAT must never be stored

PATs are session-scoped secrets. They must never be placed in:

- The project's instructions block (loaded into every future session
  and visible to anyone with access to the Claude project).
- Project knowledge (same persistence and visibility).
- Any file in any repo (committed content is permanent).
- `meta/` files of any kind.
- Any file outside `/home/claude/` in the working filesystem.

The token lives only in the chat of the session that received it, in
the session-local `/home/claude/.gh_token` file, and transiently in
`bash` shells that read it from that file. Pablo is encouraged to
revoke session-pasted PATs after sensitive sessions rather than
relying solely on expiry.

### Suggested PAT scope

When asked to advise on PAT configuration, recommend **fine-grained
tokens** (not classic) with:

- **Resource owner** matching the account or org the repos belong
  to. One PAT cannot span multiple owners — if Pablo works under his
  personal account and under an organization in the same session,
  those need separate tokens.
- **Repository access** restricted to the specific repos the session
  will touch (`Only select repositories`), not all repos of the
  owner.
- **Expiration** of 30 to 90 days. Shorter is better; 90 is a
  reasonable ceiling.
- **Permissions** scoped to what the session actually needs. Common
  combinations:
  * For tag and release work: Contents R/W, Pull requests R/W,
    Issues R/W, Workflows R/W, **Administration R/W** (this is what
    enables tag and release creation — easy to miss), Metadata R.
  * For routine state-file editing only: Contents R/W, Metadata R.

### Failure modes when using a PAT

A PAT can fail at the moment it is loaded (authentication) or at the
moment it is used (authorisation). Claude must surface these clearly
— never silently switch back to the MCP connector or pretend the
operation succeeded.

**Authentication failure (401 Unauthorized).** The token is invalid:
expired, revoked, malformed, or never existed. Before concluding the
token is bad, verify it was actually present in the call by printing
its length (never the value) —
`python3 -c "import os; print(len(os.environ.get('GH_TOKEN','')))"` —
because an empty `GH_TOKEN` (e.g. set in a previous `bash_tool` shell
that no longer exists) produces a 401 that looks identical to a
revoked-token 401. If the length is non-zero and 401 still occurs,
tell Pablo: "The PAT is invalid — likely expired or revoked. Generate
a new one in GitHub UI (Settings → Developer settings → Personal
access tokens → Fine-grained tokens) with the same configuration as
before, then paste it in a code block."

**Authorisation failure / scope insufficient (403 Forbidden,
sometimes 404 on private repos).** The token authenticates but lacks
the specific permission for the operation. GitHub sometimes returns
404 instead of 403 on private repos to avoid leaking repo existence
— interpret 404 on a known-existing repo as likely-403. Identify the
operation that failed, name the missing permission, and ask Pablo to
either regenerate the PAT with the additional permission or perform
that operation manually.

**Wrong resource owner.** The PAT was generated for one owner (e.g.
Pablo's personal account) but the session is trying to operate on
repos of a different owner (e.g. an organization). This manifests as
the PAT reading repo metadata fine but failing on writes with 403.
One PAT cannot span owners — ask for a second PAT scoped to the
other owner, or fall back to the MCP connector for those operations.

**Smoke test before substantive work.** When a PAT is first loaded in
a session, run a single low-cost API call to confirm it works:
`GET /repos/{owner}/{repo}` against one of the repos the session will
touch. The response shape (`permissions: {admin, push, pull}`) also
tells you whether the scope matches what the session needs. If the
smoke test fails, fail loudly before any state-changing operation.

**Git over HTTPS auth quirk.** The GitHub REST API accepts a
`Authorization: Bearer <token>` header straightforwardly, but `git`
over HTTPS does not always honour the same form. The reliable form
for `git clone` and `git push` against a fine-grained PAT is the
embedded-credentials URL with `x-access-token` as the username:

```
git clone "https://x-access-token:${GH_TOKEN}@github.com/<owner>/<repo>.git"
git push "https://x-access-token:${GH_TOKEN}@github.com/<owner>/<repo>.git" <branch>
```

After cloning, sanitise the remote so the token does not persist in
`.git/config`:

```
cd <repo>
git remote set-url origin "https://github.com/<owner>/<repo>.git"
```

Prefer the `x-access-token` URL form over
`git -c http.extraheader="Authorization: Bearer ${GH_TOKEN}"`, which
has been observed to fail in practice with fine-grained PATs against
GitHub's git-over-https endpoint.

### Maintenance and rotation

PATs expire. Pablo is responsible for regenerating them when expiry
approaches; Claude does not track expiry across sessions. When a PAT
fails authentication, treat it as a regeneration prompt rather than a
debugging puzzle. If a session pastes a PAT that turns out to be
invalid, Claude does not try alternative tokens and does not silently
fall back to MCP for operations that need the PAT.

## What does not belong in project knowledge

- The project repo, indexed via the GitHub sync feature. MCP only.
- Reference content from other repos. Read those via MCP.
- Material from other projects.

## Drift between project instructions and this file

Each project's instructions block points here, plus:

- The project's entry-point file (typically `meta/STATE.md`).
- The personal preferences file.
- Project-specific notes that don't generalise.

The project block is meant to stay short — most workflow content
lives here or in `claude-project-pattern.md`. Over time, drift
accumulates: a project block may duplicate content this file already
covers, or contradict guidance that's since been updated, or still
hardcode an assumption the canonical pattern has retired.

### Active drift check at session start

Once per session, after reading the project's instructions block and
this file, do a quick comparison and decide whether there's
substantive drift worth flagging. Look for:

- **Duplication.** Content in the project block that this file or
  `claude-project-pattern.md` already covers.
- **Contradiction.** A claim in the project block that disagrees with
  current wording here.
- **Obsolete shape.** The "read in order" steps don't match the
  current §5 template.

What does **not** count as drift: stylistic wording differences,
section ordering within project-specific notes, or genuinely
project-specific exceptions that diverge from the canonical pattern
on purpose.

### How to handle drift

If the check finds nothing substantive, say nothing. If it's
substantive, don't block the session's actual work — note it briefly
near the start ("the project block is out of date with respect to X
— I can prepare an updated version at the end if you want") and
proceed. At session end, if agreed, propose the updated block as a
delta, typically as a downloadable artefact since the project block
lives outside the repo.

### When project instructions and this file disagree

Independent of the drift check: if a project's instructions and this
file disagree on a specific point during normal work, the project's
instructions win for that project — but flag the discrepancy so the
divergence is intentional, not accidental drift.
