# Multi-Session Coordination Guide -- for a new Claude Code colleague joining AAAA OS

Version 1.0, written 2026-07-29 by `conv-claude-architecture-helper-pc-001`
("allo ?"), based on real coordination with a second PC-local Claude Code
session ("Bello", `conv-claude-pc-bello-001`) over 2026-07-28/29. Every
pattern below is something that actually happened and was verified, not
a theoretical design -- referenced commits/files exist in this repo.

If you are a new Claude Code session joining this project on the same
PC as an existing session (or about to be), read this before doing
anything else infra-related.

## 1. You are not alone on this machine -- check first

Multiple Claude Code sessions can run simultaneously on the same
Windows account, often from the **same working directory** (here, the
user's local `Downloads` folder) with the **same git checkout** (a single
local clone of the repository) -- not separate clones. Before touching git
or infra:

- Read `switch.md` (kept in that shared working directory) in full (or at least its
  recent entries) -- it is the hand-relayed coordination log between
  sessions. Check the last dated entries for what's in progress, what's
  blocked, and what the current division of labor is.
- Run `git status` before any `pull --rebase`, `reset`, `checkout --`,
  or broad `git add`. If you see files modified that you didn't touch,
  **that's someone else's in-progress work -- don't stage or discard it.**
- Never `git add -A` or `git add .` -- always name files explicitly. A
  broad add can silently sweep another session's uncommitted work into
  your commit.

## 2. Get a real identity, not just a nickname

A nickname in `switch.md` ("allo ?", "Bello") is not a real C2C identity
-- it's a human-readable label for the coordination log. If your role
involves being addressed a REQUEST by another session, K3, or a future
colleague, **register a real conversation ID** in
`c2c-os/02_operational_registers/conversation_registry.yaml`:

- Pick an ID matching the pattern `conv-<descriptive>-<NNN>`.
- Give it a real `mailbox:` path
  (`c2c-os/03_handoffs/mailboxes/<your-id>/inbox/`) and seed it with one
  `STATUS` message so the folder exists for real in git (empty
  directories aren't tracked).
- State your actual role/constraints in the registry entry, not just a
  vague description -- see `conv-claude-pc-bello-001`'s entry for a real
  example (standing cross-review owner, side-project default recipient,
  explicit constraints list).

**Verify your own identity before signing anything.** Check your own
memory for `originSessionId` and cross-reference against
`switch.md`/the registry before adopting a nickname from context alone
-- this exact mistake happened for real on 2026-07-28/29 (a session
signed as "conversation principale" for hours before catching, via its
own persistent memory, that it was actually "allo ?"). Getting this
wrong doesn't break anything technically, but it corrupts the shared
coordination record for everyone reading it later.

## 3. The real division-of-labor pattern that worked

When two sessions are both active, don't just react to individual asks
-- periodically compile and post a **full task list** (blocked-on-human /
in-progress / done / planned / frozen) and propose a concrete split,
then let the other session confirm or counter-propose. This was done
explicitly on 2026-07-29 (see `switch.md`, the "liste complète" entries)
and worked well because:

- It surfaces stale/forgotten threads that neither session would think
  to mention individually.
- It makes "who does what" explicit instead of assumed, reducing the
  chance both sessions duplicate the same work or both skip something
  assuming the other has it.
- A simple split that has worked here: one session stays on a specific
  deep thread until closure (e.g. a grant-proposal package) plus general
  infra monitoring; the other owns standing cross-review + side-projects
  + its own autonomous tooling (watchdogs, status briefs) with no
  handoff needed for those.

## 4. Independent cross-review catches real bugs -- use it, don't skip it

Every time work was reviewed by the *other* session rather than
self-checked, real bugs surfaced that the author had missed:

- A registry write race condition, a non-atomic state file, a plaintext
  secret in a systemd unit, and a false "committed to git" claim -- all
  found by one session reviewing the other's WhatsApp bridge + pipeline
  code (2026-07-28), all fixed same-day, all re-verified live afterward.
- A live-test of an independently-authored file (K3's own draft of a
  matcher module) found a real completeness gap and a real extraction
  bug the author hadn't caught.

**Practical rule**: when you deploy or fix something real, send the
standing cross-reviewer (once registered per §2) a REQUEST, or at
minimum post to `switch.md` with enough detail that a fresh reader could
independently re-derive whether it's correct -- not just "I fixed it."

## 5. K3 (kimi-k3) is a real coding collaborator, not just an opinion source

Dispatch real, bounded build tasks to K3's mailbox
(`c2c-os/03_handoffs/mailboxes/kimi-k3/inbox/`) when there's a
well-scoped piece of work that doesn't need confidential data (schema
design, template structure, an independent second implementation to
compare against). K3 responds automatically via an existing poller
(every 20 min, on the production server) -- but always:

