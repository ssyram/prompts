# HoarePrompt Audit Loop — Iterative Correctness Verification

Run a continuous correctness audit using the HoarePrompt methodology (see `/hoare-prompt` for method reference). The loop automatically isolates root causes, auto-fixes non-decisional findings, and accumulates decisional architectural trade-offs for a human gate.

## Target

$ARGUMENTS

If no target specified, audit the current working directory.

## Step 0 — Design Specification Gate (MANDATORY)

**No audit without a spec. Period.** An audit without a spec generates confident-sounding findings that may contradict intentional design decisions.

**Do NOT start Step 1 until Step 0 is complete.**

### 0a. Check for explicit design documents

Search for design docs: `docs/design/`, `DESIGN.md`, `docs/architecture.md`, `docs/detailed-design.md`, or similar. Also check `CLAUDE.md` for design-relevant sections (known limitations, design decisions, invariants).

### 0b. If design docs exist → extract contracts as ground truth

For each target function, extract:
- **Signature** (infallible vs fallible — if design says `-> Self`, code returning `Result` is a violation)
- **Listed errors** (only these, not more — additions are deviations)
- **"Caller guarantees"** / Pre-condition assignments (who is responsible for what)
- **"Intentionally NOT done"** decisions and their rationale (NOT bugs — trade-offs)
- **Availability/correctness trade-offs** ("fire-and-forget", "best-effort", "defensive only")

These are the **ground truth**. Code that matches the design is CORRECT, even if the auditor's intuition says "missing something".

### 0c. If NO design docs exist → run `/hoare-design` first

**Do NOT proceed to Step 1.** Instead:
1. Inform the user: "No design document found for [target]. Running `/hoare-design` to derive a descriptive spec."
2. Run `/hoare-design`. Apply Phase 5 categorization:
   - **Non-Decisional specs** (unanimous caller behavior): Auto-adopt immediately.
   - **Decisional conflicts** (caller contradictions): Mark for human review. Do not block. Assume weakest contract for this round.
3. Present the derived spec to the user for review before proceeding.

### 0d. Record the spec source

At the top of each round's output, state:
```
Spec source: docs/design/detailed-design.md (explicit design document)
— or —
Spec source: /hoare-design output (reverse-engineered from ALL call sites)
— or —
Spec source: user-provided verbal description
```

**Never proceed with "Spec source: NONE".** If no spec can be obtained, stop.

## Procedure

### Round N (repeat until both queues empty):

**Step 1 — Deployment context declaration + pick audit dimensions.**

**1a. Declare deployment context.** Write a deployment profile that calibrates what "realistic threat" means:

```
Deployment context:
  - Execution model: <CLI tool / long-running server / library / plugin / ...>
  - Trust boundary: <who provides input — user, LLM, network, internal module>
  - Persistence model: <ephemeral / in-memory / disk-backed / database>
  - Concurrency model: <single-threaded / async-single / multi-threaded / multi-process>
  - Threat model: <local-user-only / multi-tenant / internet-facing / ...>
```

This profile scopes ALL subsequent analysis.

**1b. Pick audit dimensions.** Pick 3–6 dimensions that matter for THIS target. Do NOT default to a fixed list — different projects need different angles.

Candidate angles to *consider* (non-exhaustive — pick, extend, or invent as appropriate):

- **Crash safety / null-safety** — null derefs, uncaught throws, shape mismatches at dynamic boundaries
- **Functional correctness** — contracts, invariants, branch exhaustiveness, loop termination (NSP-style)
- **External-API contracts** — assumptions about framework/library behavior, verified against source
- **Resource lifecycle** — timers, handles, subscriptions, file descriptors, release on all paths
- **Concurrency / persistence integrity** — races, partial-write recovery, atomicity, fsync ordering
- **Security / input validation** — injection, path traversal, SSRF, untrusted data reaching sensitive sinks
- **Error propagation & observability** — silent failures, swallowed errors, misleading return values
- **Protocol / state-machine correctness** — unreachable states, deadlocks, invariant violations across transitions
- **Performance / DoS** — unbounded loops/allocation, pathological inputs, quadratic behavior
- **Backward/forward compatibility** — schema migrations, on-disk formats, wire protocols
- **Privacy / data handling** — PII flow, retention, redaction in logs/telemetry
- **Determinism & reproducibility** — time, randomness, iteration order, floating point
- **Portability** — platform assumptions (paths, shells, line endings, encoding)
- **Idempotency** — retry safety, exactly-once vs at-least-once semantics
- **Permission / authorization** — privilege escalation, tool-exposure boundaries
- **Upgrade / rollback** — state survives restart, versioned artifacts, migration reversibility
- **Numerical / boundary correctness** — off-by-one, overflow, underflow, integer vs float
- **Concurrency beyond races** — starvation, priority inversion, deadlock cycles
- **Connection / session lifecycle** — connect/disconnect timing, disconnect side-effects, PID/session mapping lifetime assumptions, reconnection strategy
- **Testability / observability** — can we tell when it breaks in prod?

