# Deploy Communication Channels with a Colleague

> **Version dated 03/09/2026.** Written at John's request: *“update the
> deployment tutorials for the 7–8–9 communication channels with colleagues
> (Telegram first)”*.
>
> **This tutorial describes only what was measured as operational on 02/09 at
> 23:00.** Each channel shows its actual status and verification command.
> Announcing a channel that does not work is more costly than having no
> channel: the colleague waits for a reply that will never come.

---

## The Nine Channels — John's List and Their Actual Status

John listed them on WhatsApp on 01/09 (17:23 → 17:38). The status on the right
is measured, not assumed.

| # | Channel | What it enables | Status on 02/09 |
|---|---|---|---|
| 1 | **MCP click & browse** | Claude acts on the PC: clicking, typing and screen reading | ✅ active (`desktop-control`) |
| 2 | **Chrome** | controlled browsing and page extraction | ✅ through the same MCP |
| 3 | **BRIDGE** (session-bridge) | Claude Code ↔ Claude Code on the same machine | ⚠️ **same account only** |
| 4 | **Anthropic conversation-to-conversation** | subagents and `SendMessage` | ✅ active |
| 5 | **C2C — 3 layers** | GitHub mailboxes, 11 identities | ✅ **reference channel** |
| 6 | **Google Drive** | document sharing with humans | ✅ connector active |
| 7 | **A2A** | inter-organisation agent-to-agent protocol | ⛔ not connected here |
| 8 | **Direct SSH between VPSs** | production server ↔ secondary server, including PostgreSQL tunnel | ✅ active |
| 9 | **Telegram** | notification to a human, PC or Android device | ⚠️ **RECEIVE ONLY** |

---

# 1. TELEGRAM — Handle It First and Understand It Before Deployment

## What to Tell the Colleague in the Very First Sentence

**Telegram currently works in only one direction: it receives.** A colleague
can be notified; they cannot reply and be heard.

```
TELEGRAM_WRITE_MODE        = READ_ONLY
MCP_TELEGRAM_WRITE_ENABLED = false
```

This is **not a fault to repair**; it is a project security decision. An agent
that writes to Telegram can contact humans outside the system. The rule in
`CLAUDE.md` prohibits this without explicit human validation.

> **Verify it yourself, in ALL containers — not just one:**
> ```bash
> ssh <serveur-de-production> "sudo -n bash -c 'for C in \$(docker ps --format \"{{.Names}}\" | grep -i telegram); do
>   echo \"--- \$C\"; docker exec \$C printenv 2>/dev/null |
>   grep -E \"^(MCP_TELEGRAM_WRITE_ENABLED|TELEGRAM_WRITE_MODE)=\"; done'"
> ```
>
> **Three traps, all encountered on 02/09:**
>
> **1. These variables are not in the systemd units.** Looking for them there
> gives a false negative. They live in the poller's Compose overrides
> (`docker-compose.poller.yml`, `.agent.yml`, `.local-llm.yml`,
> `.c2c-write-v1.yml`).
>
> **2. Reading the configuration is not enough**: you must read the
> **process environment**. A correct configuration that was never applied to
> the running stack is exactly what blocked LEAD02's channel for days.
>
> **3. Most importantly, the two containers do NOT report the same value.**

### ⚠️ Correction of 03/09: The Control Is NOT Uniform

I had written here that Telegram was read-only, “verified by `printenv`”.
**I had queried only one of the two containers.** The complete measurement is:

| Container (production server) | `MCP_TELEGRAM_WRITE_ENABLED` | `TELEGRAM_WRITE_MODE` |
|---|---|---|
| `...-telegram_mcp-1` | `false` ✅ | `READ_ONLY` ✅ |
| `...-telegram_poller-1` | `false` ✅ | **`C2C_REQUESTS`** ⚠️ |

**Why**: the `docker-compose.c2c-write-v1.yml` override enforces
`READ_ONLY`, but **it applies only to the MCP container**. The poller starts
with `--env-file .env.poller`, which contains
`TELEGRAM_WRITE_MODE=C2C_REQUESTS`.

**What this changes, and what it does not change — the distinction matters:**

- **Writing *to Telegram* remains disabled.**
  `MCP_TELEGRAM_WRITE_ENABLED=false` in both containers. No agent can send a
  Telegram message.
- **What changes**: a `/handoff` command **typed in Telegram by John** creates
  a C2C message with status `SENT` instead of `DRAFT`. This is not an
  autonomous agent; it is a tool obeying an identified human
  (`ALLOWED_TELEGRAM_USER_IDS`, one person in one group).

