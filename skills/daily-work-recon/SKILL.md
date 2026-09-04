---
name: daily-work-recon
description: Reconstruct one day's AI-assisted work from transcripts, preserving distinct conversation paths before integrating working-stack progress, energy, and lessons into a report.
---

# Daily Work Recon

## Purpose

Use this skill to help the user see one day from two connected views:

- **direct history**: what happened in the day's conversations and why the work changed direction;
- **actual work progress**: which working stacks moved, from what state to what state, and what remains open.

A **working stack** is the real work being carried forward: its question, current state, decisions, artifacts, and open threads. It is not the same thing as a transcript. A transcript is an evidence container, not necessarily one linear conversation. Notice any branches, rewinds, forks, or revised paths exposed by the source, and examine their distinct work separately. Use the source's own structure; do not impose one system's tree model on another.

Do not treat session count, tool activity, or a fixed report template as the goal. Do not add a database, stable IDs, review gates, or separate assessment dimensions unless the user asks for them or the requested conclusion cannot otherwise be supported.

## QPDI Design

- **Q**: help the user recover each relevant conversation path, understand the day's actual work and working state, and draw only grounded conclusions about what changed, what was learned, and what remains open.
- **P**: distinct paths are not collapsed into a false linear history; shared history and repeated progress are counted once; work progress is distinct from conversation history; important conclusions can be traced back; the report reflects the user's actual energy distribution across stacks; time and attention claims keep their evidence boundary; the output contains no repeated or unsupported material.
- **D**: `find` → `distinguish conversation paths when needed` → `recalibrate each transcript` → `integrate working stacks` → `analyse how` → `render`.
- **I**: inspect relevant session records and only the artifacts or command results needed to support the requested claims; write the requested report and evidence packets.

## 1. Find

### Set the boundary

Establish the local date, timezone, cutoff, allowed evidence sources, and scope: selected session sources or all work. Filter entries by their own timestamp, not filename date.

### Build the source index

1. Resolve the relevant session or transcript locations.
2. Find records with at least one entry in scope, including a user-directed execution tail when relevant.
3. Before treating a transcript as linear, inspect the lineage, branch, rewind, fork, or revision information its source actually provides. If the source provides no such evidence, do not invent branches.
4. When a transcript is non-linear, examine each path whose distinct portion has in-scope entries. Keep shared history once, and identify the current path only when the source supports that judgment.
5. Record the transcript and path locator, source lineage, distinct range, latest real user entry, examined cutoff, and material gaps.
6. Exclude synthetic delegated-task and steering messages from human-message counts, but retain them as execution evidence when relevant.

Every in-scope path must be represented. Work on a path still counts after the user leaves it, but its last answer does not become the current conclusion unless later work carries it forward. Use the supported current path for conversational state and verified artifacts for artifact state. Repeated wording alone is not a reason to merge two actions; shared source entries and repeated work progress are still counted only once.

### Read only as deeply as needed

Use the lowest sufficient depth:

| Depth | Read | Use when |
|---|---|---|
| Index | headers, timestamps, IDs, tool names | locating transcripts and forks |
| Trace | turning-point messages and immediate results | recovering a transcript's history |
| Verify | current artifact, scoped diff, command/test result | making an outcome or correctness claim |
| Deep | full branch, nested transcript, or source set | a required conclusion cannot otherwise be supported |

Expand only when the missing material could change the transcript's history, its stack progress, or a conclusion in the final report.

## 2. Recalibrate Each Transcript

For every examined transcript, write one local recalibration. Before summarizing it, check whether it contains distinct conversation paths. If so, assess each path with in-scope entries separately; if not, use the ordinary linear structure. Cover only the working stacks that a path actually takes over or changes.

A path that only repeats inherited history should reference that history and describe only its distinct work.

Use this structure; omit the path heading when the transcript is linear:

