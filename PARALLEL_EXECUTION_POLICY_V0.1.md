# Parallel Execution Policy — AAAA OS

Version: 0.1
Status: VALIDATED OPERATING POLICY
Date: 2026-07-09
Thread: `THR-AAAA-ARCH-001`

## Purpose

Advance the highest-priority AAAA OS missions rapidly while maintaining bounded progress on useful secondary work whenever primary threads are waiting for external results, human gates or long-running services.

## Capacity allocation

Default portfolio allocation per execution cycle:

- **70% — P0/P1 critical path:** architecture, continuity, C2C, active project delivery, security and blocking infrastructure;
- **20% — P1/P2 background build:** Qwen knowledge, document management, telemetry, Windows Runner evolution and Telegram preparation;
- **10% — maintenance and optimization:** evidence, documentation, tests, cleanup, caching and R1000 improvements.

This is a scheduling target, not a rigid time sheet. A P0 incident may temporarily consume 100% of capacity.

## Concurrency rules

1. At most one infrastructure-changing deployment per target node at a time.
2. Documentation, corpus preparation, tests and architecture design may proceed in parallel with external deployment waits.
3. Two tasks may not write the same local file, service or registry concurrently.
4. A thread in `WAITING_EXTERNAL` or `WAITING_HUMAN` releases its active execution slot.
5. Every background task must be interruptible and must preserve a checkpoint.
6. No secondary task may unlock the secondary server, Phase 5B, public DNS, firewall changes or unrestricted shell access.

## Daily execution lanes

### Lane A — Critical delivery

- `THR-AAAA-C2C-001`;
- `THR-AAAA-ARCH-001`;
- `THR-AAAA-MEM-001`;
- `THR-<CANDIDATURE>-PROPOSAL-001` (the grant proposal currently in progress).

### Lane B — Required enabling systems

- `THR-AAAA-TG-001`;
- `THR-AAAA-DOCS-001`;
- `THR-AAAA-QWEN-001`;
- `THR-AAAA-PERF-001`.

### Lane C — Background improvements

- `THR-AAAA-WINRUN-001`;
- `THR-AAAA-OBSERVE-001`;
- `THR-AAAA-VPS-MIRROR-001` when justified by telemetry.

## Scheduling states

Each active thread must identify one of:

- `RUNNING_NOW`;
- `NEXT_READY`;
- `WAITING_EXTERNAL`;
- `WAITING_HUMAN`;
- `BACKGROUND_ACTIVE`;
- `DEFERRED`.

The command center should show at least one `RUNNING_NOW` P0/P1 thread and, when capacity permits, one `BACKGROUND_ACTIVE` thread.

## Time estimates

Record separately:

- active human-free execution time;
- expected external waiting time;
- next checkpoint;
- earliest useful result;
- definition of done.

Estimates are ranges and are updated from telemetry.

## Anti-fragmentation rule

Do not create a new thread for a minor step that belongs to an existing objective. Create a child mission or task instead. Create a new thread only when the objective, owner, lifecycle or acceptance criteria are materially distinct.

## Current allocation

### RUNNING_NOW / WAITING RESULT

- `THR-AAAA-C2C-001` — definitive Autopilot result;
- thread dashboard Word-mirror mission R3 — Windows Runner result.

### RUNNING_NOW / ARCHITECTURE

- `THR-AAAA-ARCH-001` — consolidate next architecture phase;
- `THR-AAAA-DOCS-001` — human and digital-brain document organization.

### BACKGROUND_ACTIVE

- `THR-AAAA-QWEN-001` — trusted corpus manifest and RAG preparation;
- `THR-AAAA-PERF-001` — telemetry schema/instrumentation preparation;
- `THR-AAAA-TG-001` — command parser design and read-only pilot preparation.

## Reporting

A portfolio report must show:

- critical lane status;
- background lane status;
- blocked/waiting items;
- next expected results;
- any priority displacement and its reason.
