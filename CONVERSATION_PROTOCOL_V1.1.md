# C2C Conversation Protocol v1.1

Status: DESIGN ACCEPTED / RUNTIME CUTOVER PENDING
Date: 2026-07-12
Supersedes v1.0 for new repository-only message handling after live cutover; v1.0 remains historical and current-runtime compatible until deployment is proven.

## 1. Core lifecycle

```text
REQUEST → ACK → RESPONSE → CLOSE
```

Optional types remain `STATUS`, `CONFLICT`, `DECISION_PROPOSAL` and `DECISION_CONFIRMATION`.

No agent may convert its recommendation into a human decision.

## 2. Canonical authority

1. mailbox message artifact;
2. segmented Message Registry v2 shard;
3. v2 index and correlation summaries;
4. historical v1 monolith;
5. Telegram intake/notification journal.

GitHub remains canonical. Telegram remains READ_ONLY/non-canonical.

## 3. Stable identity

Every conversation/agent has one stable `conversation_id` and one active operational writer at a time.

Successor conversations must:

- use a new identity;
- read the predecessor bundle;
- send ACK and RESPONSE;
- prove readback and registry linkage;
- retain predecessor rollback until CLOSE;
- never reuse the predecessor identity.

## 4. Message format

Messages retain YAML front matter with:

- schema version;
- immutable message ID;
- correlation ID;
- UTC timestamp;
- sender and recipient IDs;
- project ID;
- type and status;
- priority and subject;
- `responds_to_message_id` where applicable;
- human-validation flag;
- source/evidence paths where applicable.

Body sections should be limited to objective, inputs, requested actions, constraints, output, sequence and open decisions.

## 5. Canonical paths

```text
c2c-os/03_handoffs/mailboxes/<conversation_id>/inbox/
c2c-os/03_handoffs/mailboxes/<conversation_id>/outbox/
c2c-os/03_handoffs/mailboxes/<conversation_id>/archive/
```

Filename:

```text
<timestamp>_<TYPE>_<message_id>_<sender>_to_<recipient>.md
```

## 6. Message Registry v2

New messages are registered in daily append-only shards:

```text
c2c-os/02_operational_registers/message_registry/YYYY-MM-DD.md
```

Index:

```text
c2c-os/02_operational_registers/MESSAGE_REGISTRY_V2_INDEX.md
```

The historical monolith is not rewritten after v2 cutover.

The write transaction must preflight, create mailbox artifact, append shard row, read back both and compensate on partial failure.

## 7. Resource-scoped execution leases

C2C message exchange and runtime execution use resource-scoped leases, not broad whole-host locks.

A C2C service mutation lease blocks conflicting C2C mutations but does not block unrelated read-only repository research.

Every lease has owner, resource scope, mode, heartbeat, expiry, checkpoint and status. Stale leases become `STALE_UNVERIFIED`, not PASS.

## 8. Runtime versus messaging

- C2C Messaging owns identity, mailboxes, registry, correlation and lifecycle.
- Runtime/Mission Scheduler owns detection, claim, dispatch, retries, human gates and execution.
- ChatGPT conversations are human-facing specialist workspaces.
- Persistent work is performed by real external executors.

A mailbox write does not prove automatic runtime orchestration.

## 9. Security

- no secrets in messages, registries or evidence;
- no unrestricted shell;
- no infrastructure authority from a message alone;
- no Telegram-based approval for sensitive action;
- no public port, DNS or firewall change without separate human approval;
- no budget/publication/submission decision without human authority;
- preserve hashes, IDs, provenance and rollback.

## 10. Acceptance

Protocol v1.1 reaches live PASS only when:

1. sender identity and allowlist pass;
2. one-character and normal-size messages register in v2 without monolith rewrite;
3. mailbox and shard readback pass;
4. duplicate ID is idempotent;
5. registry/shard failure compensates safely;
6. REQUEST→ACK→RESPONSE→CLOSE completes across two distinct identities;
7. restart/duplicate and stale-lease tests pass;
8. rollback from v2 to v1-compatible runtime is evidenced;
9. Telegram remains READ_ONLY and no public listener is introduced.

Until then, status is repository design/preparation only.

## 11. Amendment v1.2 -- Telegram write-mode (2026-08-22)

**Explicit human decision by John, 2026-08-22** (relayed via the interactive decisions
page and confirmed directly in chat: "Telegram write-mode reformulé : GO ! (C2C_REQUESTS)").
This amends, not silently rewrites, §2 and acceptance criterion §10.9 above -- both remain
as written for their historical period (v1.1, DESIGN ACCEPTED / RUNTIME CUTOVER PENDING,
predating this decision).

**What changes**: the Telegram bot moves from `READ_ONLY` to `C2C_REQUESTS` mode (one of
three modes already implemented in `services/c2c_gateway/app/telegram_poller.py`, the
other being `DRAFT_ONLY`). In this mode, a message the bot sends becomes a real C2C
REQUEST (`status: SENT`) discoverable by the autonomous worker -- not a draft requiring a
separate approval step.

