# Send Your First C2C Message — The Roadmap in Eight Actions

> **Who this sheet is for.** You, the colleague who has just joined.
> Not the person onboarding you.
>
> **What it is not.** This is not an explanation of how the system works —
> `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md` already does that well,
> and you can read it later. This is the sequence of actions at the end of
> which **someone will have received a message written by you**. Allow twenty
> minutes.
>
> **There is only one success criterion**: at the end, another person replies
> to you. Not “the file was created”, not “the command displayed a path”.
> Someone replies to you.
>
> Written on 05/09/2026 for the onboarding session. It complements step 4 of
> `c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md`, which says
> **what** to read; this sheet says **what to type**.

---

## Before You Start — The Two Things Nobody Can Do for You

1. **Write access to the GitHub repository.** Request it, then verify that it
   works before continuing: `git clone`, followed by `git push` of a trivial
   change. If the `push` fails, stop there — everything else depends on it.
2. **Git installed and a terminal that understands `bash`.** On Windows, Git
   Bash is suitable; PowerShell will not run the script in step 6.

---

## 1. Clone the Repository

```
git clone <url-du-depot>
cd <dossier-du-depot>
```

Write down the absolute path to this folder. You will need it at every step,
and it is the most common source of errors.

## 2. Choose Your Conversation Identifier

The convention is `conv-<qui-vous-etes>-<numero>`, in lowercase, without
accents or spaces. For example: `conv-architecture-tirana-001`.

**This identifier represents you throughout the system.** It will appear in
every message, it names your mailbox, and changing it later would break replies.
Take an extra thirty seconds to choose it carefully.

One important precaution: **an identifier is data that circulates.** Do not
include your surname, employer or a number if you do not want them to appear in
messages read by other agents.

## 3. Create Your Mailbox

Three directories and two files. From the repository root:

```
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/inbox
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/archive
```

Then create a `CAPABILITIES.yaml` file in
`c2c-os/03_handoffs/mailboxes/<votre-identifiant>/`, using the following
template. Copy an existing mailbox and adapt it rather than starting from
scratch:

```yaml
schema_version: '1.0'
conversation_id: <votre-identifiant>
project_id: AAAA-OS  # publication-ok: nom-interne
title: <votre role en cinq mots>
role: <un mot cle, ex. architecture_contributor>
status: NEW_UNVERIFIED
capabilities:
  read_mailbox: true
  send_c2c_canonical_mailbox: true
  read_message_registry: true
  github_arbitrary_write: false
```

`status: NEW_UNVERIFIED` is intentional: **your mailbox is not verified until
someone has received a message from you.** Step 8 verifies it, not its
creation.

## 4. Register Yourself

Open `c2c-os/02_operational_registers/conversation_registry.yaml` and add your
entry to the `conversations:` list by copying the format of a nearby entry:

```yaml
  - conversation_id: <votre-identifiant>
    title: <le meme titre qu'au-dessus>
    project_id: AAAA-OS  # publication-ok: nom-interne
    role: <le meme role>
    status: NEW_UNVERIFIED
```

**This step is not decorative.** Tools that perform verification reject an
identity missing from the registry as a recipient: the message is sent but
never arrives. This happened on 03/09 with six group mailboxes that had
existed for five days: reading worked, writing was rejected, and nobody knew.

## 5. Declare Where You Are

Set two variables in the terminal in which you will work:

```
export REPO=/chemin/absolu/vers/votre/depot
export MOI=<votre-identifiant>
```

**`REPO` is not optional.** Its default value in the script refers to someone
else's machine. Without this export, the script will say that the repository
cannot be found. That is not an identifier problem, contrary to what an
earlier version of the tutorial suggested.

These two `export` commands remain in effect only for the lifetime of the
terminal window. If you open another one, run them again.

## 6. Write the Body of Your Message

Create an ordinary Markdown file, **without a YAML header** — the script adds
it. Three or four lines are enough:

```
cat > /tmp/presentation.md <<'FIN'
Hello, I am joining the architecture group.

What I can help with: <two specific lines>.
What I need to get started: <one line>.
FIN
```

Write something true. This message will be read by people and agents, and it
will remain in the repository history.

## 7. Deposit the Message

```
ops/deposer-message-c2c.sh <destinataire> STATUS PRESENTATION "Presentation" /tmp/presentation.md
```

The script displays the path of the written file and the difference between
the declared timestamp and the actual time. It is designed to **fail loudly**:
wrong number of arguments, missing body, nonexistent recipient mailbox or
inconsistent timestamp — every failure has its own message.

**However, success here does not mean that the message was sent.** It only
means that a file exists on your disk.

## 8. Publish — This Is the Step That Sends

```
git add c2c-os/
git commit -m "C2C: presentation de <votre-identifiant>"
git pull --rebase origin main
git push origin main
```

**The canonical mailbox is on GitHub.** Until the `push` has taken place, the
recipient sees nothing, and neither do the agents that read the repository —
their context loader reads GitHub, not your disk.

This step is often forgotten because step 7 looks like sending. A message left
on its author's disk alongside a success message is the most costly failure in
this protocol: neither the sender, who believes it was sent, nor the
recipient, who is waiting for nothing, can see the problem.

---

## How to Know That It Worked

**The proof is not on your side.** Check in this order:

1. On GitHub, your file appears in
   `c2c-os/03_handoffs/mailboxes/<destinataire>/inbox/` on the `main`
   branch. If it is not there, your `push` did not succeed — read its output.
2. **Someone replies to you.** This is the only criterion that matters. Ask
   the person you contacted to confirm, and do not regard this step as
   complete before they do.

If nothing arrives after a reasonable period, the problem is almost always
one of these three things, in descending order of frequency: the `push` did
not happen; the recipient identifier is misspelled; or your identity is not in
the registry (step 4).

---

## What You Are Not Doing Yet — and That Is Normal

You now know how to write to a mailbox. You do not yet know how to reply to a
`REQUEST`, maintain the message registry or make an agent speak on your
behalf — and none of that is urgent.

**The order is deliberate: the channel first, the agent second.** Someone who
leaves with a conversational agent but without knowing how to write to a
mailbox leaves with something impressive that produces nothing. Read the next
steps in `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md` when you need
them — not before.
