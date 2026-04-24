# HoarePrompt Audit Loop — Iterative Correctness Verification

Run a multi-round correctness audit using the HoarePrompt methodology (see `/hoare-prompt` for method reference). Iterates: audit → fix → re-audit until convergence.

## Target

$ARGUMENTS

If no target specified, audit the current working directory.

## Step 0 — Design Specification Gate (MANDATORY, before any audit work)

**No audit without a spec. Period.** The audit MUST have a design specification to check code against. Without one, the auditor will unconsciously invent specs from intuition — producing false positives that look like rigorous Hoare derivations but are actually "I think the code should do X".

**Do NOT start Step 1 until Step 0 is complete.** An audit without a spec is worse than no audit — it generates confident-sounding findings that may contradict intentional design decisions.

### 0a. Check for explicit design documents

Search for design docs in the project: `docs/design/`, `DESIGN.md`, `docs/architecture.md`, `docs/detailed-design.md`, or similar. Also check `CLAUDE.md` for design-relevant sections (known limitations, design decisions, invariants).

### 0b. If design docs exist → extract contracts as ground truth

For each target function, extract from the design docs:
- **Signature** (infallible vs fallible — if design says `-> Self`, code returning `Result` is a violation)
- **Listed errors** (only these, not more — additions are deviations)
- **"Caller guarantees"** / Pre-condition assignments (these define WHO is responsible for WHAT)
- **"Intentionally NOT done"** decisions and their rationale (these are NOT bugs — they are trade-offs)
- **Availability/correctness trade-offs** ("fire-and-forget", "允许丢失", "best-effort", "defensive only")

These extracted contracts are the **ground truth**. A code behavior that matches the design is CORRECT, even if the auditor's intuition says it's "missing something".

### 0c. If NO design docs exist → run `/hoare-design` first

**Do NOT proceed to Step 1.** Instead:

1. Inform the user: "No explicit design document found for [target]. Running `/hoare-design` to derive a descriptive spec from the current code by examining all call sites."
2. Run `/hoare-design` on the target. This systematically examines ALL usages of each function to derive Pre/Post from actual caller behavior.
3. Present the derived spec to the user for review before proceeding.
4. Only after user confirms (or provides corrections) → proceed to Step 1 using the derived spec as ground truth.

### 0d. Record the spec source

At the top of each audit round's output, state:
```
Spec source: docs/design/detailed-design.md (explicit design document)
— or —
Spec source: /hoare-design output (reverse-engineered from ALL call sites)
— or —
Spec source: user-provided verbal description
```

**Never proceed with "Spec source: NONE".** If no spec can be obtained, inform the user and stop.

## Procedure

### Round N (repeat until clean):

**Step 1 — Deployment context declaration + pick audit dimensions.**

**1a. Declare the deployment context.** Before choosing dimensions, write a brief deployment profile that calibrates what "realistic threat" means for this target:

```
Deployment context:
  - Execution model: <CLI tool / long-running server / library / plugin / ...>
  - Trust boundary: <who provides input — user, LLM, network, internal module>
  - Persistence model: <ephemeral / in-memory / disk-backed / database>
  - Concurrency model: <single-threaded / async-single / multi-threaded / multi-process>
  - Threat model: <local-user-only / multi-tenant / internet-facing / ...>
```

This profile scopes ALL subsequent analysis. A finding about "untrusted network input" is irrelevant if the tool is a local CLI invoked by its owner. A finding about "concurrent mutation" is irrelevant if the runtime is single-threaded with no shared state.

**1b. Pick audit dimensions.** Think about what could realistically go wrong in THIS codebase given the deployment context above. Pick 3–6 dimensions that actually matter for this target. Do **NOT** default to any fixed list — different projects need different angles. A web service, a CLI tool, a compiler pass, an ML pipeline, and a plugin all have distinct failure modes.

Candidate angles to *consider* (non-exhaustive, non-mandatory — pick, extend, or invent as appropriate):

