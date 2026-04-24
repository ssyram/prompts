# HoarePrompt Code Audit — Methodology Reference

Reference: Bouras, Dai, Wang, Xiong, Mechtaev, "HoarePrompt: Structural Reasoning About Program Correctness in Natural Language" (arXiv 2503.19599, 2025)

## Design Document Primacy (CRITICAL — read before anything else)

**The correctness of a Hoare derivation depends entirely on the correctness of its Requires/Ensures/Invariants.** These must come from a **design specification**, not from the auditor's intuition about what the code "should" do.

### Rule 1: Spec comes from design docs, not from intuition

When writing Requires/Ensures/Invariants for a function:
- ✅ Extract from the project's design documents (architecture.md, detailed-design.md, etc.)
- ✅ If no design doc exists, use `/hoare-design` to reverse-engineer a descriptive spec from the code first
- ❌ Do NOT write specs based on "this function should obviously check X" or "good practice requires Y"

A function that doesn't check X is not "missing a check" — it may be **delegating that responsibility to its caller by design**. The design doc (or `/hoare-design` output) tells you which.

### Rule 2: "Caller guarantees" is a valid Pre-condition source

When you see a function that trusts its input without validation:
- Check the **call graph**: does the caller validate before calling?
- If yes → the Pre-condition is **transitively established** → not a bug
- If no one validates → **that** is the finding (the gap in the responsibility chain), not the callee's missing check

Example of the mistake to avoid:
```
# WRONG: "commit_transition doesn't validate self_check_token → bug"
# RIGHT: "prepare_transition validates and consumes token before commit is called → 
#          commit's Pre 'token already validated' is established by the caller chain"
```

### Rule 3: Specs can be challenged, but challenges must be validated against ALL usages

You may suspect a design spec is incomplete or wrong. That's legitimate — design docs can have gaps. But:

- ❌ Do NOT add a "stronger" Ensures/Invariant based on intuition ("obviously this should also check X")
- ✅ DO formulate the proposed addition as a **hypothesis**, then validate it by examining **every call site** (using `/hoare-design` Phase 2 methodology)

Validation process for a proposed spec addition:
1. State the addition: "I propose adding Pre-condition P to function F"
2. Examine ALL callers of F: does every caller already establish P?
   - If yes → P is a valid implicit contract, already enforced by callers. Adding it to F's spec is redundant but harmless. Mark as "implicit, already enforced".
   - If some callers DON'T establish P → those callers are potential bugs. Report them. But the bug is in the CALLER, not in F.
   - If NO callers establish P and F works fine anyway → P is probably not a real requirement. Drop it.
3. Document the evidence: "Proposed Pre P validated against N call sites: [list]"

**Every proposed spec addition costs a full call-site sweep.** This is intentionally expensive — it prevents the auditor from casually injecting intuition-based specs.

### Rule 4 (renumbered): "Stronger spec ≠ better spec"

A design may intentionally choose a WEAKER guarantee for good reasons:
- Fire-and-forget audit logs (availability > audit completeness)
- No token check at commit (one-time token consumed at prepare forces fresh self-check on retry)
- Pure-memory lookup for hooks (simplicity, no side effects)

If your derived spec is stronger than the design's spec, the discrepancy is a **design trade-off**, not a bug. You may recommend strengthening (as a suggestion), but do NOT treat it as a confirmed finding.

### Rule 5: Requires IS the contract — don't duplicate it inside the function

If a function's Requires (from the design doc or from `/hoare-design`) says "caller guarantees X", then the function does NOT need to check X internally. A function that trusts its Requires is **correct by contract**, not "missing a check".

**The restore() lesson**: `restore()` was specified as infallible with Pre "caller guarantees definitions contain all stack entries". The auditor saw that restore didn't internally validate this and added validation + error returns. But this:
1. Changed the function's signature (infallible → fallible), breaking the contract
2. Shifted responsibility from caller to callee, contradicting the design's responsibility assignment
3. Created a cascade of downstream changes (audit logging, migration tracking, etc.) that were all unnecessary because the Pre was already established by the caller

**The principle**: In Hoare logic, `{P} S {Q}` means "if P holds before S, then Q holds after S". The proof of correctness ASSUMES P. If P says "caller guarantees X", the proof does not need to RE-VERIFY X inside S. Pulling the caller's guarantee into the callee is not "defense in depth" — it is **rewriting the contract**.

When you see a function that doesn't validate something:
- ❌ "Function F doesn't check X → add check to F" (changes contract)
- ✅ "Function F's Requires says caller checks X → verify ALL callers actually check X" (validates existing contract)

If a caller DOESN'T establish the Pre → that's a bug in the **caller**, not in the callee. The fix goes in the caller, not the callee.

### Rule 5 (renumbered): When no design doc exists, ask before proceeding

