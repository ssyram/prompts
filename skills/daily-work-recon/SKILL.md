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

- **Q**: help the user recover each relevant conversation path, understand the day's actual work and working state, resume still-relevant work across days, and draw only grounded conclusions about what changed, what was learned, and what remains open.
- **P**: distinct paths are not collapsed into a false linear history; shared history and repeated progress are counted once; directing effort, understanding gained, and actual work progress remain distinct; important conclusions can be traced back; the report reflects the user's actual energy distribution across stacks; time and attention claims keep their evidence boundary; execution and user reception remain distinct; cross-day reminders serve an evidenced next step; the output contains no repeated or unsupported material.
- **D**: `find` → `distinguish conversation paths when needed` → `recalibrate each transcript` → `integrate working stacks and relevant carryovers` → `analyse how` → `render`.
- **I**: inspect relevant session records, recent reports, and only the artifacts or command results needed to support the requested claims; write the requested report and evidence packets.

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

### Incremental collection with find-all.py

For incremental updates (`more`), start with the collector shipped in this skill:

```bash
python3 <skill-dir>/scripts/find-all.py --since 'YYYY-MM-DDTHH:MM' \
    [--until '...'] [--anomaly 'KEY,...'] [--out FILE]
```

It auto-discovers Pi (`~/.pi/agent/sessions`), Claude Code (`~/.claude/projects`), Codex (`~/.codex/sessions`) JSONL sessions and OpenCode SQLite stores, normalizes timestamps to Asia/Hong_Kong, and lists every session's user-role messages (`{provider, path, session, messages:[{time,text}]}`). It does no filtering itself.

Then work the list in three moves:

1. **Filter obvious system prompts** — drop auto templates: supervision/watchdog loops, delegated worker briefs (`Task:`, `你是已派发…`), probe messages (`say hi`, `noop`, `go`, `Reply OK`), `Inspect subagent`, steering/queued follow-ups, task-system echoes. User-role alone never proves a human.
2. **Trace the causal chain before each surviving message** — for each real human message, read the source between the previous boundary (since/last message) and this message, entry by entry, to reconstruct what this message is responding to. Never summarize a user remark without its trigger; never report a ruling stripped of the incident that caused it.
3. **Only then organize** into the normal report structure.

Run with `--anomaly` (邮件/email/SMTP/外发/暂停/越权/凭证/端点/卡死…) for an all-role incident scan. "No new human input" never means "no major event": autonomous-time incidents (safety pauses, outbound attempts, endpoint receipts) are mandatory content even when no human spoke. Huge files are streamed once; never tail-sample only — mid-file exchanges are exactly where incidents live.

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
- What that action changed in understanding or the work, beyond another presentation of existing material; supporting response/action:
- Investment signals from this path's distinct user messages, without treating effort as impact:

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

### Contributions across working stacks

Before judging an activity's contribution or cost, recover its evidenced effects on other stacks, projects, or later work, not only its local outcome. Keep independent goals separate while showing what crossed their boundaries: an artifact used, a prerequisite supplied, a judgment changed, or a conflict that caused rework.

Trace `activity/result → receiving work or understanding → concrete use or value`. Support the connection with actual reuse, explicit handoff, user explanation, or a clearly stated inference; shared vocabulary and temporal proximity alone are insufficient. Later usefulness does not prove the original activity was necessary or justify every step taken to obtain it. Count shared contributions and final outcomes once rather than awarding the same outcome anew to every supporting node.

This is the day's **What/How progress material**. Do not extract key outcomes, lessons, or reflection as a parallel account before this integration exists.

### Ongoing work across days

Preserve continuity of real work, not a second todo inventory: the user should see what to inspect, wait for, or resume without rereading the history.

- Each day, reconcile today's stacks with recent reports and previously open items. Carry forward still-relevant obligations even without today's conversation; retire completed, superseded, or abandoned ones.
- For each material item, keep its last dated substantive change, current evidence/cutoff, and next needed action or person. Distinguish AI execution, delivered results awaiting review, and actual blockers. No feedback does not prove unread work; an old running status does not prove present activity. Ordinary completed answers do not automatically become review obligations.
- Use comparisons to flag potentially dropped work, missing support or reception, and absence of observed progress. Tie reminders to a still-valid goal and useful next step; prioritize consequence and user priorities, not age alone. Missing days are unknown, not proof of stagnation.
- Start with existing reports; check only source tails or receipts that could change a significant reminder. Reuse resume points in one ongoing-work section, link rather than repeat history, and do not rerun project audits or restart old tasks merely to review their status.

### User TODO inventory in the daily stack

The steward maintains the user's complete TASK/TODO list inside the daily working stack (owner ruling, 2026-11-26):

