---
name: recalibration
description: Reconstruct and test the direction of long-running work across independent lines and their stacks. Use to regain focus, check whether the current path is justified, or recover progress and the knowledge needed to continue.
---

# Recalibration

## Purpose

Restore the user's global work orientation and current direction. Explain the active portfolio through its working stacks, evidence-backed progress, selected top, next action, and compact popped-frame history.

Do not merely summarize state. Recover a bounded working understanding of the user's purpose, test whether the current focus and its necessary parent plans still have reasons to continue, and directly correct the analysis, recommended order, or next recommendation when they do not. Returning to the same purpose does not mean returning to the earlier state of knowledge: a candidate may lose its reason while discoveries made through it remain relevant.

A point-by-point dialogue critique of whether one latest statement was understood or correct remains a separate dialogue-review task. That distinction does not demote an invocation correction: when it bears on current direction, assess it with the request history, explicit decisions, and other authority.

## Invocation material

$ARGUMENTS

Use the supplied request to establish this round's scope, question, corrections, and explicit authorization. Read it with the relevant history; honor explicit updates rather than silently restoring superseded intentions. Distinguish a request to calibrate from an explicit instruction to plan or execute: passing that instruction through this template neither removes its authority nor enlarges its scope. Inferred intent does not itself authorize execution.

## Evidence and authority

Reconstruct state and direction from available conversation history and, when present:

1. user requests, corrections, decisions, priorities, acceptance, and abandonment;
2. authoritative object facts, actual files, diffs, tests, and published results;
3. native task state for readiness, dependencies, and execution status;
4. parent verification and accepted design or task relationships;
5. master or mission ledgers as readable portfolio mirrors;
6. worker reports as evidence awaiting verification;
7. bounded conversation inference only where it helps explain the task's evolution or current meaning.

Keep distinct: original requirements, object or domain facts, accepted plans or local contracts, factual task states, and this round's inferences. If they conflict, report the conflict rather than silently choosing a convenient state.

Behavior and context can support a bounded interpretation, not mind-reading, personality claims, or proof of psychological cause. Compare the actual request with explicit corrections, acceptance or rejection, the object's use, and the work situation. Explicit correction outranks inference. Do not treat silence, continued discussion, emotional intensity, repeated terms, downstream detail, or a coherent narrative as full acceptance or new authorization.

## Recover and test direction

Do this direction check during every recalibration, not only after an obvious failure or before a separate reflection exercise. Open only the history and materials needed to resolve a live reason, conflict, or uncertainty; do not default to a full historical audit.

### 1. Suspend recent framing

Set aside the most recent high-frequency terms, AI-generated frameworks, process organization, and the current candidate long enough to reread the relevant request, corrections, acceptance or rejection, and authoritative evidence. Do not let a useful method, current artifact property, or recently active side line replace the task it was meant to serve.

### 2. Form a working understanding that can guide a choice

State what the user is trying to change, achieve, protect, or avoid; what the object is for; and the actual constraints and facts that bear on the live work. Compare plausible interpretations against the evidence above. A recovered understanding is adequate only if it can explain how to understand the current task, what to retain or exclude, whether order should change, or why a next move is warranted; a story that merely sounds coherent is not enough.

### 3. Trace the reason chain

Check the current direction of each live line within the requested scope. Trace its local obligation through the parent responsibilities needed to reach a sufficiently supporting higher purpose; do not default to auditing every historical branch. Each link needs a user decision, accepted design relation, or actual task relation. A child task's need cannot prove its parent necessary, and a root purpose cannot bypass an accepted local contract to justify arbitrary detail. Lack of verbatim permission is not itself a bar to a valid inference grounded in the request, facts, and accepted contract.

### 4. Separate history from current reassessment

For relevant historical material, distinguish what happened and what reasons were supportable then from what this round now concludes about its significance. Historical occurrence does not require present continuation, and present uselessness does not prove earlier lack of reason. Do not use a later theory, finished artifact, or downstream dependency to fabricate the original motive or authorization.

Retain verified results still needed by the live reason chain, confirmed rejection boundaries, object relationships, negative counterexamples, and unresolved clues with their conditions. Withdraw the authority of unsupported process residue over the current interpretation; do not erase the historical record. Discovering a relation through one candidate neither confines it to that candidate nor makes it mandatory for every implementation. Call a relation necessary only when the original requirement and relevant facts support that necessity; repeated encounters with it are not proof. A non-necessary mechanism is not thereby forbidden.

### 5. Turn the understanding into direction

Use the recovered understanding to confirm or revise the current interpretation, candidate, strategy, ordering, or smallest next action. Say what it supports, excludes, and leaves uncertain. Improved judgment may legitimately leave the same plan and next action in place; do not manufacture a new concept, task, dependency, or action merely to show movement.

## Deepen only when a concrete understanding problem remains

Recalibration itself performs the preceding interpretation and reason test. Load and apply the complete `reflect-and-proceed` skill when continuing well requires re-forming or deepening the working understanding of the original problem. Identify the understanding question from the recovered material; a user who asks to step back need not supply a diagnosis first. This threshold is about the need for deeper understanding, not apparent severity, task age, a fixed cycle, or whether work has visibly drifted.

Use the available skill or resource-loading mechanism to load and apply the complete skill; where installed, its native user command is `/skill:reflect-and-proceed`. Mentioning a slash name is not loading it. Pass the recovered original problem, explicit decisions, relevant facts, still-valid exploration results, and the precise question; reading the method does not add execution authority.