If the target functions lack explicit design documentation:
1. **Ask the user**: "No design document found for [target]. Should I run `/hoare-design` to derive a descriptive spec from the current code first?"
2. If yes → run `/hoare-design`, use its output as the spec source
3. If no → the user may provide context verbally, or you proceed with extreme caution, marking every spec as "INFERRED — not design-verified"

## What is HoarePrompt?

HoarePrompt adapts classical Hoare logic to LLM-driven code analysis. Instead of formal proofs, it uses **Natural Strongest Postconditions (NSP)** — natural-language descriptions of reachable program states at each code point. The LLM acts as the **state transformer**: given a precondition and a code block, it generates the most specific true description of the state after execution.

The core insight: LLMs are unreliable at holistic "is this code correct?" judgments, but much better at **local, structural reasoning** — describing the state transition of each statement, then chaining those descriptions compositionally.

## The Method (5 Steps)

### Step 1: Decomposition (CFG Traversal)

Break the program into its Control Flow Graph components: sequences, branches, and loops. Process each component independently, propagating state annotations forward through the graph.

### Step 2: State Annotation (Natural Strongest Postcondition)

For each code block, generate an NSP — the most specific natural-language description of what IS true about program state AFTER execution. Work forward from the function entry:

```
# { STATE: tasks is a non-empty array, nextId > max(task.id) }
tasks.push({ id: nextId++, ... });
# { STATE: tasks has one more element, its id === old(nextId), nextId === old(nextId)+1 }
```

Key principle: describe what IS true (postcondition), not what SHOULD be true. The "strongest" part means: the most specific statement that the code guarantees — not weaker than necessary.

### Step 3: Branch Analysis (Case Splitting)

At every if/switch/ternary, analyze EACH branch independently. Track each branch's postcondition separately:

```
if (task.status === "done") return "[done]";
# { STATE(here): task.status !== "done" }
if (task.status === "expired") return "[expired]";
# { STATE(here): task.status ∉ {"done", "expired"} }
# Verify exhaustiveness: are all possible values covered?
```

### Step 4: Loop Reasoning (Few-Shot Summarization + k-Induction)

The paper's key contribution: instead of asking the LLM to directly summarize a loop, use **few-shot-driven loop summarization**:

1. **Unroll** k iterations (typically k=3), showing the concrete state after each
2. The LLM identifies the **pattern** — how variables change per iteration
3. The LLM **generalizes** to a natural-language loop summary valid for any iteration count

```
while (i < len) {
  # SUMMARY: each iteration processes text[i], advances i by ≥ 1
  # TERMINATION: i strictly increases toward len
}
# POST: i >= len, entire text processed
```

For extra rigor, verify via classical k-induction:
- **Initialization**: summary holds before the first iteration
- **Maintenance**: if it holds at iteration k, it holds at k+1
- **Termination**: the progress measure strictly converges toward the bound

A cheaper fallback is pure unrolling without generalization — useful when k iterations cover the real input space. The tradeoff is cost vs generality.

### Step 5: Correctness Assessment (NSP ⇒ Specification)

Chain the local NSP annotations together, then check: **does the final accumulated NSP entail the natural-language specification?** The LLM acts as verifier, judging whether the postcondition satisfies the requirement.

```
function executeAdd(text, tasks, nextId):
  PRE:  text is string|undefined; nextId > max(task.id)
  POST: if !text → error, no mutation; else → tasks has one more element, id unique, nextId incremented
  VERDICT: POST entails spec "add a task with auto-incrementing id" — CORRECT
```

This entailment check is the paper's actual endpoint — not just "describe what happens" but "does what happens satisfy what was asked for?"

## Extended Method: Cross-boundary Verification

The original paper focuses on self-contained programs. For real-world code that depends on frameworks/libraries, add:

### Step 6: Assumption Extraction and Verification

For every call to an external API, explicitly state what you ASSUME about its behavior, then VERIFY by reading the actual source:

```
# ASSUME: pi.appendEntry(type, data) creates a CustomEntry that:
#   - Survives compaction (never deleted)
#   - Is never sent to the LLM
#   - Is retrievable via ctx.sessionManager.getEntries()
# VERIFIED: session-manager.ts:88-95 — "Does NOT participate in LLM context"
```

**Never assume framework behavior from documentation alone — read the source code.**

## What to Annotate (Checklist)

For each function, verify:

| Aspect | Question |
|--------|----------|
| **Null safety** | Can any variable be null/undefined here? Is it checked? |
| **Type safety** | Can the value be a different type than expected? (e.g., details modified by plugin) |
| **Exhaustiveness** | Are all cases of a union/enum handled? |
| **Termination** | Do all loops have a progress measure? |
| **Resource cleanup** | Is every allocation paired with a release on ALL paths? |
| **Exception safety** | If line N throws, what state is left? Are invariants preserved? |
| **Concurrency** | Can another handler observe intermediate state? (In JS: only across await points) |

## Multi-Angle Decomposition

For non-trivial codebases, split into independent audit angles:

