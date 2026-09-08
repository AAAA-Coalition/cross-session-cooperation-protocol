---
title: "Colleague tutorial — communication channels, bots, cooperation"
date: 2026-09-05
written_on: 2026-09-04
status: "Prepared for the knowledge-transfer session of 5 September 2026"
french_version: "docs/TUTORIEL_COLLEGUES_20260905.md"
---

# Tutorial for colleagues — 5 September 2026

This document is written for someone arriving with no context at all. It does
not replace the project's reference documents; it tells you which ones to
read, in what order, and gathers what you need to know before taking part:
which communication channels exist and what each one is for, what a bot is
here and what it is not, how we cooperate without stepping on each other, and
the mistakes that have already cost us dearly — each with its date, because a
lesson without a date is just an opinion.

Everything below comes from existing documents, measured against the systems
actually in production. Nothing is promised here that has not been proven.

## 1. The communication channels, and what each one is for

There are several ways to talk to each other in this project, and the right
choice is not obvious on day one. Getting it wrong has a cost: a question
asked on the wrong channel can sit unanswered for days, or be read by people
it was never meant for. Here are the four main channels, and the rule for
choosing between them.

**The Telegram group** is the channel for collective matters. Everyone sees
everything there, humans and bots alike. Use it when a piece of information
concerns the whole group, when you want to ask a question to the room, or
when you want the bot to answer in front of witnesses. Do not use it for
anything confidential: a group almost always mixes people with different
levels of access, and the standing rule is that you speak as if addressing
the least-authorised participant present. Do not use it either for anything
that must leave a durable, findable trace — Telegram is hard to search back
through, and the bot itself only sees the last forty messages. It is read in
Telegram, and nowhere else.

**The C2C mailboxes**, under `c2c-os/03_handoffs/mailboxes/`, carry messages
from one conversation to another — typically from one Claude working session
to another, or from a colleague to a session. Each registered identity has
its own folder with an inbox. A message is a file in the git repository, so
it is versioned, dated and traceable, and it waits for its addressee even if
they are switched off or away for days. This is the canonical channel: when
you need a request to be read, tracked and accountable, this is where it
goes. Do not use it when you need an answer within the minute — latency runs
in minutes or hours, at whatever pace sessions check their inboxes. Two
things to know before your first message: a C2C message is plain text with no
attachments (to pass a file, commit it somewhere in the repository and put
its path in the message); and the sender must be on the list of authorised
senders, otherwise their messages are received and silently ignored — this
has happened, and it cost days. It is read in the repository, and the message
registry (`c2c-os/02_operational_registers/MESSAGE_REGISTRY.md`) keeps the
index.

**The session bridge** connects, in real time, two Claude instances running
on the same machine. It is the only instant channel: you ask a question, the
session on the other side answers within seconds, with its full context. Use
it when you need an answer now and you know the other session is running. Use
it for nothing else, because it leaves no trace and does not survive a
session being switched off: a question sent over the bridge to a sleeping
session is simply lost. A known trap, paid for on 28 August: bridge
registration expires, and an identifier announced at 2 p.m. may no longer
exist ten minutes later — re-check your peers just before sending. There is
no place to read the bridge back afterwards; that is the price of
immediacy.

**The `switch.md` file**, at the root of the repository, is not a message
addressed to anyone: it is what you leave behind for whoever picks up the
work. When a session stops in the middle of something, it writes there where
things stand — what is done, what is not, and above all what must not be
redone. Use it for the state of work in progress, never for a question
expecting an answer: nobody in particular is tasked with reading it; the
person who takes over the work is the one who reads it. Read it at the start
of a shift, before touching anything.

The rule of choice fits in four lines: an answer needed now while the other
session is running — the bridge; something that must leave a trace and be
read even later — a C2C mailbox; a matter that concerns the whole collective
— the group; what you leave behind when you stop — `switch.md`. These
channels do not compete with each other; each one fails exactly where the
others excel. The full guide, which also covers moving real files through
Google Drive, is `c2c-os/00_manifest/GUIDE_CANAUX_COMMUNICATION.md`.

## 2. What a bot is here, and what it is not

In this project, a bot is a door, not a brain. It carries a human intention
from a place where people already are — typically Telegram — to the place
where the work actually happens, and it brings the result back. It reads, it
drafts, it proposes. It executes nothing irreversible: it does not write to
the repository, sends nothing in anyone's name, does not publish, does not
commit to anything, and neither creates nor manages groups. Any action
visible to a third party goes through a human — in practice, through John.
This is not window dressing: the rule is written into the bot's behaviour and
duplicated in code by a bounded write mode, whose changes are dated human
decisions.

Here is what the C2C bot can actually do today. The list comes from the audit
of 4 September 2026 (`docs/AUDIT_BOT_C2C_20260904.md`), and every capability
on it has been exercised for real, not merely deployed: reading the canonical
repository — documents, folder listings, full-text search, across twelve
authorised path prefixes; searching the web, proven by retrieving the
TRANSFO-05 deadline, 23 September 2026 at 17:00 Brussels time; reading a PDF
or an office document, querying the internal knowledge base and the article
search; re-reading the last forty messages of the group, proven, forty
messages returned; sending a file as an attachment, only from a pre-declared
list, proven in both directions — a successful send, and an effective
refusal outside the list. Nothing else. If someone tells you the bot can do
something more, the right response is to check the audit before believing it.