- Everything the user needs to do is tracked as a root frame / starting item in the day's stack — including items **not yet started but needed**, items the user states explicitly, and items observed that the user is actually doing. A newly stated todo is pushed as a frame immediately.
- When a task completes, report the completion in that day's report/`more`. Unless the user says otherwise, **pop the frame by default** — it is no longer maintained afterwards. A pop records completion, risk clause, and assessment, per the standing done format.
- Do not resurrect popped tasks. If the goal re-emerges, it enters as a new frame with fresh evidence.

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

### Effective contributions and uses

Ground a contribution in either of these non-exclusive bases:

1. **Observed use or well-supported expected use:** something was actually adopted, or concrete deployment, commitments, or dependencies make its later use reasonably likely. Distinguish used, expected, and merely available; a file's existence alone is insufficient.
2. **Exploratory, understanding, or prospective value:** the work itself clarified a consequential question, established a useful exclusion, produced something for an agreed feedback process, or has another specific purpose recognized by the user or supported by a clear inference. State that purpose and its grounds; vague "might be useful someday" does not qualify. Apply the uptake rules below when claiming user learning. User approval does not itself establish technical correctness or realized benefit.

An upstream activity can be a contribution node when it demonstrably supports one of these uses or values, including outside its originating project or day. It need not produce an independently usable final artifact. Ground the chain in a supported purpose rather than circular claims that activities justify one another.

Exclude talk or output with no supported contribution from the effective-contribution summary. Missing follow-up alone means unknown, not worthless; foreseeable use remains prospective until realized. Substantial effort without effective progress may still belong in the history or a grounded lesson, not as an achievement.

### Prior knowledge vs new learning

A problem, concern, correction, rule, theory, or design request stated by the user normally shows what the user already knew or believed before asking. It belongs in direct history, the user's problem framing, or a decision record. It is not evidence that the user learned or should reflect on that same idea today.

Claim new learning only when the record after an AI answer shows uptake, such as:

- the user restates a specific answer in their own words and asks whether it is correct;
- the user changes or narrows an earlier belief because of the answer;
- the next question depends on a concrete fact or distinction introduced by the answer;
- the user makes a decision using that answer;
- the user explicitly says what they learned.

When this evidence exists, report the **concrete matter itself**: the actual mechanism, fact, distinction, or corrected understanding. Do not replace it with a deep-sounding abstraction. For example, write “the user learned that `.i` is a preprocessed TU and does not contain the real linker configuration,” not “the user learned the tension between form and reality.”

A specific content-matching follow-up supports engagement with that point, not full-document reading, new learning, acceptance, or project control. Generic praise and requests for clearer, shorter, or differently structured versions do not establish uptake. Preserve disagreement, correction, and uncertainty.

If an answer exists but no later message shows uptake, omit it from learning; include the answer in direct history when relevant. Mention missing reception only when it affects a substantive judgment or an actual review need, not as a routine disclaimer. Never mine the subject matter of the user's request for a philosophical lesson merely because the problem sounds deep.

Reflection is optional. Include it only when the user explicitly formulates a changed principle, demonstrates a change across turns, or when repeated evidence supports a bounded method update. Otherwise omit it.

### Directing effort is not project control

Judge contribution by what changed in understanding or the work, not time spent directing, instruction count, or versions produced. Closely examine substantial steering with little demonstrated progress.

- Compare the actual content and consequences of successive instructions: rewording/reformatting, new facts or reasoning, and changed decisions or execution are not interchangeable. Name the concrete change; do not classify solely by the requested format.
- Follow the material into specific uptake, a judgment or action, and its observed effect where evidence exists. Do not invent missing links. A useful artifact can exist without user learning; understanding can improve before implementation; existing understanding can also support an effective decision without new learning.
- Treat a clearer-looking version as preparation until its intended use is demonstrated, unless the representation itself is the required deliverable. Repeated rewrites, abstraction, sentence/paragraph readings, or generic approval do not by themselves show learning or control. If a rewrite reveals a real gap or changes a decision, preserve that substantive gain.
- No demonstrated uptake means uptake is unknown, not that the user definitely did not read or that the material is useless. Preserve anxiety or reassurance-seeking as the user's self-interpretation unless independent evidence supports the causal claim.
- When this distinction matters, revise the overview, history, reflection and next step together. State whether the next need is to use/understand existing material or to transform it for a specific unresolved obstacle; do not automatically prescribe another version or add a scoring ritual.

### Energy / attention distribution

Determine where attention went to allocate history and explain friction, not to award progress. Give deliberate focus proportionate detail while prominently showing its actual gains or lack of them; a busy, well-directed-looking conversation is not itself a successful working stack.

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

