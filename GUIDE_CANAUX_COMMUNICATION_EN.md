# Communication Channels Guide — Claude Sessions and Colleagues

**Written on 28 August 2026**, after half a day spent discovering that three
channels existed, none was connected, and we were communicating by copying
screenshots between sessions.

This document explains **which channel to use and when**, **how to connect a new
session**, and **how to verify that communication actually flows** — because
each of these channels remained installed but unused for weeks.

---

## 1. The three channels, and which one to choose

| Channel | Scope | Latency | Survives an offline session | Attachments | Record |
|---|---|---|---|---|---|
| **session-bridge** | same machine | real time | **no** | no — text only | no |
| **C2C mailboxes** | via GitHub, anywhere | minutes | yes | **no** — text only | yes, versioned |
| **`DEMANDES_JOHN.md`** | via GitHub, anywhere | next handover | yes | no | yes, versioned |
| **Google Drive** | anywhere, including humans | **under one minute** *(measured)* | yes | **yes**, including large binaries | outside the repository |

**The selection rule fits into four lines.**

- A response is needed **now** and the other session is running → **bridge**.
- It must **leave a record** and be read later if necessary → **C2C mailbox**.
- **John** asks **everyone at once** → **`DEMANDES_JOHN.md`**.
- A **real file** must be transferred — PDF, video, spreadsheet, or something
  intended for a person outside the repository → **Google Drive**, accepting
  its additional latency.

These channels do not compete. A question sent through the bridge to a sleeping
session is **lost**; the same question placed in a file waits until it wakes up.

### How to transfer a file nonetheless

A C2C message is **text only, with no attachment**. The repository itself is a
file channel, however: commit the file somewhere in the repository and include
**its path** in the message. The other session reads it after a `git pull`.

This is the correct method for anything versionable — code, documents and
working spreadsheets. Keep Google Drive for material that does not belong in a
repository: large binaries, videos, or a deliverable intended for a person who
does not have GitHub access.

### Google Drive — tested on 28 August: what works and what does not

The channel was tested for real between allo and allo-desktop, using a content
canary (`POMMIER-4471`) to prove that the **content**, not merely the title, had
been read.

**What has been proved:**

- **Writing**: yes, from both sides. “The native connector is read-only” is
  **false**.
- **Propagation**: under one minute. Two files created at 13:18:07Z and
  13:18:20Z were immediately readable. The description “slow” was an unmeasured
  hypothesis, and it was false.
- **Sharing with a non-Google address**: yes. `share_file` with a Hotmail
  address in the `reader` role was verified by reading the permissions back. A
  session can therefore grant a colleague access **without action by John**.

**Connector limitations and their workarounds:**

1. **The connector cannot edit a file in place.** `update_file` only handles the
   title and parent folder — its schema says: *“currently only title and
   parent_id are supported”*. This was confirmed on both sides, so the limitation
   belongs to the connector, not to a session. **It is solved with rclone**; see
   below.
2. **Conversion damages content.** By default, a `text/plain` file is converted
   into a Google Doc: `1.` becomes `1\.` and blank lines are doubled. The
   `disableConversionToGoogleType: true` flag preserves the file exactly.
3. **Reading `text/plain` requires the correct tool.** `read_file_content`
   returns **an empty string** because `text/plain` is not among its supported
   types. But **`get_file_metadata` returns the complete content** in its
   `contentSnippet` field. Therefore: write with
   `disableConversionToGoogleType: true`, then read with `get_file_metadata`.
   This preserves both fidelity **and** readability.

   *I initially wrote here that we had to choose between “readable and
   distorted” and “faithful and silent”. That was wrong: I had tested a single
   reading tool and inferred a property of the file. A tool that cannot read a
   file is not proof that the file is unreadable.*

### In-place writing and editing — rclone, installed and proved on 28 August

`rclone` v1.75.0 (winget `Rclone.Rclone`), remote `gdrive:` on the team Google
account, with the full `drive` scope. **It is not an MCP server**: it is an
executable. It loads no tool definition into the context and therefore **costs
zero tokens per turn** — unlike a Google Workspace MCP server exposing more
than 120 tools in every message.

```bash
rclone lsf   "gdrive:<dossier-canal-sessions>/"        # lister
rclone cat   "gdrive:<dossier>/<fichier>"              # lire, fidèle
rclone copyto <local> "gdrive:<dossier>/<fichier>"     # ÉCRIRE EN PLACE
rclone lsjson "gdrive:<dossier>" --files-only          # ids, tailles, dates
```

**Proved, not inferred**: read, add a section, and rewrite. Before and after,
the Drive identifier is **the same** (`16QhAmEeKhSU…`), `createdTime` is
**unchanged**, only `modifiedTime` changes, and the size grows from 3,226 to
3,472 bytes. This was verified using two independent instruments — `rclone lsjson`
and the connector's `get_file_metadata`. The file was **modified**, not
recreated: existing links and shares survive.

