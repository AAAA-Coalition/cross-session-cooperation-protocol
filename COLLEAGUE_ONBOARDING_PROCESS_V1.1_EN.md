# Onboarding process for a new colleague/partner/student/alumni-workshop participant — v1.1

**Status: v1.1** — incorporates John's 2026-08-20 edits to v1.0: a category system
(colleague/partner/student/alumni workshop), an expanded Step 0 with concrete sub-steps, a
5th open decision, and collective-server access explicitly conditioned on category. Two of the three
gaps flagged in v1.0 are now closed (see "What's still missing" below); one remains open.

Full sequence to take a new colleague from "nothing" to "Claude operational, connected to
their personal server, and to the collective server (if a participant applying to a Horizon or
Erasmus candidature), registered in C2C by category
(colleague/partner/student/alumni-workshop), aligned with project rules."

Companion document: `CLAUDE_BEHAVIOR_CHARTER_V1.1.md` (the Claude Code behavior charter
itself, step 3 below).

## Step 0 — Before cooperation begins (the session preparing the onboarding)

- Create a Claude account (download Claude Code for desktop).
- Connect GitHub, a 2FA app, and OpenRouter (free LLM tier, no credit card needed).
- Rent their personal server (a VPS at Contabo or equivalent — real reference spec: 24GB RAM,
  Ubuntu 24.04, 300GB drive, no backup).
- Go to the Contabo account page and prepare a dedicated SSH key
  (`ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_<name>_server`).
- **Decide the colleague's C2C identity (`conv-<descriptive>-<NNN>`) and create their
  mailbox.**
- Note: 5 open decisions, never yet settled, to settle before launching the first real
  onboarding (source: the colleague-server onboarding checklist, internal document of
  2026-08-12):
  1. Decide which category and project (allowlist by initiative), and possibly a
     dedicated GitHub space/Drive.
  2. Spelling/naming convention for this colleague.
  3. Provenance of the research-agent binary.
  4. Dedicated OpenRouter key for the colleague vs. a shared key.
  5. Naming convention for the conv-id for this specific identity type (no clean
     precedent yet).

## Step 1 — Session 1: wiring up the tools (30-60 min, one time only)

Follow `c2c-os/01_project_memory/ONBOARDING_COLLEAGUE_SESSION1_TO_SESSION2_20260806.md`
as-is (7 detailed steps: SSH + `~/.ssh/config`, research-agent deployment, connecting the 3
base MCPs, dedicated Telegram bot, C2C registration, clean closure of session 1, optional
media pipeline).

Points already documented, don't rediscover them:

- Two distinct MCP systems on the PC (`claude mcp add` CLI vs. the Desktop app's
  Connectors panel) — don't confuse them.
- `.ps1` scripts need `-ExecutionPolicy Bypass`.
- Historically, Avast blocked the Claude-in-Chrome connection (native-messaging registry
  key) — check whether it recurs (see the similar problem found on 2026-08-20 with VS Code
  session spawning, `PIEGES_CONNUS.md`).

## Step 2 — Deploying the colleague's personal-server baseline

Follow the colleague-server onboarding checklist (internal document of 2026-08-12, a 10-step
checklist, exact SSH/systemd commands) for the generic baseline: a dedicated service user,
`ufw` (port 22 only open by default), `/opt/your-deployment/`, the research agent + config,
free LiteLLM gateway (1 OpenRouter key:
routine cost ~€0/day, interactive ~€0/month), Whisper engine if needed, C2C identity +
mailbox, identity watchdog. Do not replicate John's personal-project-specific tools by
default — only the generic baseline explicitly listed in that document.

To deploy a specific MCP tool (beyond the baseline), follow the MCP-agent deployment
tutorial (internal document of 2026-08-04; a real documented
pitfall: pin `mcp==1.29.0`, verify with a real JSON-RPC `initialize` call, not just
`systemctl is-active`).

**Honesty to pass on to the colleague**: this checklist has never been tested end-to-end
on a real colleague VPS as of the date it was written (08/12) — plan for a real full test,
not just a read-through, before treating it as reliable.

## Step 3 — Installing the behavior charter