```markdown
# <label> — <plain-language conversation name>

> Source:
> Message range:
> Examined cutoff:
> Lineage / paths when relevant:

## Summary
<What this transcript's relevant paths did today and where the current work ended.>

## <path label and current / left / unknown status, only when needed>
> Distinct in-scope range and latest real user entry:

### <working stack name>

**Entering state**
- What was being worked on:
- What was already true or unresolved:

**Direct history**
- <event> → <your response> → <what changed>

**Actual work progress**
- Before:
- Today's change:
- After:
- What did not advance, was rejected, superseded, or narrowed:

**How material**
- What you specified, challenged, verified, redirected, paused, or accepted:
- What that action changed:
- Investment signals from this path's distinct user messages:

**Resume point**
- What remains open and where work should resume:

## Evidence and limits
- Key source locations or artifacts:
- What was not checked:
- What this transcript does not establish:
```

Direct history must follow the actual path rather than assumed file order. Actual work progress records a state difference:

```text
entering state → this path's real change → leaving state
```

Do not write that work progressed merely because it was discussed, delegated, or used tools. A real change can be a clearer question, a tested or narrowed assumption, collected source material, a design decision, an implementation, a validation result, a justified pause, or an explicit abandonment.

Keep the causal form `AI material or execution → user response → resulting change` whenever it explains a turn, but do not connect separate paths unless later evidence carries material across.

## 3. Integrate Working Stacks

Read the local recalibrations and combine conversation-path episodes only when they concern the same working stack. If the relation is uncertain, keep them separate and say so.

For each stack:

1. connect its path episodes in the order that explains the work;
2. keep direct history separate from actual work progress;
3. count shared history and repeated transitions once;
4. retain what happened on paths the user left, while using the supported current path for conversational direction and verified artifacts for artifact state;
5. preserve links to the transcript paths that support each important statement.

This is the day's **What/How progress material**. Do not extract key outcomes, lessons, or reflection as a parallel account before this integration exists.

## 4. Analyse How

Only after the day's working stacks are integrated, examine how the user worked across them:

- **energy and attention distribution** (see below — a first-class part of the How analysis, not an afterthought);
- what problems received attention and when direction changed;
- how the user specified, challenged, verified, paused, delegated, or redirected work;
- which judgments changed a stack's progress or prevented a false conclusion;
- what caused waiting, rework, repeated explanation, or unnecessary process;
- what concrete lessons or AI communication methods were actually demonstrated.

Keep three things separate:

- **direct record**: messages, artifacts, command results, and timestamps;
- **user statement**: what the user explicitly says about their attention or intent;
- **inference**: a conditional reading of the first two.

### Prior knowledge vs new learning

A problem, concern, correction, rule, theory, or design request stated by the user normally shows what the user already knew or believed before asking. It belongs in direct history, the user's problem framing, or a decision record. It is not evidence that the user learned or should reflect on that same idea today.

Claim new learning only when the record after an AI answer shows uptake, such as:

- the user restates a specific answer in their own words and asks whether it is correct;
- the user changes or narrows an earlier belief because of the answer;
- the next question depends on a concrete fact or distinction introduced by the answer;
- the user makes a decision using that answer;
- the user explicitly says what they learned.

When this evidence exists, report the **concrete matter itself**: the actual mechanism, fact, distinction, or corrected understanding. Do not replace it with a deep-sounding abstraction. For example, write “the user learned that `.i` is a preprocessed TU and does not contain the real linker configuration,” not “the user learned the tension between form and reality.”

A content-matching follow-up proves that the user read and worked with that specific material; it does not by itself prove unconditional acceptance. Preserve disagreement, correction, and uncertainty.

If an answer exists but no later message shows uptake, write “the AI answered X; no later user message confirms uptake” when that boundary matters, or omit it from learning. Never mine the subject matter of the user's request for a philosophical lesson merely because the problem sounds deep.

Reflection is optional. Include it only when the user explicitly formulates a changed principle, demonstrates a change across turns, or when repeated evidence supports a bounded method update. Otherwise omit it.

### Energy / attention distribution

