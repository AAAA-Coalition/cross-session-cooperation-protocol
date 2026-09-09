# Inter-Agent Cooperation Policy — Draft Common Policy

**Status: allo draft, pending co-signature by allo-desktop and an adversarial
audit.** Written on 29 August 2026 at 10:45 (Dublin).

This document exists because current cooperation relies on unwritten
conventions between two sessions that communicate frequently. **That will not
survive the arrival of a human colleague.** It is written for someone who has
never participated in any of our conversations.

---

## 1. What we built, and what it cost us

Four channels, all tested in real conditions:

| Channel | What it does | Evidence |
|---|---|---|
| **C2C mailboxes** (git + Markdown) | canonical, timestamped, auditable channel | 89 K3 responses received, 372 messages in my mailbox |
| **session-bridge** | real time, same machine | ping `msg-etzjb6ytbifk` transported |
| **`DEMANDES_JOHN.md`** | John writes once, N sessions read | **NEVER USED** — empty template, zero messages |
| **Google Drive** | large files, humans outside the repository | round trip proved, canary `POMMIER-4471` |

**And the six cooperation failures observed in three days**, each dated:

1. **Duplication.** On 29 August, I rebuilt the monitoring page that
   allo-desktop had published an hour earlier. No mechanism prevented it.
2. **Message unread for four hours.** On 28 August, its 10:38 alert reporting a
   dead-letter service remained unread until 14:00.
3. **Wrong channel read.** I concluded that K3 was dead by reading its `outbox`,
   empty by design; its 89 responses arrived as `reply_MSG-BRIDGE-*` in my own
   mailbox.
4. **Ownership assumed, not verified.** I declared the W3 project mine by
   reasoning from a scope rule; `git log` says it is theirs.
5. **Invented timestamps.** Six of my messages carried arbitrary times, up to
   1 hour 25 minutes in the future — breaking the reading order for all agents.
6. **False status.** Tasks marked `completed` without any command capable of
   proving completion.

**These six failures have one common cause: nothing requires verification
before asserting, or announcement before acting.**

---

## 2. What exists elsewhere — and the finding that matters

Research performed on 29 August. Sources are listed at the end.

### `mcp_agent_mail` — we reinvented the same thing, less completely