1. **Crash Safety** — Can it throw/crash? Every function, every path.
2. **Functional Correctness** — Does it do what it claims? Prove each invariant.
3. **Cross-boundary Contracts** — Are API assumptions correct? Verify against source.
4. **Resource Safety** — Are resources (timers, sessions, handles) properly managed?
5. **Spec vs Implementation** — Read the original spec line by line; tick each requirement. Self-consistent code can still miss a spec sentence.
6. **Adversarial Inputs** — Construct oversized, malformed, reserved-name, recursion-bomb, boundary-encoding inputs. Surfaces a distinct bug class from correctness-focused rounds.

Each angle should be audited **independently** — different agents, no shared findings — to avoid groupthink. Rotate angles across audit rounds; convergence is meaningful when rounds from different angles agree, not when a single angle saturates.

## Common Pitfalls

- **Overstating coverage**: "PROVEN" means you traced every path. If you skipped an edge case, it's "PARTIAL".
- **Trusting types**: TypeScript types can be circumvented at runtime (any, type assertions, external data). Verify runtime guards exist.
- **Ignoring framework chaining**: If multiple plugins modify the same data (e.g., tool_result content), the Nth handler sees modified data, not the original.
- **Assuming single-call**: A function proven correct for one call may break invariants when called in sequence (e.g., cycle detection that only checks one update at a time).
- **Missing resource paths**: A try/finally for cleanup is correct ONLY if the allocation is INSIDE the try. If between allocation and try, an exception leaks the resource.
- **Summarizing instead of annotating**: "This function sorts an array" is a summary. "After line 5, arr[0..i) is sorted and arr[i..n) is unchanged" is an NSP. Always write NSPs.

### Rule 6: Lesson/spec citations must verify their preconditions

When citing a Recurring Lesson or a spec constraint as evidence for a finding, you MUST verify that the lesson's **preconditions actually hold** in the current code context. Lessons are conditional rules, not universal laws.

Validation process:
1. State the lesson being invoked and its implicit precondition
2. Check whether the precondition holds in the current code path
3. If it does → the lesson applies, cite it
4. If it doesn't → the lesson is inapplicable, do NOT use it as evidence

Example of the mistake:
```
# WRONG: "Lesson B14 says 'tolerant parsers hide user mistakes, prefer strict' 
#          → this lenient parse is a bug"
#   (B14's precondition is 'human-authored config' — but this code parses 
#    machine-generated internal data where leniency is intentional)
#
# RIGHT: "B14 applies to human-authored config. This parser handles 
#          machine-generated wire data where the producer is trusted. 
#          B14's precondition does not hold here — not a finding."
```

The same applies to design-doc constraints: a constraint scoped to "external input" does not apply to internal module-to-module calls. A constraint scoped to "persistent state" does not apply to ephemeral in-memory buffers. **Always check the scope.**

## Recurring Lessons

Brief reminders of patterns seen repeatedly across audit loops. Language-agnostic — concrete manifestations vary.

- **Identity vs equivalence in derived values.** Transformations (promise chaining, mapping, cloning) produce a new identity each call; don't compare by re-deriving. Store once, compare the stored reference.

- **Reserved-name keys from untrusted input.** Any collection keyed by user/LLM strings must reject names the runtime treats specially (`__proto__`, `constructor`, shell builtins, etc.). Enumerate and reject at the boundary.

- **Round-trip filtering is silent data loss.** A narrowing reader paired with a round-trip writer erases out-of-shape data on next persist. Preserve unknown shapes, or reject at write time.

- **Commit ordering across files.** When two files must agree, designate one as the commit marker; write the other first. A crash between then leaves a consistent pre-transition view.

- **Subtree cleanup on signal.** Killing a process doesn't kill the subtree it spawned. Use the platform primitive (process groups, job objects) if the child can spawn background work.

- **Multi-source settlement.** When several events can settle a state (timeout, completion, error), use an idempotency flag AND record the source — some side effects should run only for specific sources (e.g. "kill child" after timeout but not after natural close).

- **Tolerant parsers hide user mistakes.** Skipping unexpected input absorbs the user's error as a silent default. For human-authored config, prefer strict.

- **Forward ≠ reverse reachability.** "Path from start to end exists" doesn't imply "every node can still reach end". Run the reverse traversal when the invariant is "no dead ends".

- **Hook callbacks run in swallow-errors envelopes.** A throw in a framework callback often disables the callback silently. Wrap the body; surface real errors explicitly.

- **Line citations drift.** `file:line` is stale after the next edit. Prefer function-name anchors, unique-string anchors, or a "lines are approximate" caveat.

- **Dead code is a signal.** Unused exports, unreachable branches, stale comments are often remnants of design changes. Before deleting, verify whether an invariant at a distance makes them "unreachable"; if so, keep as defense-in-depth.

- **Disk is authoritative; memory is a cache.** For state that crosses process boundaries, design the persisted form first and treat in-memory as derived.