Determine where the user actually invested attention, and let that ranking shape the whole report: stack order in the overview table, thickness of each stack's history, and the prominence of How material. Uniform treatment of all stacks is a failure mode — low-investment stacks must not read as heavy as deliberate focus.

Judging criteria, in priority order:

1. **Content-match precision (primary signal)**: does a user follow-up reference the specific content of the assistant's recent output (a named mechanism, a quoted term, a specific claim)? A question that could only come from reading that output proves real engagement. Trust-style authorizations ("可以，修吧" / "go ahead") prove steering, not reading.
2. **Where the user's own thinking appears**: original rule-setting, theory-building, restating a concept in their own words to confirm understanding — the heaviest investment signals, regardless of message count.
3. **Message length and count (secondary)**: long, structured messages indicate effort; many short authorizations do not.
4. **User self-description is checkable, not authoritative**: the user's claim about their own attention ("我没详细读") is a statement to verify against the record, not a fact to copy. The record may show more or less precision than the user remembers.

Distinguish engagement modes, because they carry different weight:

- **rule-setting / theory-building**: the user writes the rules the work must follow — heaviest;
- **comprehension-driven probing**: user restates, asks to explain, drills into specifics of the output — heavy;
- **structural design participation**: user shapes the structure with substantive corrections — medium;
- **steering / boundary challenges**: short but precise corrections, possibly cross-session insights injected into a low-investment stack — low volume, high value; do not rank a stack high just because one such correction landed there;
- **closure authorizations**: "加上", "可以", "cont" — minimal.

All of this stays conditional inference: it supports "here is where attention plausibly went", never reading speed, total attention duration, or a stable personal trait.

### Intent vs proposition

When the user's messages contain factual self-descriptions or claims ("I only gave a couple of instructions", "I discovered this in another session"), separate:

- **intent / desire**: what the user wants done — execute it as-is; factual inaccuracies in the surrounding text do not discount the intent;
- **propositions**: factual claims about what happened — verify against the record and correct them when wrong.

Only challenge the intent itself when its factual foundation collapses entirely (every route it depends on is impossible).

### Time and efficiency boundaries

Time can describe response rhythm, content-available windows, elapsed span, or agent runtime. It does not directly prove reading speed, total attention, or a stable personal trait. Evaluate efficiency only when the goal, actual result, and relevant cost or friction are all known; otherwise report the observed progress and friction without scoring it.

### Report ordering and thickness

Order stacks by the user's energy ranking (highest first), not by transcript number or chronology. Thickness must follow the same ranking:

- the main-investment stacks get the fullest direct history and How material;
- low-investment stacks get proportionally compressed treatment — their detail lives in the evidence packets;
- note the ranking basis explicitly in the report (one short block: criteria + evidence boundary).

### Raw data for energy assessment

During recalibration (section 2), collect user message count, total/average/max length, and content-matching follow-ups vs trust-style authorizations. For a non-linear transcript, calculate these from each path's distinct messages and aggregate shared entries once; judge content matches within the same path. Record separate path rows in `evidence/INDEX.md` only when the distinction exists. Repeated work progress is consolidated later in the report.

## 5. Render

Generate only the output the user requested. When writing a broad daily log into an already chosen log repository, use:

```text
<LOG_REPO>/<YYYY-MM-DD>/
  REPORT.md
  evidence/
    INDEX.md
    transcripts/
      01.md
      02.md
      ...
```

Do not create `timeline.md`, `outcomes.md`, `human-response.md`, `ai-techniques.md`, or `time.md` by default. Add another evidence file only if the user's question cannot be expressed by the index and transcript recalibrations.

### `evidence/INDEX.md`

Record only the source map and mechanical relationships:

```markdown
# Evidence Index

> Date:
> Timezone:
> Overall cutoff:
> Scope:
> Known gaps:

| Label | Address | Lineage / path when relevant | Message range | Examined cutoff | Depth | Working stacks touched | Gaps |

User-message stats (feeds energy analysis; mechanical only):

| Label / path | User msgs | Total chars | Avg / max chars | Content-matching follow-ups | Trust-style authorizations |
|---|---|---|---|---|---|
```