State your chosen dimensions and *why each is relevant* before proceeding.

**Step 2 — Launch one agent per dimension in parallel.** Each reads source code from scratch. No agent sees another's findings. Each writes to `docs/audit/RoundN/audit-<dimension>.md`. Agents MUST consult framework/library source as their source of truth, not docs or assumptions.

**Step 3 — Compile findings.** Read all reports. Apply:

**3a. Hoare finding vs code review nit — evidence requirements.** A Hoare finding must have:
- A specific Pre/Post/Invariant that is violated, AND
- A concrete counterexample (input, state, or execution path) that triggers the violation, AND
- **For code findings**: a concrete test case, PoC script, or step-by-step reproduction that demonstrates the bug fires. "This could theoretically happen" is not sufficient — show it happening.
- **For documentation findings** (stale content, contradictions, missing markers): triple-confirm that no other part of the document mitigates the issue (e.g., a changelog entry, a disclaimer paragraph, a cross-reference). If mitigation exists elsewhere in the document, downgrade to nit unless the mitigation is clearly insufficient for the document's primary use pattern (e.g., reference lookup vs sequential reading).

Observations lacking the above are **code review nits** — log separately in `docs/audit/RoundN/nits.md` if desired.

Exception: dead code / stale comments / unused params remain valid findings (maintainability signal-to-noise), but still require citing the exact location and explaining why the staleness causes confusion or maintenance burden.

**3b. Runtime impact trace.** For each candidate, trace whether the issue is **observable at runtime** given the deployment context:
- Does the violating input actually reach the affected code path?
- Is the path reachable from actual entry points, or only from dead callers?
- If reachable only under a non-matching threat model → downgrade to **hardening suggestion**.

**3c. Classify surviving issues:**
- VULNERABLE / DISPROVEN / REFUTED / LEAK → **candidate** for fix (must pass Step 4)
- GUARDED / PARTIAL / UNVERIFIABLE → evaluate case-by-case
- SAFE / PROVEN / CONFIRMED / NONE → no action

**Step 4 — Independent Challenge (Disprove-First).** Every candidate must be independently **challenged** by ≥2 fresh agents that did not produce it. The challenger's job is to **disprove** the finding — find reasons it is wrong, overstated, or based on misreading. Fresh eyes only — do NOT give challengers the original audit report, only the specific claim.

**Rationale**: Confirmation bias is the #1 source of false-positive audit findings. An agent asked to "confirm" a finding will look for evidence supporting it. An agent asked to "disprove" a finding will look for evidence against it. Findings that survive disproval attempts are far more reliable.

Give each challenger:
- The specific claim (file, function, description, proposed severity) and relevant source files.
- The deployment context from Step 1.
- Explicit instruction: "Your job is to find reasons this finding is WRONG. Look for evidence that contradicts it. Be adversarial."

Require one of these verdicts in `docs/audit/RoundN/challenge-<finding-id>.md`:
- **UPHELD**: challenger tried to disprove but could not. Finding stands. Challenger must state what disproval angles were attempted and why they failed.
- **WEAKENED**: finding is partially correct but overstated. Challenger must state what is correct and what is exaggerated, with evidence.
- **REJECTED — deployment context mismatch**: claim assumes a threat model that doesn't match.
- **REJECTED — not reachable**: claim assumes a state that invariant Y prevents; cite invariant.
- **REJECTED — misreading**: original auditor misread the code; explain what it actually does.
- **REJECTED — mitigated**: the issue exists in isolation but is mitigated by another mechanism the auditor missed; cite the mitigation.
- **INCONCLUSIVE**: cannot decide; state what would be needed.

Batch up to 6 challengers in parallel. Each finding needs ≥2 independent challengers.

**Verdict aggregation**:
- All challengers UPHELD → finding proceeds to Step 4.1
- All challengers REJECTED → finding discarded (log in rejected findings)
- Mixed verdicts (split) → finding proceeds to Step 4.1 with a "disputed" flag
- All challengers WEAKENED → finding proceeds with adjusted severity/scope

**Step 4.1 — Counter-Challenge (for surviving findings).** For each finding that survived Step 4 (UPHELD or disputed), launch one more fresh agent as a **counter-challenger**. This agent receives:
- The original claim
- A summary of the challenger verdicts (what disproval angles were tried)
- Instruction: "Previous challengers tried to disprove this and failed. Make one final attempt. Look for angles they missed."

If the counter-challenger also cannot disprove → finding is **confirmed**. If counter-challenger finds a valid disproval → finding is **disputed** and must be presented to the human with both sides' evidence in Step 8.

**Step 4.2 — Lead Auditor Verification.** The lead auditor (you) must personally verify the key evidence for each confirmed finding by reading the actual source code. Do not rely solely on agent reports. For each finding, state: "I verified [specific evidence] at [file:line]" or "I could not verify [claim] — agents may have misread."

For disputed findings, compile all sides' arguments and evidence into a balanced summary for the human decision gate.

**Step 4.5 — Reduction and Classification ("只诛首恶" / Isolate Root Cause).**