Time can describe response rhythm, content-available windows, elapsed span, or agent runtime. It does not directly prove reading speed, total attention, or a stable personal trait. Judge whether a contribution exists separately from whether the effort was proportionate: useful results can be obtained inefficiently, and effort does not create value by itself. Evaluate efficiency only when the goal, actual result, and relevant cost or friction are known; otherwise report progress and cost uncertainty without inventing a ratio or score.

Assess the increment against what was already available. If a lengthy discussion merely recovers an existing rule, record recall rather than a new design contribution; credit changed understanding or use only when supported. Before adding to a skill, plan, or report, compare the proposed requirement with its existing content. Do not manufacture additions to justify the discussion. When substantial repetition produced no effective increment, retain the cost or lesson without promoting it to a contribution node.

### Report ordering and thickness

Lead with what advanced the user's goals and any consequential shortfall, not transcript order or activity volume. Use attention to allocate explanatory detail, never as a substitute for contribution:

- give major investments enough history to explain both gains and costly non-progress;
- compress minor threads while retaining a brief but consequential decision or discovery;
- distinguish substantive direction from repeated presentation requests in the main account; keep mechanical statistics and assessment instructions in evidence files.

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

### Reader-facing content

The report is about the user's work, not how you assembled it.

- Never include collection or update narration, verification/compliance disclaimers, counting or classification rules, self-instructions, or reminders about how the report must be written. Apply the rules privately; write the actual event, result, and current state. Do not announce that you omitted this material either.
- Keep source maps, mechanical statistics, collection limits, and method notes in the index or evidence packets. The report needs only a brief cutoff, necessary scope, and evidence links.
- Keep a factual limit only where it changes a substantive conclusion or next action. State it once and directly: “draft delivered; not installed”, not “delivery must not be mistaken for installation”. Failures and checks in the user's work remain relevant when they explain its outcome; the report writer's compliance checklist does not.
- For incremental requests, reuse existing material and read only relevant additions. Update affected overview, history, conclusions, and ongoing-work entries in place; remove stale states rather than narrating their replacement. Do not append a supplement unless requested.

Describe reconstruction itself only when the user asks to assess that work; even then, report actions and consequences, not self-instructions.

### `evidence/INDEX.md`

Record the source map, mechanical relationships, and necessary collection/method notes:

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

### Steward working stack (steward mode)

When the user has granted the report assistant steward duty (2026-11-05 authorization: "你作为汇总者正式拥有了帮我管理多线的职责"), after rendering the report also maintain the day's steward working stack in the log repo:

- Create or update `ws-<YYYYMMDD>` via the `working-stack` binary when available (`.working-stack/` in the log repo); fall back to a hand-maintained Markdown stack only when the binary is unusable.
- Leaves = each work line's latest real progress (one frame per line; a session may own several lines). Frame titles prefixed 【等待你】 mark lines waiting for the user's own confirmation/ruling; all other leaves are executing lines. The binary's single-TOP focus model must not be read as "paused = stopped" — real status lives in each frame's `goal` text.
- Report the line inventory in the reply using the steward format, in plain language (/explain): one block per line, opening with a one-sentence 背景 recap of what the whole thing is (assume a conservative memory: no abbreviations or codewords without a gloss, but never re-teach what the user personally said), then 前沿 = latest progress and newest facts. Lines marked 【等待你】 must state the awaited action concretely: what to do, where (TUI/command/file), and what stalls without it. Executing lines report only the frontier, no decision-nagging.
- Frame advance/transition follows **the user's messages only**, never AI execution behavior: a line's frame changes state only when the user signals an explicit attitude — 明确继续基于当前讨论推进 (advance), 明确接受 or 明确拒绝 (close). AI-side progress updates the `goal` text, not the frame lifecycle.
- **Silent-success tails**: when a task on a line completes silently (the user ordered it but has not seen the result), do not drop it — keep a 【已毕待知】 frame and remind the user in every steward report until an explicit signal (advance further, accept, or reject) arrives; before that, assume the user does not know. Such frames carry across days.
- **Universal done format**: every done carries **risk clauses and an assessment** — no done is complete; risk differs only in size. Pop records and report summaries uniformly write: **outcome + risk clause + assessment** (assessment = mastery mode [hand-calibrated / judged from output / fully delegated] and confidence [low/medium/high]). "Risk-default done" (user has not examined details, judges by final effect, no incident = OK) is only the most conspicuous typical form, not a separate class; when risk is salient the record must make it felt, never passing it off as reviewed acceptance.
- **Attitude inference for pending results (core abstraction)**: for every pending work/temporary result, infer the user's actual attitude from their cross-session temporal causality — did this work (including interim results) later manifest in some way, or not at all? Silence may mean default-acceptance, forgetting what the session was doing, or a temporary topic switch; distinguish these. Handling spectrum: most conservative = legitimately treat as "changed but unconfirmed" and remind the user they may have forgotten (fully correct and acceptable); smarter = notice the deliverable being directly exercised by later work (e.g. the round after a bare "more") = tacit live testing ("test state" is merely a typical case of this abstraction, not the mechanism itself).
- **Three-day expire rule**: a pending item repeatedly ignored may be in default or expiring; **three days with no further advance and no user extension = default expire** — stop reminding, but record one line in that day's report that the item expired (expire carries larger risk; it is not acceptance).
- **Engineering discipline**: reusable operations are固化 as scripts (in `.pi/`) invoked with usage notes, never re-inlined by hand each time; the collector lives at `.pi/collect-increment.py`.
- Carry-over lines from previous days stay as frames until closed by a user decision; do not silently drop them.
- Report the line inventory (executing vs waiting-for-user) in the reply and in the report's ongoing section.

