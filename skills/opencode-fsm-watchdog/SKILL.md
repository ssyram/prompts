---
name: opencode-fsm-watchdog
description: Set up, run, stage-aware monitor, diagnose, and safely recover persistent OpenCode sessions driven by mcp-server-fsm while preserving native OpenCode permissions. Use for long-running or concurrent FSM work, same-session continuation, or controlled reruns after infrastructure failure.
---

# OpenCode FSM Watchdog

## Purpose

Operate the actual OpenCode↔FSM path, observe it without changing run artifacts, and repair infrastructure without corrupting evidence. This skill covers infrastructure and operation; it does not judge project results or replace FSM gates.

## Principles

### Establish the actual run

Resolve and verify what will really execute: OpenCode identity/configuration, plugin/client/hook, FSM daemon and registered definition graph, workspace/input/toolchain, provider/model/credential, and mutable resources. Names and version strings are not substitutes for exercising the effective path.

FSM names are identities. Concurrent definitions require unique names across the complete transitive graph, including dispatched children. Otherwise isolate registries/daemons. Registry overwrite after actor entry is only a warned fallback.

### Observe the current stage without changing it

Derive monitoring from the current state's responsibilities, transitions, tools, expected outputs, legitimate long operations, known pitfalls, and the user's explicit concerns. Monitoring is read-only by default: no prompt, FSM call, workspace edit, or worker management without authorization.

### Preserve native OpenCode permissions

Use OpenCode's native global/project/session permission resolution. Run-data isolation must not discard, narrow, or replace the existing global permission policy. Do not introduce a permission broker, manually answer requests, or ask the user to re-approve operations already allowed. If an allow-all environment unexpectedly asks, diagnose and repair configuration inheritance instead of creating an approval workflow.

Permission isolation or custom approval handling is allowed only when the user explicitly requests exclusion of global authorization. Record the exact exclusion and use native permission mechanisms; it is not a default consequence of unattended runs, read-only monitoring, or XDG isolation. FSM gates remain enforced; native OpenCode permission inheritance does not bypass them.

### Preserve run facts during repair

Keep failed runs as evidence and repair outside them. Continue the same root only when identities/history remain intact, no action can be lost or repeated, and the user permits continuation. Otherwise start fresh.

## Phase Workflow

Load the reference for the phase being performed; do not load every reference by default.

1. **Preparation** — read [`references/preparation.md`](references/preparation.md). Establish identities, compatibility, gate environment, mutable-resource ownership, and real-path smoke evidence. Do not launch expensive work while preparation is unresolved.
2. **Launch** — read [`references/launch.md`](references/launch.md). Start persistent server/client tmux processes, establish workspace readiness, bind the root session, and preserve logs.
3. **Watch** — read [`references/watch.md`](references/watch.md). Build stage-sensitive, read-only monitoring from the live FSM and user request; report through recurring wakeups.
4. **Recovery** — read [`references/recovery.md`](references/recovery.md) when infrastructure is suspect, a run stalls, or continuation/rerun is being considered.
5. **Quota governance** — read [`references/quota-governance.md`](references/quota-governance.md) when the run consumes a metered provider budget with daily limits, when a budget stop is approaching, or when resuming runs stopped for quota.

The phases are operational contexts, not a mandatory one-way state machine. A watch finding may return work to preparation; recovery may resume watch or require a fresh launch.

## Core Operational Rules

- Treat the server-owned transcript and FSM committed history as authoritative; attached-client stdout and Git gate commits are supporting evidence only.
- Use the same root session for an explicitly authorized continuation.
- Preserve failed turns and workspaces; never rewrite evidence to make a run appear clean.
- Do not stop shared infrastructure on behalf of one run.
- Do not claim child/subagent observation without captured child evidence.
- Keep credentials out of argv, logs, manifests, and reports.
- Preserve native OpenCode permissions by default; only an explicit user request to exclude global authorization permits a different permission policy.
- Load [`references/supervision-lessons.md`](references/supervision-lessons.md) before supervising a live run: probe by default, status fields are not host facts, never hot-patch product source to unblock, pin identity across rebuilds, recognize the TUI stop-hook loop. Its rulings are binding; append new lessons there instead of re-deriving them per run.

## Approved concurrent-monitoring addition

When a human asks for concurrent batch progress, report both current direct-child activity and manifest-backed coverage. Count only manifest entries in the requested target class and their referenced artifacts; distinguish produced artifacts, required headings, and terminal acceptance. Exclude the root and unrelated sessions from worker counts, and leave unreadable sources unknown rather than treating them as zero or healthy.

Keep phase wall time, run wall time, worker/batch duration, and explicitly evidenced permission waiting separate. A tool/call lifecycle timestamp and a later observer timestamp describe different events. With insufficient like-for-like accepted samples, do not invent a universal timing threshold: activity beyond the approved observation cadence triggers a focused cause query only, never an automatic recovery or termination. Do not create another polling loop merely to reduce observation latency.

## Human-approved Evolution

Executors are encouraged to propose concrete additions when a run reveals a reusable infrastructure fact, stage-specific check, failure signature, or safe recovery rule. They must not modify this skill or its references merely because a new event occurred.

A proposed addition must state:

```text
phase; observed situation; primary evidence; generalized lesson
scope/limits; proposed wording or check; security/project-name sanitization
```

Only write it into the skill after a human explicitly approves the addition or the exact class of additions. Generalize project-specific names, omit credentials, and keep examples non-exhaustive. An approved entry may be corrected or removed when later evidence invalidates it; existing text does not gain authority merely by being present.