**Our own OAuth client and the deadline it creates.** rclone initially used
Google's shared `client_id`, scheduled for removal “during 2026”. On 28 August,
we replaced it with our own: a dedicated Google Cloud project, a “Desktop app”
client, and credentials in `rclone.conf`. In-place writing was **proved again
after the change** — same Drive identifier, 3,472 → 3,631 bytes.

**The avoided trap was serious.** An OAuth application left in **“Testing”**
publication status has refresh tokens that **expire after seven days**. We
would have exchanged an unclear 2026 deadline for a weekly outage. The
application was **moved to production on 28 August** — the screen shows “In
production” and offers “Back to testing”. This channel now has no deadline.

Publication required a home page, a **privacy policy**, and **terms of use**.
They were written, deployed and verified: `/legal/privacy.html` and
`/legal/terms.html` on the Coalition's public website, served by the secondary
server's Caddy from `/opt/<deploiement>/legal`.

Fallback retained: `rclone.conf.bak-20260828` contains the configuration using
the shared client.

**Two lessons learned here.** “It is fixed” is only true after checking what
the fix introduced: the first fix created its own dated failure. And **a stale
screen is not a failure** — the console still showed “Testing” after
publication; refreshing before drawing a conclusion prevented a false failure
report.

**What has not been proved and must not be inferred:** both sessions use the
**same Google account**. Whether a shared file appears in the Claude Drive
connector of **a colleague using their own account** remains to be tested.
Receiving access and seeing the file through the connector are two different
things.

**An error not to repeat.** I previously wrote that discovery was asymmetric —
that the other session's file did not appear in my `list_recent_files`. That
was false: allo-desktop showed that both files appear when
`orderBy=lastModified` is forced. This was an artefact of the default ordering,
not a property of the channel. The seventh measurement artefact in two days.

---

## 2. The bridge — real time between sessions on the same machine

Plugin `session-bridge` v0.1.1, by Shreyas Patil.
Marketplace: `https://github.com/PatilShreyas/claude-code-session-bridge.git`

### Check that it is installed

**Beware of the trap**: `ListPlugins` and `SearchPlugins` query the **cloud**
catalogue on claude.ai, not the CLI's **local** plugin system. They return empty
results even when the plugin is installed. On 28 August, this confusion led to
the false conclusion “I do not have the plugin”, although it had been there for
a month.

The real test:

```bash
ls ~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts
```

### Connect

```
/bridge start          # s'enregistrer, renvoie un identifiant court
/bridge peers          # voir qui est actif
/bridge connect <id>   # se connecter à l'autre (démarre le pont si besoin)
/bridge ask <question> # poser une question et attendre la réponse
/bridge listen         # se mettre en écoute permanente
```

### The expiry trap

**A registration expires.** On 28 August, an identifier announced at 14:00 no
longer existed ten minutes later, and the peer session was looking for a ghost.

Therefore, **always run `/bridge peers` again immediately before giving an
identifier to someone**, and provide it again if it has changed. An identifier
communicated from memory is a false identifier.

### Verify that messages actually flow

One ping is enough, and it must be performed — a “connected” bridge carrying no
messages looks like a working bridge.

```bash
S=~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts
BRIDGE_SESSION_ID=<mon-id> bash "$S/send-message.sh" <son-id> ping "test"
ls -t ~/.claude/session-bridge/sessions/<son-id>/inbox/*.json | head -1
```

If the file appears in the peer's inbox, messages are flowing.

---

## 3. C2C mailboxes — the reference channel

`c2c-os/03_handoffs/mailboxes/<identité>/inbox` and `outbox`, versioned in the
GitHub repository. This is the **canonical** channel: everything that matters
passes through it.

### Write a message

A Markdown file with a YAML header. Name:
`<horodatage>Z_<TYPE>_<message-id>_<expéditeur>_to_<destinataire>.md`

Minimum header:

```yaml
---
schema_version: '1.0'
message_id: MSG-<horodatage><suffixe>
correlation_id: CORR-<SUJET>-<date>-<n>
timestamp_utc: '2026-08-28T14:00:00Z'
sender_conversation_id: <mon-identité>
recipient_conversation_id: <son-identité>
project_id: AAAA-OS  # publication-ok: nom-interne
message_type: REQUEST | RESPONSE | STATUS | ACK | CLOSE
status: SENT
priority: LOW | NORMAL | HIGH
subject: 'une phrase qui dit ce qu il faut faire ou savoir'
responds_to_message_id: null
human_validation_required: false
---
```

Always end with the protocol constraints: canonical GitHub mailbox, no secrets,
sensitive actions subject to human validation, and **never mark your own
recommendation as a human decision**.