- **Crash safety / null-safety** — null derefs, uncaught throws, shape mismatches at dynamic boundaries
- **Functional correctness** — contracts, invariants, branch exhaustiveness, loop termination (NSP-style forward tracing)
- **External-API contracts** — assumptions about framework/library behavior, verified against source
- **Resource lifecycle** — timers, handles, subscriptions, file descriptors, module-level state, release on all paths
- **Concurrency / persistence integrity** — races on shared files or in-memory state, partial-write recovery, atomicity, fsync ordering
- **Security / input validation** — injection (shell, SQL, env, prompt), path traversal, SSRF, untrusted data reaching sensitive sinks
- **Error propagation & observability** — silent failures, swallowed errors, misleading return values, logging gaps
- **Protocol / state-machine correctness** — unreachable states, deadlocks, invariant violations across transitions
- **Performance / DoS** — unbounded loops/allocation, pathological inputs, quadratic behavior, regex catastrophic backtracking
- **Backward/forward compatibility** — schema migrations, on-disk formats, wire protocols, config evolution
- **Privacy / data handling** — PII flow, retention, redaction in logs and telemetry
- **Determinism & reproducibility** — time, randomness, iteration order, floating point
- **Portability** — platform assumptions (paths, shells, line endings, encoding)
- **Idempotency** — retry safety, exactly-once vs at-least-once semantics
- **Permission / authorization** — who can call what, privilege escalation, tool-exposure boundaries
- **Upgrade / rollback** — state survives restart, versioned artifacts, migration reversibility
- **Numerical / boundary correctness** — off-by-one, overflow, underflow, integer vs float domains
- **Concurrency beyond races** — starvation, priority inversion, deadlock cycles
- **Testability / observability of the fix surface** — can we tell when it breaks in prod?

State your chosen dimensions and *why each is relevant to this target* before proceeding. It's fine to pick only 3 if the project is small; it's fine to pick 6 for a complex service; it's fine to invent a dimension the list doesn't name.

**Step 2 — Launch one agent per chosen dimension in parallel.** Each reads source code from scratch. No agent sees another's findings. Each writes to a SEPARATE file under `docs/audit/RoundN/` (no write conflicts), named `audit-<dimension>.md`. Create the `RoundN` directory before launching. Agents MUST consult framework/library source as their source of truth, not docs or assumptions.

**Step 3 — Compile findings.** Read all reports. Apply the following filters:

**3a. Hoare finding vs code review nit.** A Hoare finding must have:
- A specific Pre/Post/Invariant that is violated, AND
- A concrete counterexample (input, state, or execution path) that triggers the violation

Observations that lack both of these are **code review nits**, not Hoare findings. Examples of nits: "this variable name is misleading", "this function is longer than it needs to be", "this could use a more idiomatic pattern", "this error message is unhelpful". Nits may be useful feedback but they are NOT correctness findings — do not route them through Step 4 confirmation. Log them separately in `docs/audit/RoundN/nits.md` if desired.

Exception: dead code / stale comments / unused params remain valid findings (per existing rules) — the rationale is maintainability signal-to-noise, not Pre/Post violation.

**3b. Runtime impact trace.** For each candidate finding, trace whether the issue is **observable at runtime** given the deployment context declared in Step 1:
- Does the violating input/state actually reach the affected code path in the declared deployment model?
- Is the code path reachable from the system's actual entry points, or only from test harnesses / dead callers?
- If the path is reachable only under a threat model that doesn't match the deployment context (e.g., "attacker controls internal module-to-module wire format"), downgrade to a **hardening suggestion**, not a finding.

**3c. Classify surviving issues:**
- VULNERABLE / DISPROVEN / REFUTED / LEAK → **candidate** for fix (must pass Step 4 first)
- GUARDED / PARTIAL / UNVERIFIABLE → evaluate case-by-case (also route through Step 4 if treated as a finding)
- SAFE / PROVEN / CONFIRMED / NONE → no action

**Step 4 — Independent confirmation of each candidate finding.** Before any code change, every flagged issue must be **independently confirmed** by a fresh sub-agent that did not produce it. The goal is to filter out false alarms, speculative concerns, and "theoretical but unreachable" claims.

For each candidate finding, launch a confirmation agent with a narrowly scoped prompt:

- Give the agent the specific claim (file:line, description, proposed severity) and the relevant source files.
- **Give the agent the deployment context from Step 1** — so it can assess reachability within the actual deployment model.
- Do NOT give the agent the original audit report or the original agent's reasoning — fresh eyes only.
- Require the agent to write its verdict to `docs/audit/RoundN/confirm-<finding-id>.md` with one of:
  - **CONFIRMED — triggering**: produce a concrete input/scenario/code path that demonstrates the bug actually fires at runtime **within the declared deployment context** (or a precise dead-code/redundancy claim: "this function has no callers; grep shows X").
  - **CONFIRMED — solid rationale**: the issue is real but not directly triggerable (e.g., defense-in-depth gap, API contract drift, stale comment). State the rationale in terms the maintainer can evaluate.
  - **REJECTED — deployment context mismatch**: the claim assumes a threat model or execution environment that doesn't match the declared deployment context. State the mismatch.
  - **REJECTED — not reachable**: the claim assumes a state that invariant Y prevents; cite the invariant and where it is enforced.
  - **REJECTED — misreading**: the original reviewer misread the code; explain what the code actually does.
  - **REJECTED — lesson precondition unmet**: the finding cites a lesson/spec whose preconditions don't hold in this context. State which precondition fails and why.
  - **INCONCLUSIVE**: cannot decide without more context; state what would be needed.

Batch confirmation agents in parallel (one per finding, or grouped by file). Run batches of up to 6 in parallel to keep wall-clock manageable.

**Discard findings that are REJECTED or INCONCLUSIVE.** Only CONFIRMED findings proceed to Step 5. A genuine bug survives independent confirmation; a speculative one does not. Completely-redundant code (dead functions, unused params, duplicated helpers) IS a valid confirmation reason — don't reject it for lack of a "runtime trigger".

**Step 5 — Fix all confirmed issues.** Apply code changes. Type-check. Do NOT re-annotate code with proofs at this stage.

**Step 6 — Quick verification sweep.** Launch a single agent to verify each specific fix (PASS/FAIL per fix point). The agent writes its report to `docs/audit/RoundN/verify.md`. If any FAIL, go back to Step 5.

**Step 7 — Check convergence.** If the round found zero issues that survived Step 4 confirmation → done. Otherwise → increment N, **do not delete prior `RoundN` directories** (they are the trail of how convergence was reached), go to Step 1 with fresh agents that are instructed not to read prior `RoundN/` directories.

### Final Output

Write `docs/correctness-audit.md` containing:

1. **Executive Summary** — table of dimension → result counts (raised / confirmed / fixed)
2. **Proven Objects** — strict scope: what exactly is proven, what assumptions it depends on
3. **Cross-boundary Contracts** — table of all verified assumptions
4. **Issues Found and Fixed** — table per round: what was raised, what survived confirmation, what was done
5. **Rejected / Inconclusive Findings** — log of candidate findings discarded at Step 4, with the reason (reachability blocked by invariant Y / misreading / insufficient evidence). This prevents the same false alarm from being re-raised in later rounds.
6. **Remaining Limitations** — anything that couldn't be fully proven (PARTIAL/UNVERIFIABLE)
7. **Assumptions Registry** — every external dependency, for re-verification if APIs change

## Rules

- **Preserve the trail**: Every round's per-dimension reports, confirmation verdicts, and verification sweep live under `docs/audit/RoundN/`. These are never deleted — they are the record of how convergence was reached, and let a future reader reconstruct the reasoning even after code has moved.
- **Fresh eyes each round**: New agents must NOT read prior `docs/audit/RoundN-k/` directories. Instruct them explicitly: "do not read any subdirectory of `docs/audit/`". The preservation is for human readers and later meta-analysis, not for in-loop agent context.
- **Final summary is the rolled-up view**: `docs/correctness-audit.md` summarizes across all rounds. Per-round files are the raw material; the summary is the distilled narrative.
- **Framework = source of truth**: "The docs say X" is not evidence. `runner.ts:line_N` is evidence.
- **Fix before re-prove**: Never accept a "known limitation" if the code can be changed to eliminate it.
- **Strict verdicts**: PROVEN means every path was traced. If you skipped an edge case, it's PARTIAL.
- **No annotations in source**: The proofs live in `docs/`, not as inline comments. Source code should have only functional comments.
- **Independent confirmation is mandatory**: No finding becomes a fix without Step 4 confirmation by an agent that did not produce the finding. The confirmer must either (a) exhibit a triggering scenario, or (b) provide a solid rationale (including "completely redundant code"). Speculative concerns without either are rejected and logged in the rejected-findings section.
- **Rejected findings are logged, not forgotten**: Adding the rejection reason to `docs/correctness-audit.md` prevents later rounds from re-raising the same false alarm.
- **Dead code / stale comments / unused params are valid findings**: They don't need a "runtime trigger" to be confirmed — the rationale is that they mislead readers and erode the audit's signal-to-noise.