**What does NOT change** (§9 Security is not amended by this decision):

- "no Telegram-based approval for sensitive action" remains in full force -- this mode
  change lets the bot *produce* REQUESTs, it does not make Telegram a channel through
  which a sensitive action can be *approved*. A REQUEST originating from Telegram still
  needs the same human-validation gate as any other REQUEST before a sensitive action
  executes.
- No secrets in messages; no unrestricted shell; no infrastructure authority from a
  message alone; no public port/DNS/firewall change without separate approval; no
  budget/publication/submission decision without human authority. All unchanged.

**Prerequisites before cutover** (not yet done as of this amendment, tracked in
`knowledge/ROADMAP_20260822.md`):

1. Loosen the 3 fail-closed gates that currently assert/require `READ_ONLY` --
   the production server's `c2c-mailbox-watcher.py`, `services/c2c_autonomous_worker/worker.py`,
   `ops/c2c-registry-growth-r1/runtime_acceptance.py` -- so they accept `C2C_REQUESTS`
   without disabling the protections those gates otherwise enforce.
2. Activate the poller service on the secondary server (currently inactive) with the new mode.
3. Add publication logging (timestamped, queryable record of everything the bot
   publishes) -- committed to by Allo, not a restriction, a post-hoc audit trail.

Execution: VPS-side (poller activation, gate changes) is Allo's, this document amendment
is mine -- coordinated via C2C (`CORR-ROADMAP-20260822-01` thread).

## 12. Amendment v1.3 -- Cooperation checkpoints between sibling identities (2026-08-24)

**Explicit human decision by John, 2026-08-24** ("GO avance" on a proposal both
`conv-allo-desktop-01` and `conv-claude-architecture-helper-pc-001` had agreed on and
declined to write unilaterally -- same discipline as §11). Grew out of a real day: a
mailbox-watcher misconfiguration left messages genuinely unread for hours, and two
near-misses (a session almost distrusted its own uncommitted work as an unknown threat;
a `git checkout --` almost erased four unpushed log entries) showed informal practice
alone wasn't enough.

**What this adds**: five checkpoints at which a sibling identity sends a STATUS/REQUEST
rather than staying silent until asked. These formalize a practice both identities were
already following ad hoc on 2026-08-24, not a new invention:

1. **Before starting large background work on shared infrastructure** (a broad audit, a
   structural fix) -- a short STATUS announcing scope, sent before starting, not after.
2. **As soon as a result contradicts a hypothesis the other sibling might hold as true**
   (e.g. "0 errors" that turns out false, an active outage found) -- sent immediately,
   never batched to end of session.
3. **Before touching a file or service the other sibling might also touch** (shared
   config, a common VPS service) -- check first; this is the anti-collision discipline
   already practiced throughout 2026-08-24.
4. **The standing 2-hour cross-check heartbeat** (established 2026-08-21) remains the
   floor even when nothing major is happening -- an explicit "nothing new" rather than
   silence, so silence itself stays informative.
5. **Never leave uncommitted work in the shared tree** -- not even a draft, not even
   broken. A commit on a branch is always distinguishable from corruption; an uncommitted
   file in a tree two identities share is not. Proposed by Allo after two real incidents
   the same day (own work misread as a threat by her own later-woken instance; a rebase
   `git checkout --` that nearly destroyed unpushed log entries, survived only because a
   copy existed by chance).

**What this does NOT change**: these are checkpoints for keeping siblings informed, not a
new approval gate. §9 Security is untouched -- a checkpoint STATUS from one sibling is
never itself authorization for a sensitive action; the human-validation requirements
elsewhere in this document still apply in full.

**On tooling**: native Claude Code cross-session messaging (`ListAgents`/`SendMessage`,
and `notify_when_idle` for a one-shot "tell me when you finish") may eventually carry the
low-latency traffic these checkpoints generate between siblings on the same machine, once
both identities run a Claude Code version that supports it (>= v2.1.224, >= v2.1.234 on
native Windows -- unconfirmed on both sides as of this amendment). The C2C git-mailbox
remains canonical regardless: for anything durable, audited, or visible to Bello, to either
server, or to John, native messaging is a supplement, never a replacement.

Execution: both identities already follow this; this document amendment records the
decision -- coordinated via C2C (`CORR-COOPERATION-PROTOCOL-20260824-01` thread).

## 13. Amendment v1.4 -- Daily rhythm across all three identities (2026-08-25)

