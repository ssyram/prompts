# /finegrained-check

Run a fine-grained consistency check on a target. Supports a single file, a directory, a full project, or an arbitrary topic.

Usage:
- `/finegrained-check <file-path>` - Check one document.
- `/finegrained-check <directory-path>` - Check consistency across all relevant files in a directory.
- `/finegrained-check <topic-description>` - Check consistency across all project content related to that topic.

## Execution Steps

### Phase 0: Scope Determination

Determine the inspection scope based on `$ARGUMENTS`:

**Single file**: Read it directly.
**Directory**: List all files in the directory, group by type (code/config/docs/tests), include all.
**Topic/project**: Use Glob + Grep to find relevant files (code/config/docs/tests) and define boundaries.

Output a scope checklist:
```
Scope: N files
- Code: path1, path2, ...
- Config: path3, ...
- Docs: path4, ...
- Tests: path5, ...
```

### Phase 1: Proposition Extraction

Extract **atomic propositions** from all files in scope, numbered P1, P2, ...:

**Code files**: interface contracts, function signature constraints, type invariants, error-handling assumptions, import dependencies
**Config files**: config semantics, default values, valid ranges, dependency relations among config fields
**Doc files**: design claims, behavior rules, architecture constraints
**Cross-file**: import/export contracts, caller/callee assumptions for APIs, publisher/subscriber contracts for events

Proposition format: `**Pn**: <subject> <verb> <constraint> (source: <file:line> or <Section>)`

Target density: around 5-10 propositions per 500 words/lines, with no hard upper limit.

### Phase 2: Contradiction and Missing-Coverage Checks

Cross-check all propositions (with emphasis on **cross-file** proposition pairs):

**Contradiction**: two propositions make mutually incompatible claims about the same subject.
- Format: `### Contradiction N: Px vs Py`, describe the conflict and label severity.
- Common patterns:
  - Signature mismatch (what caller passes vs what callee expects)
  - Event/field name spelling mismatch
  - Different default values for the same concept across files
  - Documentation claim vs implementation behavior mismatch

**Missing coverage**: proposition A depends on B, but no proposition covers B within scope.
- Format: `### Missing N: Px depends on undefined <B>`, explain impact.
- Common patterns:
  - Imported export does not exist
  - Tool/command/event is called but never registered
  - Config is read but never defined
  - A prompt references a nonexistent tool name

Severity levels:
- High: runtime failures, data loss, security risk
- Medium: non-deterministic behavior, test failures, feature degradation
- Low: unclear docs, redundant code, naming inconsistency

### Phase 3: Design-Point Cross-Coverage Matrix

From propositions, derive **8-20 core design points** (D1-DN). Design points may include:
- Modules/components (for example: "task system", "concurrency manager", "config loading")
- Cross-cutting concerns (for example: "error handling", "lifecycle management", "type safety")
- User flow paths (for example: "full background-task flow", "plan generation to execution")

For each pair of design points, check whether interaction coverage exists:

| A | B | Covered? | Location/Notes |
|---|---|----------|----------------|
| D1 | D2 | yes | file:line |
| D1 | D3 | **no** | Why this interaction matters and what is missing |

Highlight all **no** rows.

### Phase 4: Summary

**Key issues** (ordered by severity):
- For each issue: description + involved files + suggested fix direction

**Design strengths** (optional):
- Areas that are coherent and well-structured

**Data summary**:
- Total propositions, contradictions, missing items, matrix holes
- High-severity issue list (one sentence each)

---

## Output

If checking a single file: write report to `<same-directory>/<filename>_finegrained_report.md`
If checking a directory/project: write report to `<directory>/_finegrained_report.md`

**Writing flow** (to avoid context exhaustion):
1. Use Write to create a skeleton first (Phase headings + TODO placeholders).
2. Fill section by section via Edit, and complete each section before the next one.

## Reporting

When complete, output:
- Scope summary (file count, total line count)
- Total propositions, contradictions, missing items, matrix holes
- High-severity issue list (one sentence each)