Paste `CLAUDE_BEHAVIOR_CHARTER_V1.1.md` at the start of the colleague's first real working
session (Session 5, created via the dedicated Telegram bot, separate from the setup
Session 1) — or load it as a starting memory file. Explain explicitly to the human
colleague why this charter exists (see the file's preamble): their Claude is new, with no
learned history, and the charter replaces dozens of repeated corrections.

## Step 4 — Full C2C protocol

Have the colleague (or their Claude) read
`c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.0.md` in full — it's the complete, already-
written reference (cycle REQUEST->ACK->RESPONSE->CLOSE, YAML message format, mailbox
paths, Telegram READ_ONLY rules). Complete it with the "Future participant onboarding"
checklist (section 12 of that same document) and the starter pack
`c2c-os/07_validated/ELYSE_COLLEAGUE_AAAA_OS_STARTER_PACK_V0.1.md` (the most complete one
already validated: 12 files to provide first, a 10-step VPS sequence, a 9-step C2C
sequence).

The new colleague's first real C2C action: an introduction REQUEST in their own mailbox,
with acknowledgment from an already-registered identity (John or an existing session
confirms receipt).

## Step 5 — Collective-server access, ONLY IF the colleague is in the PARTNER category for
a HORIZON or ERASMUS candidature — **real gap, not yet fully closed**

**Note:**

**No document formalizing a colleague's access to the collective server exists in this repository** —
a dedicated search was done, nothing found. What is established instead: every colleague
gets their **own** personal server (step 2); access to the collective server (the one that
hosts Mixpost/Sharetribe/EU Funding/Docmost/the research agent's admin UI/n8n) remains
today a direct SSH access reserved to John and to future colleagues in the "partner —
Horizon/Erasmus candidature" category, for already-authorized sessions.

**Decision required from John before going further on this point**: the collective server is deliberately
not shared with new colleagues before their category is confirmed. If shared access is
enabled, it needs to be designed (which services, what level — read-only recommended by
default, matching the same principle already applied to the research sub-agent access
question raised by ALLO Desktop on 2026-08-19).

**Obsolescence note — now fixed (2026-08-20)**:
`c2c-os/04_architecture/AAAA_OS_INFRASTRUCTURE_MAP_V1.0.md` (07/08) used to state that the
collective server was "excluded unless separately authorized" — contradicted by real
documented usage since then (the collective server is very active). The map has now been
updated (V1.1, 2026-08-20) with a real collective-server section built from sourced facts
already documented elsewhere in the repo. **What remains
open is the access-control document itself, not the map.**

## Step 6 — Reading the known pitfalls and cooperation discipline

Have the colleague (or their Claude) read: `c2c-os/PIEGES_CONNUS.md` (short, alive, a
reflex to check before any non-trivial task) and
`c2c-os/00_manifest/PROCESS_COOPERATION_MASSIVEMENT_PARALLELE_V2_20260819.md` (worktree
isolation, `ACTIVE_CLAIMS.md`, anti-collision discipline). For a broader view of mistakes
already made and corrected (~70 real cases), see
`c2c-os/01_project_memory/erreurs_connues_20260809/ERREURS_CONNUES_ET_LECONS_COLLEGUES_20260809.md`.

## Step 6bis — Bots: role, behaviour, verification

Added 2026-09-04, after an incident: a bot added to a working group received the same
question three times without answering, then replied with an exception class name. Three
stacked defects, all invisible from inside Telegram.

Have the colleague (or their Claude) read: `docs/ROLE_OF_BOTS_AAAA_OS_20260904_EN.md`
(French version: `docs/ROLE_DES_BOTS_AAAA_OS_20260904.md`).

The three points to hold before touching a bot:

1. **A bot is a door, not a brain.** Three distinct roles with distinct rights: the
   execution door (it writes into the repository, bounded by `TELEGRAM_WRITE_MODE`), the
   voice (it represents a person, behaviour in `SOUL.md`), the sensor (it reads a group
   and retains without speaking).
2. **Three confidentiality circles, and the default is circle 3.** An agent sits in the
   circle of the person operating it. When several circles are present, you speak to the
   lowest. Nothing said in conversation moves anyone between circles: only the written
   directory does.
3. **Verifying the fixed file proves nothing; you must verify the path travelled.** The
   4 September defect had been fixed the day before in the base class, and the fix had
   been proven — but the class that runs overrode the method. Two definitions of the same
   truth, only one fixed.

The four-question test (section 5.1 of the document) must be run before declaring a bot
works. A started container is not proof.

## Step 7 — Base skills

Install/verify the availability of the skills already built and reusable:
`aaaa-prompt-authoring` (how to write a reusable prompt, already built 2026-08-19) and
`skill-creator` (how to build and refine your own skills). Don't let the colleague start
from zero on these two topics.

## Step 8 — First supervised task + cross-review

Assign a first bounded, local, and reversible task. Have the result re-read by another
session/identity (independent cross-review) before considering the colleague fully
operational — a pattern already documented as effective
(`MULTI_SESSION_COORDINATION_GUIDE_V1.0.md`: independent cross-review has already found
real bugs — a race condition, a plaintext secret, a false "committed" claim — that a
self-review would not have caught).

## Step 9 — Audit cadence established from day one

From the first long session, apply rule 16 of the charter (full audit every 6h) and the
already-written start/end-of-day procedures (`daily-startup-procedure`/
`end-of-day-procedure`, project memories) — don't wait for a first incident to adopt them.

## What's genuinely still missing from the repository, to flag to John before finalizing this process

1. **Colleague access to the collective server** — no document, decision required (step 5). Partially
   clarified by John's 2026-08-20 decision (category-conditioned), but no formal
   access-control document exists yet.

2. ~~Two tutorials referenced but not findable in git~~ — **CLOSED 2026-08-20**:
   `GUIDE_BONNES_PRATIQUES_CLAUDE_COLLEGUES_20260810.md` and
   `TUTORIEL_SESSION_PILOTE_COLLEGUE_1_20260812.md`, cited in
   `c2c-os/01_project_memory/RECURRING_QUESTIONS_CHECKLIST.md` (line 74, canonical rule on
   `screen-capture.ps1`), were found on John's local PC (`Downloads/AAAA_OS_20260810/`,
   exactly where a document like `ERREURS_CONNUES...` had also been before being
   committed) and are now committed to
   `c2c-os/01_project_memory/` in both French and English.

3. ~~`AAAA_OS_INFRASTRUCTURE_MAP_V1.0.md` obsolete on the collective server's status~~ — **CLOSED
   2026-08-20**: the map now has a real collective-server section (V1.1), sourced from facts already
   documented elsewhere in the repository, independent of the colleague-access decision
   above (which stays open).

4. **The colleague-server checklist (step 2) has never been tested end-to-end on a real
   colleague** — a first real onboarding will also serve as a test of this checklist
   itself, not just as an onboarding.