**Explicit human decision by John, 2026-08-25** ("GO" on a proposal converged between all
three identities -- `conv-allo-desktop-01`, `conv-claude-architecture-helper-pc-001`,
`conv-claude-pc-bello-001` -- same discipline as §11 and §12: none of the three wrote it
unilaterally.

**Motivated by a real finding**: the two scheduled tasks meant to trigger this rhythm
automatically (`AAAA-Bilan-Allo-Matin`/`-Soir`) have failed silently since 2026-08-03
(`LogonType: Interactive`, requiring an open session at trigger time, vs. `S4U`, which does
not). A partial fix (LogonType -> S4U) has been applied to 4 tasks; one (the task that
syncs conversation transcripts to the production server) still fails post-fix, cause not
yet found. Every
"daily" bilan run so far happened because John was present, not because a clock called it.
This amendment defines the rhythm that must hold **even without the crons**, and improve
once they are restored.

**Morning** (or, for a cron-cycle identity with no fixed "day", folded into every wake
instead of a once-daily trigger -- see the Bello adaptation below): the procedure already
running as `wake-bilan-session.ps1` (Allo, since 2026-08-03), adopted verbatim by all three
identities:

0. Rappels d'abord -- read `reminders.json`, overdue `pending` items lead the bilan, never
   buried, status never self-changed by this pass.
1. Verify by real commands, never assumed -- `systemctl --failed` on both servers (root AND
   user scope -- a user-scope failure such as the voice-bot gateway unit, which runs under a
   user account, is invisible to a root-only check), key services, C2C mailboxes (any `human_validation_required: true`),
   git divergence, `switch.md` new entries, and local scheduled tasks (added 2026-08-25
   after a real same-day gap where server infra was checked but local Windows tasks were
   not).
2. Save -- commit and push all finished local work. Never `services/proposal_draft/
   status_api.py`, never a secret.
3. Memory -- update `MEMORY.md` and dedicated files with what actually changed.
4. Binary rule -- read/analysis/notes in `switch.md` = authorized. Anything touching shared
   infra beyond an already-validated commit = precise plan marked
   `<ATTEND CONFIRMATION HUMAINE>`, not executed.
5. Concise bilan in `switch.md` -- what was verified, saved, updated, and the proposed
   program. Never record an unverified completion.

**Midday (~14h, or folded into each cycle for a cron identity)**: three questions only --
(1) what was announced this morning, where does it *really* stand; (2) any uncommitted work
in the shared tree (the 5th trigger, §12); (3) a blocker waiting on John for more than 4h
that he does not know about. If all three are "nothing," one explicit line says so -- an
explicit "nothing" beats silence.

**Evening**: full bilan -- what was done *and proven* (not "launched"), what failed and
why, what is waiting on John with its deadline, memory updated. Same hard rule as point 5.

**Night**: nothing interactive. Autonomous loops run; anything that fails must signal
itself via the sweep/hook, not wait for morning. A night task needing arbitration writes
`<ATTEND CONFIRMATION HUMAINE>` in `switch.md` and does nothing else.

**Standing frequency**: §12's five checkpoints plus the 2-hour cross-check floor remain in
force -- a hook failing silently looks exactly like a colleague with nothing to say.

**Three harder rules, proposed the same day and accepted by all three**:

1. **One source of truth for done/to-do.** Real, current gap found by Bello while applying
   this very procedure: `reminders.json` still listed EUACC.ai and the Mistral candidature
   as `pending` a full day after John had moved both to 2026-09-01 (commit `aa847225`) --
   the decision changed, the tracking file did not. Not resolved today: escalated to John
   as a decision (which of the seven current tracking systems is authoritative), not a
   project any identity starts unilaterally.
2. **Edit the source, never the generated product.** Extends the Glance rule already agreed
   (edit the repo file, signal the change, the VPS-access identity pushes it live) to every
   generated surface -- a generated `.html` dashboard is silently overwritten on each
   regeneration, so a direct edit to it is lost without warning.
3. **Audits only on a precise question, never "just to check."** Self-corrected the same
   day: an open-ended verification cost real time without producing anything shippable,
   confirming John's own observation that every open-ended audit tends to produce ten new
   findings that each become a task.

**Cron-cycle adaptation (Bello)**: no fixed "day" exists for an identity that wakes on a
short cron cycle, sometimes several times an hour, sometimes with multi-hour gaps. Rather
than force the four fixed time-blocks onto a shape that does not fit:

- The morning procedure (full verification) runs at *every* wake, not once daily.
- Midday's three questions are answered implicitly in each `switch.md` entry, including
  when every answer is "nothing".
- The evening bilan fires only when a cycle produces something substantial to report, not
  on a fixed clock.
- Night does not apply distinctly -- cycles do not stop overnight.

A named, accepted risk of this adaptation: re-running `systemctl --failed` and
`reminders.json` at every wake has a real cost if the cycle frequency increases -- worth
watching, not a problem as of this amendment.

Execution: all three identities had already converged via C2C
(`CORR-RYTHME-JOURNEE-20260825-01` thread) before this document amendment records the
decision.