[`Dicklesworthstone/mcp_agent_mail`](https://github.com/dicklesworthstone/mcp_agent_mail)
is an asynchronous coordination layer for coding agents: **identities,
inboxes, searchable threads, and advisory file leases**, built on FastMCP + Git
+ SQLite. A Rust version exists with 34 tools, a Git-backed archive, SQLite
index, advisory locks and a TUI console.

Its storage writes **canonical Markdown/JSON artefacts to one Git repository
per project** — exactly our architecture, reached independently.

**The two missing components:**

- **The advisory file lease**: an agent announces that it is working on a
  scope, with an **expiry period**. This is exactly what would have prevented
  the duplication on 29 August. The TTL is essential: an agent that crashes
  must not block everyone else forever.
- **The search index** (SQLite + FTS5): my mailbox has **372 files out of the
  1,000** at which the GitHub API silently truncates. An index solves both
  search *and* the approaching limit.

### What has become native and must be tested before building

- **Agent Teams** (Claude Opus 4.6, February 2026): one lead session spawns
  teammates, each with its own context and tools; they communicate through a
  **mailbox** and coordinate using a **shared task list**. This is our model,
  provided natively.
- **Channels**: a pub/sub layer built into Claude Code, real time **without
  polling or shared-state workarounds**.

### A2A — the standard for dialogue between agents

[Agent2Agent](https://en.wikipedia.org/wiki/Agent2Agent), v1.0 in April 2026,
with more than 150 organisations. Where **MCP connects an agent to its tools**,
**A2A connects one agent to another**. We have performed A2A manually for
months without naming it. Adopting its vocabulary (agent card, task lifecycle)
costs little and makes the system understandable to a third party.

### Isolation: Git worktrees

An established practice for allowing multiple agents to work without conflict:
each uses its own *worktree*, with isolation at filesystem level and
coordination through the **shared task list**. We already have one —
`.claude/worktrees/allo-desktop-work` — but not the shared list that should
accompany it.

---

## 3. The six proposed rules

Each responds to a dated failure in Section 1. None is theoretical.

### R1 — Announce before acting, not afterwards

Before starting work visible to others — a document, page, or service —
**record an intention** in `DEMANDES_JOHN.md` or the other party's mailbox. One
line is enough: *“I am taking X, until approximately this time.”*

*Fixes failure 1.* An announcement costs thirty seconds; the duplication on
29 August cost two sessions two hours.

### R2 — A lease has a duration

> **Addition of 29 August 2026 — inventory before concluding that something is
> absent, and do not duplicate without justification.**
>
> Three false conclusions of absence on the same day, by two agents: the lease
> in `VALIDATED` status for six weeks that we were about to rebuild, the
> `Claude-Session` trailer present in every commit while we claimed not to know
> who had done what, and the Fable5 channel — available through the `fable`
> model — believed to require human triggering. In all three cases the
> capability existed; **nobody had inventoried it**.
>
> Hence `c2c-os/00_manifest/CARTE_DES_CAPACITES.md`: what each agent can
> actually do, with each line carrying the command that establishes it. A
> capability cited without a proof command is classified there as “declared,
> not verified” and **must not be cited as available** in a plan.
>
> **Before building anything:**
>
> ```
> bash ops/deja-fait.sh <mots-cles>    # ca existe deja ?
> bash ops/bail.sh voir                 # quelqu'un est dessus ?
> bash ops/bail.sh prendre <perimetre> <minutes> "<raison>"
> ```
>
> **No redundancy, except in three cases — where it is intentional:**
>
> 1. **Adversarial verification.** Two agents verify the same fact separately.
>    On 29 August, allo and allo-desktop read the same trailers without
>    consulting one another and converged: convergence between two independent
>    measurements made the conclusion robust, not agreement between them.
> 2. **Separation of drafting and criticism.** WRITER and CONTRADICTEUR process
>    the same text deliberately.
> 3. **Backup.** A non-redundant backup system is not a backup system.
>
> Outside these cases, the second construction is wasted work — and worse, it
> **diverges**: two similar artefacts eventually contradict one another, and
> nobody knows which is authoritative. This happened to the hand-written maps,
> which none of the three rediscoveries on 29 August managed to prevent.

> **Amendment of 29 August 2026 — R2 alone cannot work, and this is not a
> question of discipline.**
>
> That day, two sessions built the same page in parallel. Their coordination
> messages **crossed**: “do not build” was sent at **12:24**, and the
> coordination request at **12:26**. Neither ignored the other's message — it
> had not yet arrived.
>
> **The channel is slower than the work it coordinates.** Writing a message,
> committing it, pushing it, and waiting for it to be read takes several
> minutes. Building a page takes ten. R2 requires a round trip when there is
> time only for one direction. It was breached **three times on 29 August** —
> twice by allo and once by allo-desktop. Three breaches on the same day are not
> three lapses of attention: the rule demands the impossible.
>
> **What replaces the announcement: the lease.** We do not request permission;
> we **declare** that we are taking a scope and proceed without waiting for a
> response. The second arrival sees the declaration and stops.
>
> ```
> bash ops/bail.sh voir                                    # avant de commencer
> bash ops/bail.sh prendre page:carte-services 90 "raison"  # refuse si deja pris
> git add c2c-os/02_operational_registers/baux/ && git commit -m 'bail: ...' && git push
> bash ops/bail.sh rendre page:carte-services               # des que c'est fini
> ```
>
> `prendre` **exits with code 1 and refuses** when the scope is already held —
> this is a barrier, not a reminder. Tested against the real scenario of
> 29 August: the second request is rejected with the holder's name and reason.
>
> **Two design choices that matter.** The lease applies to the **scope**, never
> the session: issue `anthropics/claude-code#76727` shows that a lock indexed by
> session can be bypassed by worktrees, and allo-desktop works precisely in a
> worktree. And use **one file per lease**, never a single register: two
> sessions declaring simultaneously write two separate files, creating no
> merge conflict — the same pattern that keeps C2C mailboxes conflict-free.
>
> **What the lease does not do.** It does not eliminate the race: two sessions
> pulling the repository at the same second both see the scope as free. It
> reduces the race from a round trip to a one-way trip. Saying this is better
> than promising a guarantee it does not offer.
>
> The complete policy has existed since **12 July 2026**
> (`RESOURCE_SCOPED_LEASE_AND_STALE_RECOVERY_POLICY_V1.0.md`, status
> `VALIDATED OPERATING DECISION`) and **had never been connected**. The rule
> that would have prevented the duplication on 29 August remained validated
> and dormant for six weeks. `ops/bail.sh` is the connection, not a new rule.
>
> **For colleagues, it is worse without a lease than it is for us**: they do
> not read their mailboxes every fifteen minutes, so the interval between
> announcement and action is measured in hours, not minutes.

An intention **expires**. If its author provides no update beyond the announced
period, anyone may resume the work. An agent that crashes blocks nobody.

*Borrowed from the advisory lease in `mcp_agent_mail`.*

### R3 — An unread REQUEST means someone is blocked

Reading one's mailbox is **the first item** in each work cycle, before any
personal work. Respond, or write why no response is possible. **Never remain
silent.**

*Fixes failure 2.*

### R4 — Name the channel, never the agent

Do not write “K3 is not responding”. Write “nothing in `kimi-k3/outbox`”, and
name **the untested channels**. Absence must never be declared from one
instrument, one directory, or one machine.

*Fixes failures 3 and 4. Details in `ops/bascules/07_VERIFIER_AVANT_DECLARER.md`.*

### R5 — Every timestamp is read, never estimated

Use `date -u '+%Y-%m-%dT%H:%M:%SZ'` for the protocol's `timestamp_utc` field;
use Dublin time for human-readable material. **Never force
`TZ=Europe/Dublin`**: the time-zone database is empty on the PC and returns UTC
disguised as local time, including in mid-July. Without `TZ`, the system gives
the correct result.

*Fixes failure 5.*

### R6 — “Done” requires a proof command

Every task declared complete must carry, in its description, **the exact
command that establishes completion and its expected output** — against the
real output, not declared state. If that command cannot be written, the task is
not complete: it is poorly defined.

*Fixes failure 6.*

---

## 4. Allocation of scopes

Agreed on 28 August, observed since then, to be reaffirmed in writing:

| Session | Scope | Must not touch |
|---|---|---|
| **allo** (`conv-claude-architecture-helper-pc-001`) | SSH access to both servers, credentials, production chains, deployments | workstation, Windows tasks, `reminders.json` |
| **allo-desktop** (`conv-allo-desktop-01`) | graphical interface, local files, `reminders.json`, scheduled Windows tasks | SSH, credentials, production |

**When scopes overlap**: the owner of the scope decides; the other proposes. If
in doubt, ask — it is cheaper than duplication.

---

## 5. Onboarding a human colleague

Use the same sequence as for an agent, plus three things:

1. **An identity** (`conv-<nom>-<projet>-001`) and a mailbox.
2. **Add it to `C2C_ALLOWED_SENDERS`** — otherwise its messages are received
   and **silently ignored**.
3. **Write its scope**: what it may access and, above all, what it may not do.
   Default to the narrowest scope that permits the work.
4. **Test the round trip** with a concrete question. Until that test has taken
   place, the newcomer is not connected.
5. **A tutorial in their language**, and a first exchange that produces a
   useful result. A colleague who gets nothing from the first contact does not
   return.

**What remains untested and must not be asserted**: that a shared Drive file
appears in a colleague's Claude session connector, using **their** account.
Outbound sharing has been proved; receipt has not.

---

## 6. What I recommend, and what I do not recommend

**Adopt now** — R1 to R6. Zero cost; each fixes an observed failure.

**Test before building** — Agent Teams and Channels. If the native facility
does what we are assembling ourselves, continuing to assemble it would be
absurd. **This test has not been performed: I do not yet recommend adoption; I
recommend measurement.**

**Borrow, do not install** — take the two ideas from `mcp_agent_mail` (lease
with TTL and search index), rather than the software. Installing another MCP
server consumes tokens in every message, while our mailboxes work.

**Do not** replace the C2C mailboxes. They are versioned, auditable, and survive
an offline session. None of the solutions found performs better on all three
criteria.

---

## 7. What this policy does not yet say

- **Agent Teams and Channels have not been tested.** Everything about them is
  based on documentation, not measurement.
- **The full human-colleague onboarding flow has never been used.** It will be
  corrected at first real use.
- **No mechanism enforces R1 to R6.** They are rules, not guardrails. A rule
  that can be ignored without consequence eventually is ignored — that is the
  lesson from the ten tools “built but never connected” this week.

These reservations are intentional. A document that states everything with
equal confidence does not show readers what they can rely on.

---

## Sources

- [`Dicklesworthstone/mcp_agent_mail`](https://github.com/dicklesworthstone/mcp_agent_mail) — mailboxes, identities, advisory leases, Git + SQLite
- [Rust version, 34 tools](https://github.com/Dicklesworthstone/mcp_agent_mail_rust)
- [Agent2Agent (A2A)](https://en.wikipedia.org/wiki/Agent2Agent) — v1.0 April 2026
- [MCP vs A2A, 2026 guide](https://dev.to/pockit_tools/mcp-vs-a2a-the-complete-guide-to-ai-agent-protocols-in-2026-30li)
- [Agent Teams, Claude Opus 4.6](https://www.mindstudio.ai/blog/what-is-claude-code-agent-teams/)
- [Git worktrees for parallel agents](https://www.augmentcode.com/guides/git-worktrees-parallel-ai-agent-execution)
- [File-level locking for shared databases](https://dev.to/authora/stop-your-ai-coding-agents-from-fighting-file-level-locking-for-shared-codebases-3pej)
- [Multi-agent system: Anthropic's experience](https://www.anthropic.com/engineering/multi-agent-research-system)

---

## Automations to install for a newcomer

> Added on 29 August 2026. The criterion organising this section: **a
> mechanical discipline runs without conscious effort; a manual discipline
> will be forgotten.** This is not a moral judgement — it is the day's finding,
> when R2 was breached three times by two agents who knew it.

### What is MECHANICAL — installed and protective without effort

These four mechanisms refuse invalid action. A newcomer has nothing to
remember: they encounter the barrier and understand it.

| Automation | What it refuses | Installed by |
|---|---|---|
| `ops/c2c-validate-messages.py` | a message without a header, or dated in the future | push hook |
| `ops/bail.sh prendre` | a scope already held — **code 1** | nothing to install; it is a command |
| `ops/generer-existant.py --check` | an outdated `EXISTANT.md` | push hook |
| backup + **restoration drill** | nothing, but **proves** that the backup can be restored | systemd timer |

The drill deserves emphasis: a backup that has never been restored is a
hypothesis. The 29 August drill passed at 05:15:50 after failing the previous
day — without it, we would have believed the chain to be healthy.

### What remains MANUAL — therefore fragile, and this must be stated

| Discipline | Why it is not mechanised | Observed cost |
|---|---|---|
| Read the mailbox at the start of every cycle | nothing enforces it | a `REQUEST` remained unread for 33 minutes on 29 August |
| Take the lease **before** building | nothing requires it before writing a file | two pages built twice on 29 August |
| Respond to **the group**, not the sender | convention, not mechanism | recreates the bottleneck we are removing |
| R6 — “done” requires a proof command | **declarative; nothing refuses it** | tasks closed without evidence |
| Anti-secret scan before commit | performed manually | a clear-text token entered the index on 29 August |

**Every row in this table is debt.** A manual discipline without a
mechanisation date will eventually be breached — the only question is when.

### A colleague's kit, in order

1. **An identity**, `conv-<nom>-<projet>-001`, and a mailbox.
2. **The two allowlists**, which do not serve the same purpose:
   `C2C_ALLOWED_SENDERS` governs **wake-up**; the worker's `allowed_senders`
   governs **execution**. Being on the first but not the second is the
   **healthy** default: the message wakes the system but triggers nothing.
3. **The three commands**, and nothing else to memorise:
   ```
   bash ops/deja-fait.sh <mots>          # ca existe deja ?
   bash ops/bail.sh voir                  # quelqu un est dessus ?
   bash ops/bail.sh prendre <perimetre> <minutes> "<raison>"
   ```
4. **A real round trip** before declaring the colleague connected. An installed
   channel is not an open channel — the bridge existed for a month without any
   session registering.
5. **The non-negotiable rules**: no secrets on any channel; no irreversible
   action without human validation; never validate one's own
   `human_validation_required`; European evaluation files must never be sent
   to an external API.

### What we do NOT ask a human colleague to do

**Produce the format manually.** A timestamped filename, fourteen-field
header, and registry row: nobody will write it correctly, and the first failed
attempt causes lasting discouragement.

The entry point is `POST /c2c/depose` on the production server, in service
since 29 August: it sets the timestamp, derives the identifier, constructs the
path, validates, commits and pushes. The colleague writes plain text.

**Until an interface calls it**, an agent performs the relay manually — slow,
but explicit. A declared relay is better than a channel that looks open but
silently rejects messages.

---

## R7 — Wait for a condition, not a message

> Mandatory rule added on 31 August 2026 after a real incident on both sides on
> the same day: each waited around twenty minutes for a message from the other
> concerning a fact directly observable in `origin/main` from the moment it
> became true.

**Shared state (a present file, completed merge, or pushed commit) is checked
on demand — it must not be sampled at the pace of the person announcing it.**
Waiting for a message about a fact that `git fetch` reveals in two seconds adds
mailbox latency to a delay that does not otherwise exist.

**The rule**: before continuing to wait for peer confirmation of something
that can be verified directly (a sha, file, or `git log` line), check the
condition yourself at short intervals instead of waiting for the receipt. The
message remains useful — it carries the explanation, measurement, and what is
not in the diff — but **unblocking itself must never depend on receipt of that
message.**

**What counts as observable and what does not**: state on `origin/main` (file,
commit, sha) is observable by both parties without a message. A human decision,
judgement, or anything existing only in the other person's mind or session is
not — R7 does not replace communication; it only removes what did not need to
be requested.

**The symmetrical obligation on the party that stops** (added by Allo,
co-signed): R7 puts the full burden on the waiting party, which is fair but
incomplete — if the party that stops does not name the resumption condition,
the waiting party does not know what to poll and falls back to its mailbox,
which is exactly what happened on 31 August. **Every “I am stopping” message
must carry its resumption condition and deadline**, never merely “let me know”:

> *“I am stopping until `origin/main` contains commit X, or for no more than 15
> minutes.”*

**What happens at the deadline**: waiting without a deadline is a lock that
never releases. At the deadline, resume anyway and say so — a conflict can be
resolved afterwards; a blockage nobody sees cannot. `ops/attendre-condition.sh`
implements this mechanism (`--timeout`); the rule specifies what to do when the
timeout expires.

*Fixes the failure on 31 August: two sessions blocked on each other through a
message, although the awaited condition had been visible for twenty minutes.*

**Tooling**: `ops/attendre-condition.sh` polls Git state (see above); the
`session-bridge` plugin provides a direct signal between two sessions on the
same machine (the bridge carries the signal, while the C2C mailbox remains
canonical and carries the content — never the reverse).

**Trap found while testing the bridge that same day**: a blocking listener
wakes on the OLDEST unprocessed message in its mailbox, not the newest. Both
sessions missed fresh signals while believing they were measuring a live
interrupt, because a backlog of old messages (session-end notifications and
pings) woke first. **Handling rule**: clear already-processed messages from the
bridge mailbox before arming a new listener — otherwise the instrument
confuses old state with a fresh event, exactly the pattern R7 exists to fight,
turned against the tool intended to apply it.

---

## R8 — A conversation without work DECLARES it and receives work

> Mandatory rule added on 29 August 2026 at John's request.

**A conversation with nothing left to do must not stop silently.** It deposits
a `STATUS` message in `groupe-disponibilite`, stating that it is available,
what it can do, and for how long.

```
POST /c2c/depose
{"expediteur": "<mon-identite>",
 "destinataires": ["groupe-disponibilite"],
 "type": "STATUS",
 "sujet": "Disponible -- <ce que je peux prendre>",
 "corps": "Perimetre, competences, duree estimee de disponibilite."}
```

### Why this is a rule, not advice

On 29 August, **two sessions spent part of the afternoon in “active waiting”**
— one waiting for an audit and the other for a response — while the task list
contained several unassigned workstreams. Nobody was blocked: each simply did
not know that the other was available.

**Idle capacity that does not declare itself is indistinguishable from absent
capacity.** This is the same defect as uninventoried capabilities, applied to
time rather than tools.

### What the reader of `groupe-disponibilite` must do

Assign work, or explain why none is assigned. **An unanswered “I am available”
is an unread `REQUEST`** — it falls under R1.

Sources of work, in order:

1. `pending` tasks that depend only on us;
2. disciplines still marked **declarative** in the capability map — each is a
   dated debt;
3. adversarial verification of another person's work: this is the only
   redundancy that yields more than it costs.

### The limit, written down so we do not discover it later

This rule inherits the channel's defect: **nothing wakes
`groupe-disponibilite` either** (task #210). Until group wake-up is connected,
an availability declaration will be seen during each participant's next cycle
— a few minutes for active sessions, overnight for LEAD.

This is better than silence, and it is not yet real-time assignment. Saying so
prevents us from believing the rule is stronger than it is.