*This step is executed EVERY round, not just the final one.*

Apply the reduction pipeline to all confirmed findings:
1. **Deduplicate**: Merge multiple findings flagging the same logic flaw from different angles.
2. **Isolate Root Cause (只诛首恶)**: If Finding B only exists because Finding A exists, drop B and keep A. Only the root cause survives.
3. **Filter Resolved**: Drop findings already eliminated by fixes in prior rounds.

Split remaining reduced findings into two buckets:
- **Non-Decisional (Autonomous)**: Trivial fixes, local logic errors, off-by-one, null derefs, explicit undisputed spec violations — unambiguous bugs with a clear correct fix.
- **Decisional (Human-in-the-loop)**: Architectural trade-offs, API contract redefinitions, module boundary changes, conflicting specs (from Step 0), design philosophy choices.

**Step 5 — Auto-fix Non-Decisional findings.** Automatically implement fixes for all Non-Decisional findings. No human permission needed. Apply minimal patches targeting root causes from Step 4.5. For non-trivial fixes, document root cause and fix rationale (what was wrong, why this fix, expected behavior after fix).

**Step 6 — Verification sweep.** Launch an agent to verify each fix (PASS/FAIL per fix point). Write to `docs/audit/RoundN/verify.md`. Add regression tests where applicable. If any FAIL, go back to Step 5.

**Step 7 — Convergence check.**
- If ANY Non-Decisional findings remain (or Step 5 fixes generated new issues) → increment Round N, loop back to Step 1.
- If Non-Decisional == 0 → proceed to Step 8.

**Step 8 — Human Decision Gate.** Present all accumulated Decisional findings to the human:
- Summarize each architectural trade-off with pros/cons.
- Include the spec conflict context from Step 0 if applicable.
- Wait for human decisions.

**Step 9 — Apply decisions & restart.** Execute human decisions. Because these alter the system's specification/architecture, **restart the loop** (Round M, from Step 0) to re-evaluate under the new specs. Continue until BOTH Non-Decisional and Decisional queues are empty.

### Final Output

Write `docs/correctness-audit.md` containing:

1. **Executive Summary** — table of dimension → result counts (raised / confirmed / fixed)
2. **Proven Objects** — strict scope: what exactly is proven, what assumptions it depends on
3. **Cross-boundary Contracts** — table of all verified assumptions
4. **Issues Found and Fixed** — table per round: what was raised, what survived confirmation, what was done
5. **Rejected / Inconclusive Findings** — log of discarded candidates with reasons (prevents re-raising in later rounds)
6. **Decisional Findings Log** — all decisional items, human decisions made, and rationale
7. **Remaining Limitations** — anything that couldn't be fully proven (PARTIAL/UNVERIFIABLE)
8. **Assumptions Registry** — every external dependency, for re-verification if APIs change

## Rules

- **Preserve the trail**: Every round's reports, confirmations, and verifications live under `docs/audit/RoundN/`. Never delete.
- **Fresh eyes each round**: New agents must NOT read prior `docs/audit/RoundN-k/` directories. Explicitly instruct: "do not read any subdirectory of `docs/audit/`".
- **Framework = source of truth**: "The docs say X" is not evidence. `module.rs:line_N` is evidence.
- **Fix before re-prove**: Never accept a "known limitation" if the code can be changed to eliminate it.
- **Strict verdicts**: PROVEN means every path was traced. If you skipped an edge case, it's PARTIAL.
- **No annotations in source**: Proofs live in `docs/`, not as inline comments.
- **Independent challenge is mandatory**: No finding becomes a fix without surviving Step 4 challenge. Speculatives without a trigger or solid rationale are rejected and logged. Findings that no challenger attempted to disprove are treated as unvetted.
- **Rejected findings are logged, not forgotten**: Prevents re-raising the same false alarm.
- **Dead code / stale comments / unused params are valid findings**: They don't need a runtime trigger — rationale is signal-to-noise.
- **Non-Decisional first**: Always prioritize autonomous fixes. Only escalate to human when the decision fundamentally alters system design.
- **Isolate root cause**: Step 4.5 pruning is mandatory every round. Never patch symptoms.
- **Continuous spec refinement**: If a fix changes behavior boundaries, update the spec.
- **Circuit breaker**: If the same module has confirmed findings in 3+ consecutive rounds, stop patching. Do a root cause analysis of why fixes keep generating new issues in that module.
- **Disprove-first, not confirm-first**: The default stance toward any finding is skepticism. Findings must survive active disproval attempts before becoming actionable. An uncontested finding is weaker than a contested-and-surviving one.
- **Evidence over assertion**: For code findings, "I believe this could happen" is not evidence. Show a concrete test case, PoC, or step-by-step trace that demonstrates the issue fires. For doc findings, show the exact text that is wrong AND confirm no other part of the document corrects or mitigates it.
- **Disputed findings go to human**: If challengers and counter-challengers disagree, present both sides' evidence to the human. Do not resolve disputes by majority vote — present the arguments.