### Active identities

| Identity | Role |
|---|---|
| `conv-claude-architecture-helper-pc-001` | “allo” session — SSH access to both servers, credentials, production chains |
| `conv-allo-desktop-01` | “allo-desktop” session — graphical interface, local files, Windows tasks |
| `conv-claude-pc-bello-001` | Bello — supervision, Glance widgets |
| `kimi-k3` | delegated development |
| `conv-<candidature>-lead-001` | LEAD2 — drafting the current application |
| `conv-<module-media>-001` | media module — media-content production |
| `conv-codex-001`, `conv-vps-deployment-00x` | deployment |

### The rule that matters

**An unread `REQUEST` means someone is blocked.** On 28 August, an alert about
a dead-letter service remained unread for four hours. Reading the mailbox is
the first item at each handover, before any personal work.

---

## 4. `DEMANDES_JOHN.md` — John writes once, everyone reads

`c2c-os/00_manifest/DEMANDES_JOHN.md`

John adds a dated entry with a recipient (`libre` if it is intended for the
first available person). Each session replies **below it**, signs its reply,
and sees the others' responses — so sessions complement one another instead of
repeating the same work.

This solves two real problems: typing the same question into three windows and
losing a question sent to a sleeping session.

---

## 5. Connect a colleague's Claude session

For a new colleague — Sindi, Anna Zacharian, or the next person — whose Claude
session must join the team.

### Decisions required first

1. **A C2C identity** in the form `conv-<nom>-<projet>-001`. It serves as the
   mailbox and sender name.
2. **Its scope**: what it may access and, above all, what it may not. Default:
   read the repository, write to its own mailbox, nothing else.
3. **Who validates its sensitive actions.** Never the session itself.

### Steps

1. Create `c2c-os/03_handoffs/mailboxes/<identité>/inbox` and `outbox`.
2. Add the identity to `C2C_ALLOWED_SENDERS`; otherwise its messages are
   received and **silently ignored**. This happened and cost several days.
3. Send a welcome message containing its identity, a link to this guide, the
   non-negotiable rules, and **a first concrete question** proving that the
   round trip works.
4. **Wait for its response and verify it.** An untested channel is not an open
   channel.

### Rules to provide without exception

- Never include a secret in a message — no token, password or key.
- No irreversible action, or action visible to others, without explicit human
  validation: no `git push` to shared material, no deployment, no message to a
  partner, and no deletion.
- Never validate your own `human_validation_required` action.

---

## 6. Manage and monitor connections

Perform these checks at every handover — they are inexpensive and prevent a
channel from dying silently.

```bash
# Qui est en ligne sur le pont, à l'instant
bash ~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts/list-peers.sh

# Ma boîte : ai-je du non-lu ?
ls -t c2c-os/03_handoffs/mailboxes/conv-claude-architecture-helper-pc-001/inbox | head -6

# Qui a écrit dans les dernières 24 h, toutes boîtes confondues
for d in c2c-os/03_handoffs/mailboxes/*/inbox; do
  n=$(find "$d" -newermt "24 hours ago" -type f 2>/dev/null | wc -l)
  [ "$n" -gt 0 ] && echo "$n  $(basename $(dirname $d))"
done

# Qui dort depuis plus de 7 jours
for d in c2c-os/03_handoffs/mailboxes/*/; do
  [ "$(find "$d" -type f -newermt '7 days ago' 2>/dev/null | wc -l)" -eq 0 ] \
    && echo "DORMANT : $(basename $d)"
done
```

An agent silent for a week is **broken, forgotten, or finished** — and those
three states require different responses. Determine which one applies before
drawing a conclusion.

---

## 7. Traps encountered and paid for on 28 August

**The instrument lies more often than the system.** Six false conclusions in
one day, all caused by incorrect measurement:

- `ListPlugins` queries the cloud, not the local system → “I do not have the plugin”.
- `journalctl -u ssh` against a nonexistent unit name → “nothing is logged”.
- `comm` on sorted lines from a BOM+CRLF file → “1,486 lines lost”.
- `grep | head -5` → “this document does not exist”.
- host `pg_restore` instead of the container version → “version conflict”.
- `POST /api/comments/` interpreted as a write → “14,255 comments per day”.

**The rule**: first suspect a surprising result of being a measurement
artefact. Verify it from a second angle before drawing any conclusion.

**And the structural pattern**: eight times this week, a tool was built, tested
and deployed — but one component had never been connected. The bridge installed
for a month without any session registering is the latest example. **Verify the
connection, not the existence.**

---

## 8. Remaining work

- Write the onboarding tutorial intended for colleagues themselves, in English
  and in Albanian for Sindi.
- Automate channel health checks during handovers instead of performing them
  manually.
- Decide whether bridge registration should be renewed automatically, given
  that it expires.