On return, check the skill's sources, inference labels, authorization boundary, and factual states before integrating its deeper understanding, supported or excluded implications, conditions or gaps, and continuing recommendation into this calibration. Do not create a mutual invocation loop: the skill does not call recalibration back, and recalibration does not automatically reload it merely because an open question remains. The responsible method may refine its reasoning with relevant material until its completion condition is met; a distinct later understanding question must be scoped and justified rather than triggered by a fixed pass count or a required new user turn.

If the complete resource is unavailable, identify the specific missing material, do the bounded direction check that the available evidence supports, and do not claim that deeper reflection was completed.

## Build the portfolio

### 1. State the global goal

Start with one sentence in the user's task language. Assume the user remembers the task they created; restore coordinates instead of reteaching it.

### 2. Separate working lines

Create a new working line only when an item has an independent goal, completion condition, or responsibility boundary. A helper, method, stage, or temporary problem does not become a separate line merely because it recently received attention.

Keep queued, active, and blocked lines visible in the current view. Omit completed and expired lines there by default; preserve their frame-level evolution in the separate popped-frame view.

### 3. Rebuild each working stack

Within each live line, reconstruct only its current frame stack:

```text
root live frame → suspended live frame(s) → current top
```

For every live frame recover:

- **Push:** why a parent task was suspended and this context was entered;
- **Work:** what this frame must complete;
- **State:** queued, active, blocked, returned, verified, accepted, or expired;
- **Pop:** what evidence-backed condition closes this frame.

A non-root frame requires an actual context switch and one suspended parent task. Strict LIFO stack order uniquely determines the parent; do not add a separate return field. Ordinary sequential, parallel, verification, and rework tasks remain in the frame's task graph rather than becoming frames.

When an accepted or expired frame is popped, retain one compact record under its original working stack:

- **Main focus:** the frame's local goal;
- **Push:** why it was entered;
- **Result:** what it established or why it expired;
- **Evidence / live residuals:** only what is needed to understand or audit that result.

Keep popped records compact. Expand related attempts or discarded options only as needed to assess a live justification, recover relevant learning, or answer the user's request; do not promote those ordinary tasks into stack frames.

### 4. Calibrate progress

Do not collapse distinct states:

```text
planned ≠ dispatched ≠ returned ≠ verified ≠ accepted
```

Write `done` only for verified or accepted work. Preserve task evolution in its owning working stack, but keep popped frames compact and separate from the current control view. Omit chat-by-chat noise and raw orchestration logs. A direction recommendation never rewrites factual execution state.

### 5. Answer the current-state questions

For every live line answer:

- **Goal:** what final outcome does this line seek?
- **Stack:** which live frames lead to the current top?
- **Established:** which verified results and supported discoveries from live or popped work still support this line, and under what conditions?
- **Current:** what is selected, running in the background, blocked, or awaiting verification?
- **Next:** what is the smallest current action?

Keep new interpretations and unresolved clues visibly tentative rather than presenting them as established results. Then identify the portfolio's sole selected focus without treating it as a ban on asynchronous progress in other working stacks.

## Output shape

Render two views over the same underlying task and stack state; include only the direction evidence needed to make the current judgment intelligible, not a large reflection report.

### Print A — Current Stack View

```text
Global goal:
- ...

Direction check:
- Recovered understanding:
- Reasons that still support / no longer support the live directions:
- Boundaries, conditions, or unresolved evidence:
- Direction and next move:

Working line A — <name>
- Goal:
- Stack: root live frame → ... → current top
- Established: <verified results and supported learning from live or popped work; relevant conditions>
- Current: <selected / background / blocked / verification>
- Next:

Working line B — <name>
- ...

Portfolio focus:
- Selected:
- Background:
- Queued / blocked / verification:
- Look at now:

Final sentence: <global goal + actual state + justified next move>
```

### Print B — Popped Frame View

Group compact popped records under the working stack that owned them:

```text
Working line A — <name>
- [popped] <main focus>
  - Push: <why this frame was entered>
  - Result: <what it established or why it expired>
  - Evidence / residual: <only when still useful>
```

Do not promote ordinary completed tasks to popped frames. A small portfolio with no popped frames may omit Print B.

## Guardrails

- Do not begin with the latest worker, reviewer, run ID, or callback unless it defines the global state.
- Do not let the most recently active side line erase the main line or define its purpose.
- Do not dump task or ledger rows without translating them into the user's goals and the reasons that still support them.
- Do not invent dependencies, make an ordinary task a stack frame, or turn a discussed method into an independent goal.
- Do not hide unresolved names, mappings, conflicts, missing evidence, or a break in the reason chain.
- Do not mix popped-frame evolution into the Current Stack View or flatten it into a global history list.
- Do not let Print A and Print B maintain conflicting state; both are projections of the same native tasks, evidence, and stack structure.
- A calibration request permits correction of this analysis, report, direction judgment, and next recommendation; it does not by itself authorize execution, change project or task state, or settle user-reserved tradeoffs.
- Preserve explicit, still-valid authorization in the current request or prior mandate; do not enlarge it because an intent was inferred, an analysis was performed, or a skill was loaded.
- End with one short sentence that lets the user recover goal, present position, and justified next move.