Use separate rows for distinct in-scope paths only when the source supports that distinction; otherwise use one transcript row. List shared history once. Do not put final judgments or cross-transcript conclusions here.

### `evidence/transcripts/NN.md`

Write one local packet per transcript from section 2. Distinguish its conversation paths inside the packet only when needed. These files are detailed, recoverable histories, not copied session archives.

### `REPORT.md`

`REPORT.md` is the final integration. Organize it by working stack, not by a flat transcript timeline.

```markdown
# <YYYY-MM-DD> — Daily Recalibration

> Cutoff: `<ISO-8601 with timezone>`
> Scope: `<session source(s) / stated scope>`
> Sources examined: `<count and human-readable summary>`
> Evidence: [`evidence/INDEX.md`](evidence/INDEX.md)

## Today's Working Stacks

| Investment | Working stack | Supporting transcripts / paths | Entering state | Today's actual progress | Leaving state |
|---|---|---|---|---|---|

<Order rows by inferred investment, highest first.>

## Direct History

### <working stack>
<The day's conversation-path episodes, important turns, and why direction changed.>

## Actual Work Progress

### <working stack>
- Entering state:
- What moved today:
- What did not move, was rejected, or was narrowed:
- Current state:
- Resume point:

## How the Work Moved
### Energy distribution
<Where the user's attention plausibly went, by stack: criteria (content-match precision first), evidence, and boundary — conditional inference only.>
### How you made decisions
<Corrected AI, changed direction, verified work; engagement modes (rule-setting / probing / steering).>
<Concrete answer material that later user messages demonstrably took up; state the matter itself, not an abstraction of it.>
<Concrete friction and what it cost or prevented:>
<Lessons or communication methods only when the user explicitly formed or demonstrated them, with conditions and limits:>

## Key Nodes and Results
<Extract only the genuinely important decisions, outcomes, and limits already shown above. Do not create a second account.>

## Reflection and Recalibration
<Optional. Include only an explicit or demonstrable change in the user's own principle or method. A problem the user brought to the AI is prior problem awareness, not today's reflection. When the user learned from an answer, report the concrete matter under How instead of philosophically rewrapping it here. Omit this section when no independent reflection is supported.>

## Unclosed
<Only work that materially affects the next continuation or limits today's conclusion.>
```

A report may name or link the transcript paths that support a stack, but should not repeat their full content. The packets preserve each relevant path's history; the report explains how those histories jointly moved the day's real work.

## Evidence and Writing Rules

- Lead with the conclusion relevant to the user's question, then give the necessary history and boundary.
- Follow `/explain` in reader-facing prose: use plain-language headings, keep analysis labels out of the final report, and explain a necessary code or technical term at first use.
- Use the user's existing words. Introduce a new term only when failing to distinguish it would change the conclusion or next action.
- Do not use “tension”, “bridge”, “candidate”, QPDI layer labels, or philosophical vocabulary merely to organize prose. Retain such a term only when it is the user's established term or the exact distinction changes the judgment, and explain it immediately.
- A user question or correction is not evidence of learning. Require post-answer uptake evidence, and report the concrete learned matter rather than an abstract lesson.
- State what a result supports and what it does not support.
- Verify artifacts only for strong claims: a research conclusion, design, implementation, test result, or scoped change.
- A test invocation is not a passing test; a staging directory is not a completed delivery.
- A single-day observation is not a general personal pattern without a stated bridge and scope.
- Never attribute an entire dirty worktree to one day without dated, scoped evidence.
- Keep raw transcript bulk out of evidence packets. Quote wording only when the wording itself matters; otherwise summarize the decision and retain locators.
- Do not fabricate progress, a lesson, an efficiency judgment, an attention claim, a key result, or a reflection to complete a section. Omit an empty section.
- Do not initialize a daily-log repository, edit project artifacts, run tests, commit, or clean worktrees merely to produce the reconstruction unless explicitly asked.