### `REPORT.md`

`REPORT.md` is the final integration. Organize it by working stack, not by a flat transcript timeline.

```markdown
# <YYYY-MM-DD> — Daily Recalibration

> Cutoff: `<ISO-8601 with timezone>`
> Evidence: [`evidence/INDEX.md`](evidence/INDEX.md)
<Add a brief scope only when needed to understand the report's coverage.>

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
<Which stacks received the user's attention, shown through concrete follow-ups and decisions; do not explain the assessment procedure.>
### How you made decisions
<What a significant instruction changed in understanding or the work; distinguish substantive direction and demonstrated uptake from another presentation of existing material.>
<Concrete answer material that later user messages demonstrably took up; state the matter itself, not an abstraction of it.>
<Concrete friction and what it cost or prevented:>
<Lessons or communication methods only when the user explicitly formed or demonstrated them, with conditions and limits:>

## Effective Contributions and Uses
<Briefly summarize the meaningful contributions already established: what changed, whom or which work it serves (including another project), and whether its value is realized, expected, or exploratory. Include necessary supporting nodes and consequential effort/friction without repeating the full history or counting a shared outcome multiple times. Use a short list or compact table, not node counts, scores, or a second results section; omit when nothing qualifies.>

## Reflection and Recalibration
<Optional. Include only an explicit or demonstrable change in the user's own principle or method. A problem the user brought to the AI is prior problem awareness, not today's reflection. When the user learned from an answer, report the concrete matter under How instead of philosophically rewrapping it here. Omit this section when no independent reflection is supported.>

## Ongoing Work and Cross-day Reminders
<Use section 3: relevant unfinished work, including dated AI execution and delivered material with an actual review need; current state, evidence and next action/person.>
<Briefly compare recent substantive changes and identify actionable gaps, with the latest dated state and next action; omit bookkeeping instructions.>
```

A report may name or link the transcript paths that support a stack, but should not repeat their full content. The packets preserve each relevant path's history; the report explains how those histories jointly moved the day's real work.

## Evidence and Writing Rules

- Lead with the conclusion relevant to the user's question, then give the necessary history and boundary.
- Follow `/explain` in reader-facing prose: use plain-language headings, keep analysis labels out of the final report, and explain a necessary code or technical term at first use.
- Use the user's existing words. Introduce a new term only when failing to distinguish it would change the conclusion or next action.
- Do not use “tension”, “bridge”, “candidate”, QPDI layer labels, or philosophical vocabulary merely to organize prose. Retain such a term only when it is the user's established term or the exact distinction changes the judgment, and explain it immediately.
- A user question or correction is not evidence of learning. Require post-answer uptake evidence, and report the concrete learned matter rather than an abstract lesson.
- Preserve factual limits that change the conclusion or next action; state them directly where relevant, without repeating generic cautions.
- Verify artifacts only for strong claims: a research conclusion, design, implementation, test result, or scoped change.
- A test invocation is not a passing test; a staging directory is not a completed delivery.
- A single-day observation is not a general personal pattern without a stated bridge and scope.
- Never attribute an entire dirty worktree to one day without dated, scoped evidence.
- Keep raw transcript bulk out of evidence packets. Quote wording only when the wording itself matters; otherwise summarize the decision and retain locators.
- Do not fabricate progress, a lesson, an efficiency judgment, an attention claim, a key result, or a reflection to complete a section. Omit an empty section.
- Do not initialize a daily-log repository, edit project artifacts, run tests, commit, or clean worktrees merely to produce the reconstruction unless explicitly asked.
