# Supervision Lessons — binding rulings from supervised runs

Lessons extracted from real supervised runs (anchors cited). Append new rulings here; never re-derive them per run. Load this reference before supervising a live run.

## Probe by default (2026-11-24/25, self-scan round 1)

A stalled run is the watchdog's work, not the user's. When progress readings go flat, read the subject's native records first: the exact inbound message text it received, the real tool receipts, and the FSM state change. Escalate to the user only for scope/content decisions, irreversible operations, or exhausted credentials/quota. User ruling: 探查是监护模型的明确职责；运行范围与内容已给定时，其余全自动。

## A constant reading is not progress

Re-printing the same occupied/zero-batch number is monitoring theater. For every stall, name the holder (task/actor), when the lock started, and the evidence it is really held; sample on a cadence and compare readings instead of echoing them.

## Status fields are not host facts (I-06 / I-15, `planned` misread)

`launch.status=planned` does not prove "never dispatched". Declaring a worker dead or a task never-sent requires pairing the real actor's host-side terminal state (provider terminal error, real exit). If the pairing fails, the state is unknown — keep the occupation and say so. Unknown must never become a silent release.

## Never hot-patch product source to unblock a live run (I-15)

A temporary edit to `scripts/cca_runtime/recovery.py` (early return for planned-without-writer) bypassed a safety check and changed what a later accepted release actually was: one package (a85f1dbf) was released through the bypass branch. Recovery goes through existing legitimate channels — in that incident, restarting the resident service let the original implementation release four packages on its own. When a hot-patch conflicts with an existing test, the test wins and the patch is reverted.

## Keep the control-side evidence chain

Every intervention: record the exact command/API, its receipt, and the authority state (SQL revision or equivalent) before and after. Controller-side records do not substitute for subject-side native evidence; "subject transcript shows nothing" is unclosed, not proof of no intervention. File/process side effects the subject cannot see (recover commands, service restarts, model switches, archive moves) need the control-side record attached.

## Pin identity across rebuilds (2026-11-23, round 2 voided)

Re-attaching a workspace with a fresh session id is a new identity: it hits binding conflicts and produces void work. For continuation, pin root session id, workspace, real provider, and the real child FSM history path `~/.mcp_server_fsm/sessions/<root>/<child>/history.jsonl` (not `…/<child>/main/`), and resume with `opencode -s <original-id>`.

## Never compare failure totals in a polluted environment (I-15)

An active `_cca_analysis/` workspace at the repo root turns 66 test files red through cwd pollution. Two failure counts collected in the same polluted tree can hide a regression. Move the workspace aside (or use a clean tree) before any before/after comparison.

## Stage handoff is an explicit instruction

"A_DONE must not self-start the next phase" is by design, not a stall. The watchdog performs stage transitions by sending an explicit instruction and verifying what the subject actually received (native receipt), never by assuming the FSM will move itself.

## Completion is not acceptance

Done-evidence ≠ user acceptance; silence ≠ approval; watchdog reports and investigations cannot proxy the user's acceptance. Present the completion evidence and wait for the verdict.

## Recognize the TUI stop-hook loop (framework gap, not agent fault)

When the subject runs as OpenCode TUI (no HTTP server), the FSM stop-hook judge cannot call back into OpenCode: `runHook` injects `OPENCODE_SERVER_URL` only when the plugin received a `serverUrl`, which exists only in `opencode serve` mode. The judge fails, the soft-gate exit-code policy turns it into a block, and the block reason is fed back to the agent as a continuation message — a self-sustaining loop (3,113 identical injections, ~62k generated tokens in one run). Signature: repeated "FSM Stop 分析未能形成合法判定 … Connect" user messages every few seconds while the agent tries to finish. Do not blame the agent and do not count these as subject messages; either run the lane as `opencode serve` or surface the framework gap. Never modify OC/FSM source to fix it.