- **State the scope boundary explicitly** in the dispatch (what NOT to
  touch, what data must never be invented).
- **Live-test what comes back** before merging -- don't take a
  self-reported "hand-checked" status as machine-verified. K3 itself
  flagged this distinction in a real response (2026-07-29): "hand-checked
  ... but not yet machine-validated by me" -- the receiving session ran
  the actual validator and confirmed it, closing the loop properly.
- Send a `CLOSE` message back when done, crediting what was kept.

## 6. Never invent missing information, even under deadline pressure

This project's own standing rule (see the context bundles of the grant proposal then in progress)
applies to every session: if a real deliverable (a grant-proposal
document, an evidence register, a technical claim) references facts you
don't have verified access to, **mark them `OPEN`/`TBD` explicitly**
rather than filling gaps with plausible-sounding content. This was
tested for real on a dense, time-pressured R15 request (2026-07-29) --
the response that shipped was a faithful transcription of what was
already given plus an honest `BLOCKED` status on the one piece that
genuinely couldn't be done (a VPS mirror with no approved path), not a
fabricated success.

## 7. Windows/PC-local realities worth knowing before you hit them yourself

- **`schtasks`/Task Scheduler**: a sandboxed Claude Code PowerShell
  session may get `Access denied` even creating a brand-new scheduled
  task -- this is a tool-sandbox limitation, not a Windows permission
  issue on the user account. The user running the same command in their
  own **elevated** PowerShell window works. Prepare the exact script for
  them rather than assuming you can apply infra-level Windows changes
  yourself.
- **Nested quoting through `schtasks /change /tr`** is unreliable for
  complex command lines. Write a small dedicated `.vbs` wrapper with the
  full command as a VBS string literal instead of trying to pass nested
  quotes through the command line.
- **`WScript.Shell.Run(cmd, 0, True)`** (via a `.vbs` launcher) is the
  only mechanism found on this machine that reliably hides a console
  window regardless of caller (Task Scheduler, Startup folder) --
  PowerShell's own `-WindowStyle Hidden` alone does not, in this context.
- **git's `core.autocrlf`** converts LF to CRLF on Windows checkouts of
  loose files, but does **not** alter what's already stored in a git
  blob or inside a binary archive (a `.zip`) built from those files
  before the checkout conversion happens -- verified by hashing a
  committed file both locally and via the real GitHub API and getting
  identical hashes. Don't assume CRLF corruption without checking.

## 8. What NOT to do

- Don't guess a credential/token exists somewhere without searching
  (memory, this repo, both VPS `.env` files) first -- and don't ask the
  human to re-supply something you haven't actually confirmed is
  missing. If they say "I already gave you X," search harder before
  concluding they're wrong.
- Don't treat a `claude.ai` "Connectors"-style tool (card rendering,
  `list_connectors`, `search_mcp_registry`) as equivalent to Claude
  Code's own plugin marketplace -- they're different systems. An empty
  result from `search_mcp_registry`/`list_connectors` in a CLI session
  does not mean no plugins/connectors exist at all: verified 2026-07-29
  that both returned empty for every query (github, slack, notion,
  google drive...) while `claude plugin list --available --json` showed
  a real, working catalog of 276 plugins from the official marketplace
  (`claude-plugins-official`), including an official `github` MCP
  plugin. Check the plugin CLI directly before concluding "nothing is
  available."
- Don't rewrite another session's history in `switch.md` to "fix" a
  mistake -- append a correction instead. Concurrent edits to the same
  historical lines risk a real collision.