Two behaviours of the bot surprise newcomers and are not malfunctions. First,
the bot reads the whole group but only speaks when addressed — a mention of
its name, or a reply to one of its own messages — or when an authorised
person writes. A question thrown into the void by someone outside the list
will get no answer, and that is by design. Second, a bot that answers is not
proof that the chain behind it works, and a silent bot is not necessarily
broken. The document that explains all of this, including how to verify that
a bot really works, is `docs/ROLE_OF_BOTS_AAAA_OS_20260904_EN.md`.

## 3. Cooperating — between agents, and between humans

Three rules govern cooperation here. Each one was paid for before it was
written down.

**Announce before you act.** On 28 August 2026, two sessions built two daily
routine systems in parallel, each unaware the other was doing the same thing.
The duplicated work was only discovered afterwards. Since then, the rule is
to announce what you are about to undertake — in the group if the matter is
collective, in the relevant C2C mailbox otherwise — before starting, not
after finishing. Thirty seconds of announcement cost less than a day of
duplicated work.

**Pass errors on raw, never paraphrased.** On 3 September 2026, an error that
was reworded by the person reporting it masked a failure: the rewritten text
looked like the previous error, and everyone believed they were facing the
same problem when a second, distinct barrier had just appeared. What revealed
it was pasting the exact error, word for word. When something fails on your
side, copy the error message as it is — the real text, not your summary.
Your summary contains your hypothesis, and if your hypothesis were right, you
would not have an error.

**A coordination rule must be executable by the people it is addressed to.**
A rule stated in late August — "a collective matter goes to the group" —
was, in practice, only executable by the sessions that could actually write
to that channel. For everyone else it was a wish. Before laying down a
cooperation rule, check that every addressee genuinely has the means to
follow it; otherwise you have not written a rule, you have written a
reproach in advance.

## 4. The traps that cost us, with their dates

**A search that finds nothing proves nothing, until you have checked its
scope.** On 4 September 2026, a reference sheet stated that no social media
account existed for the Coalition. That was false: the accounts had been on
record since 8 August in
`c2c-os/01_project_memory/REGISTRY_EXTERNAL_ACCOUNTS_APPS.md`. The search
that produced that sentence simply did not cover that folder. The gap was not
in the repository; it was in the place where the search had looked. Before
writing "it does not exist", always ask yourself where you looked — and
where you did not. This is the most useful trap for a newcomer to know,
because it is the one you commit in your very first week.

**Checking the corrected file proves nothing; you must check the path the
code actually travels.** On 3 September 2026, a faulty filter was corrected
in a file, and the correction was proven: the right values were in the file,
the service had been restarted. It had no effect — the code actually running
was a subclass that overrode the corrected method and redid the same check
with the old value. Two definitions of the same truth, only one of them
fixed. The bot stayed silent until the next day. When you deploy a fix, the
counter-proof goes through the real interface — redo the action that was
failing — never through re-reading the file.

**A missing fact is fixed by a readable document, not by a rule.** On
4 September 2026, in front of eighteen people, the bot expanded the acronym of
one of our organisations into the name of an entirely unrelated one. It did not
disobey: no document in its context described that organisation, so the model invented a plausible
expansion. Forbidding invention does not supply the right answer; what it
took was writing a fact sheet
(`c2c-os/00_manifest/REFERENCE_ORGANISATIONS.md`) and purging the group
memory that already contained the wrong version — because a rule does not
beat a false fact already present in the context. The lesson applies to
everyone, humans included: when someone gets it wrong for lack of knowledge,
the answer is to make the fact readable where they will look, not to add
another instruction.

These three traps share a family resemblance: in all three cases, something
looked true — an empty search, a corrected file, a written rule — and nobody
had checked the path between the appearance and the reality. That is the
central reflex of this project: verify the wiring, not the existence.

## 5. Concrete first steps

If you are arriving today, here is the reading order. First, this document.
Then `docs/ROLE_OF_BOTS_AAAA_OS_20260904_EN.md`, which explains the bots you
will meet in the group and how to tell whether they are working. Then
`c2c-os/00_manifest/REFERENCE_ORGANISATIONS.md`, the fact sheet on
organisations and acronyms — it is what will keep you from saying something
false about one of our organisations or handing out a wrong social media link. Next,
`c2c-os/PIEGES_CONNUS.md`, short and kept alive, to consult before any
non-trivial task. Finally, on the day you receive your own working
environment, the full process is in
`c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1_EN.md`.

To ask a question: if it concerns everyone, the Telegram group; if it is
addressed to a specific session or person and can wait a few hours, their C2C
mailbox; if it is urgent and you know the session is running on the same
machine, the bridge. When in doubt, the C2C mailbox is the choice that never
hurts: it leaves a trace and always ends up being read.

That leaves the most important question: how do you know whether what you
believe to be true still is? The project's canonical memory is the git
repository — not the conversations, not the bots; a figure quoted by a bot
without a source is not a fact. Every serious document in the repository
carries a date and often a status line: read them, because a page is a
snapshot of the day it was written, not a perpetual present. This tutorial
itself will be wrong one day; it was written on 4 September 2026, and
everything it states about the bot's capabilities holds for that date. When
the stakes deserve it, go back to the measured source — the audit, the
registry, the production file — rather than the document that summarises it.
And if you catch yourself about to write "it does not exist", re-read the
first trap in section 4 before pressing Enter.
