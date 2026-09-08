# Thread Parallel Execution Policy — AAAA OS

Version: 0.1
Status: VALIDATED PROTOTYPE
Date: 2026-07-09
Owner: `conv-vps-deployment-001`

## Purpose

Ensure that AAAA OS advances high-priority work and secondary improvement work in parallel without allowing background tooling to displace the core architecture and active project delivery.

## Capacity allocation

Default allocation while no emergency incident is active:

- 70% of active execution capacity: P0 and P1 threads;
- 20%: P2 background improvements;
- 10%: monitoring, telemetry, documentation and preventive maintenance.

When a P0 blocker exists, allocation becomes:

- 85% P0/P1;
- 10% P2;
- 5% monitoring/documentation.

## Scheduling rules

1. Work the highest-priority unblocked thread first.
2. When a thread enters `WAITING_EXTERNAL` or `WAITING_HUMAN`, immediately advance the next eligible thread.
3. Maintain at most three simultaneous active implementation threads per agent unless a dedicated worker owns the additional work.
4. Do not start a new deployment while another incompatible deployment is active.
5. Keep documentation and thread history synchronized with implementation milestones.
6. Every active thread must expose a next action, next checkpoint and definition of done.
7. Background work may be paused instantly when a P0 incident appears.

## Current execution lanes

### Lane A — Core architecture and continuity

- `THR-AAAA-ARCH-001`
- `THR-AAAA-MEM-001`
- `THR-AAAA-C2C-001`
- `THR-AAAA-DOCS-001`

### Lane B — Active project delivery

- `THR-<CANDIDATURE>-PROPOSAL-001` (the grant proposal currently in progress)
- future project-specific Lead/Watch threads

### Lane C — Background capability improvement

- `THR-AAAA-TG-001`
- `THR-AAAA-QWEN-001`
- `THR-AAAA-PERF-001`
- `THR-AAAA-WINRUN-001`
- `THR-AAAA-OBSERVE-001`

## Completion and closure

A thread is moved to the closed section only when it reaches one of:

- `PASS_FINAL`;
- `FAIL_FINAL`;
- `CANCELLED`;
- `SUPERSEDED`;
- `DEFERRED_CLOSED`.

Closed threads remain visible for audit and retain their Word history document. Any substantial new work after closure receives a new thread ID or an explicitly versioned successor.