**Therefore, this is not a violation of `CLAUDE.md`, but a control described
as fully implemented when it is only partly so.** John must decide whether
this is intentional.

> **The lesson applies to this entire tutorial**: checking one container and
> drawing a conclusion about the service is the same failure as scanning
> `.env` and drawing a conclusion about secrets. *The gap lies in where you
> look.*

## Deploying Reception — What Works Today

**Step 1 — the colleague joins the channel.** Nothing needs to be installed on
their side: Telegram on a phone or PC is enough. John adds them to the relevant
group.

**Step 2 — verify that the poller is alive.** A poller showing `Up` while
receiving nothing is the classic silent failure.

```bash
ssh <serveur-secondaire> "sudo -n docker ps --filter name=telegram --format '{{.Names}}  {{.Status}}'"
ssh <serveur-secondaire> "sudo -n docker logs --since 1h c2c-telegram-poller-telegram_mcp-1 2>&1 | tail -20"
```

`Recoverable poller error: ReadTimeout` messages on `getUpdates` are
**normal** — this is the usual long-polling pattern, not a fault.

**Step 3 — accept only the proof.** Do not conclude “it works” until **a real
message has arrived**, rather than merely seeing a green container.

## Target Architecture for Bidirectional Communication — John's Design

John wrote it on 01/09 at 17:36:

```
claude ──── subagent (agent de recherche + LLM gratuit) ──── telegram (PC ou Android)
```

**This is not “enabling Telegram writes”.** It inserts a subagent that assumes
responsibility for outgoing content. The security question changes from
*“do we authorise writing?”* to *“which intermediary is accountable for what
is sent, and within which boundaries?”*

**What this requires, and what does not yet exist**: an allowlist of senders,
bounded message types and a prohibition on the agent issuing its own
confirmation of a human decision — exactly the three disciplines that make
the C2C channel safe today.

**This cannot be deployed this month.** Tell the colleague plainly.

---

# 2. C2C — The Reference Channel and the Only Truly Bidirectional One

This is the channel that works best: **40 messages exchanged in 12 hours** on
02/09 between four different entities.

## How It Works in One Sentence

A mailbox is a directory in the GitHub repository. Writing a message means
depositing a Markdown file in the recipient's `inbox`. Reading it means
reading the directory. **Git is the transport; there is no server to
maintain.**

```
c2c-os/03_handoffs/mailboxes/<destinataire>/inbox/<horodatage>_<TYPE>_<id>_<de>_to_<vers>.md
```

## The 28 Mailboxes, Including 6 Thematic Groups

```bash
cd /chemin/vers/votre/clone && ls c2c-os/03_handoffs/mailboxes/
```

Groups: `groupe-architecture`, `groupe-candidatures`,
`groupe-infrastructure`, `groupe-accueil-collegues`,
`groupe-disponibilite`, and one group dedicated to the current application
(`groupe-<nom-de-la-candidature>`).

> **Design gap reported by John and not yet corrected:**
> *“thematic groups mixing humans and conversations”*. The current six groups
> contain **agents only**. They were intended to mix humans and conversations.
> This is a correction to carry forward, not a minor detail.

## The Two Ways to Write and Their Actual Difference

| Route | Who uses it | Registry |
|---|---|---|
| **bounded MCP tool** | LEAD02 and allowlisted identities | row written **automatically** |
| **direct Git deposit** | me and allo-desktop | row to be written **manually** |

**Both are legitimate.** On 02/09 I stated that the Git route “cannot
register” — **that was false**, and LEAD02 correctly reclassified it: *the
channel can; my procedure omitted the step*. allo-desktop has always
registered its messages.

**The invariant that must hold, whichever route is used:**

```
1 mailbox file  ↔  1 message_id  ↔  1 registry row
```

The registry is located at
`c2c-os/02_operational_registers/MESSAGE_REGISTRY.md`.

> **Measured counting trap**: the pattern `MSG-[0-9]{20}` **ignores 112
> rows** — every identifier containing letters. Count with `^| *MSG-`.
> One known **collision** remains: `MSG-20260821140000000084` identifies two
> different messages with reversed senders and different correlations. It
> must be repaired under a single-writer rule.

## The Coordination Rule That Was Missing

At 20:55 on 02/09, allo-desktop and I wrote our evening reports to `switch.md`
**at the same time**. This caused a rebase conflict in a 6,000-line file.

> **For every high-traffic shared file** (`switch.md`,
> `MESSAGE_REGISTRY.md`, maps): **announce before writing** in
> `groupe-infrastructure`. Three lines. That is less costly than a rebase
> conflict.

## Which Channel to Use for What

| Nature of the exchange | Channel |
|---|---|
| request to **one** conversation | its individual mailbox |
| subject concerning **several** participants | the thematic **group** |
| personal activity log | `switch.md`, dated and signed section, **append only** |
| alert to a human | Telegram (receive only today) |
| **an established fact that others will reuse** | **the repository, not a message** |

**The last row is the most important.** A message gets lost; a file can be
cited. Several important measurements from 02/09 exist only in C2C messages,
which makes them unusable by a third party arriving tomorrow.

---

# 3. SESSION-BRIDGE — Useful, but Know Its Limit

It enables two Claude Code sessions on the **same machine and same account** to
communicate.

```bash
bash "$HOME/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts/list-peers.sh"
```

**The important limitation**: it cannot reach an entity hosted elsewhere.
LEAD02 runs at OpenAI — the bridge will never reach it. For a remote colleague,
it is **C2C or nothing**.

Another measured limitation: **bridge registration expires**. A bridge that
“worked yesterday” but no longer replies is not broken; it is deregistered.
Run `register.sh` again.

---

# 4. GOOGLE DRIVE — The Channel for Non-technical Humans

This is the only channel that requires **nothing** from the colleague: they
receive a link and open it.

Active connectors: file search, reading, creation and sharing.

**Non-negotiable project constraint**: EU evaluation folders must not be sent
to external APIs. Drive is for shared working documents, not sensitive
materials.

> **Dated trap**: our Google Drive OAuth application must remain
> **published**, otherwise the token expires every seven days (task #208).

---

# 5. SSH BETWEEN MACHINES — The Invisible Channel

Production server ↔ secondary server, with a PostgreSQL tunnel on port 15432.

```bash
ssh <serveur-secondaire> "systemctl is-active aaaa-pg-tunnel.service"
ssh <serveur-secondaire> "ss -ltnp | grep 15432"
```

**Secret-handling discipline, learned the hard way**: never put a value on the
command line — `sudo` logs the entire command. Copy file by file; never use
`echo`, and never put a variable inline. To compare two secrets without
displaying them, compare their fingerprints:

```bash
sha256sum < fichier_a | cut -c1-12    # puis comparer les deux empreintes
```

---

# 6. THE PC CONTROL MCP — The Most Powerful and the Most Constrained

*“ALLOW CLAUDE TO ACT ON YOUR PC”*, in John's words.

It enables clicking, typing, reading the interface tree, using the clipboard
and taking screenshots. **This is the channel that can act in someone's
place**, and therefore the one requiring the greatest caution: what it touches
belongs to the human.

Order of preference when several routes exist, from best to worst: application
connector → script (PowerShell) → accessibility tree
(`find_element`, `click_element`) → **coordinate-based clicking only as a
last resort**.

---

# 7. A2A — Announced, Not Connected

An inter-organisation agent-to-agent protocol. **Nothing is deployed here.**
Presenting it to a colleague as available would be false.

It is linked to task #100 (*accelerate agent-to-agent access in response to
EUACC*) and remains at the design stage.

---

# Recommended Deployment Order for a New Colleague

| Day | Channel | Why this order |
|---|---|---|
| **1** | Telegram reception | zero installation, immediate value, expectations framed from the outset |
| **1** | Google Drive | document sharing without technical skills |
| **2** | C2C | the real working channel — requires understanding mailboxes and the registry |
| **3** | SSH | only if the colleague operates a machine |
| **4** | PC control MCP | only on their own machine and with their explicit agreement |
| — | session-bridge | not useful for a remote colleague |
| — | A2A | nothing to deploy |

---

# The Verification Standard That Applies to Every Channel

Our internal scale, and the only row that matters:

| Level | What it means |
|---|---|
| S0 | written |
| S1 | connected |
| S2 | **actually produces** locally |
| S3 | **monitored** — a failure is visible |
| **S4** | **received and usable by the recipient** |

**A channel must not be announced to a colleague until it reaches S4.** On
02/09, three components were at S2 — they really produced output — yet nothing
arrived: the group mailboxes that nobody read, the silent-thread detector that
nothing invoked, and LEAD02's channel whose configuration had not been applied
to the running stack.

> **In all three cases, a report saying “it produces” would have been accurate
> and misleading.**

---

## Version History

| Date | Change |
|---|---|
| 03/09/2026 | Initial measured version. Telegram is presented as receive-only; the two-container discrepancy is documented; C2C is the reference bidirectional channel; non-operational target channels are labelled explicitly. |
