# Rewrite 85 writer result

## Changed deliverables

- `本归档/SYNTHESIS.md`
  - Rewritten around the chain from actual report task to required content, selected expression, and integrated verification.
  - Separates task constraints, operational methods, conditional 指定读者-style defaults, and examples.
  - Explains the root failures behind checklist/template substitution, preserves the ten prior writing sources and their read limits, and adds bounded Occam references (Q.A.2, architecture, E03–E12/F14–F30).
  - Uses Tool Auth Lang and the July program-behavior report as organization examples without turning their project claims into general skill evidence.
- `本归档/SKILL-REVISION.md`
  - Rewritten as a concise installation proposal aligned with the candidate rather than the superseded five-step structure.
  - States preserved protections, replaced rules, exclusions, 指定读者-style default role, and future replacement target.
- `本归档/SKILL-DRAFT.md`
  - New standalone candidate, directly portable as `write-technical-report/SKILL.md`.
  - Has valid frontmatter, general-purpose task-first construction, conditional opening/default organization guidance, body/case guidance, modifier handling, and whole-document verification.

The installed `本地代理资料（未随归档提供）` and its reference file were not changed.

## Checks run

- Markdown local-link, fence, and trailing-whitespace validation for all three deliverables.
- Citation validation: every numeric reference in `SYNTHESIS.md` has a matching source row; source rows 1–13 exist.
- Candidate YAML validation: `name: write-technical-report`, valid name pattern, description length within 1–1024, and 153 lines (<500).
- Snapshot comparison: only the two pre-existing authorized files changed; `SKILL-DRAFT.md` is new; ten protected files remain byte-identical.
- `git diff --cached --name-only`: empty.

## Remaining issues

- This is a candidate only; user approval is still required before replacing the installed skill.
- `README.md` still describes the earlier revision as a “five-step structure.” The authorized scope allowed only the three deliverables, so it was not changed.
- No new 指定读者 reading test or external writing research was run; the source limits stated in `SYNTHESIS.md` remain in force.

```acceptance-report
{
  "criteriaSatisfied": [
    {
      "id": "criterion-1",
      "status": "satisfied",
      "evidence": "Only the authorized SYNTHESIS.md and SKILL-REVISION.md were rewritten and the authorized new SKILL-DRAFT.md was created; installed skill and protected files remain byte-identical."
    },
    {
      "id": "criterion-2",
      "status": "satisfied",
      "evidence": "Validated links, citations, Markdown, portable YAML frontmatter, protected-file hashes, expected scope, and absence of staged files."
    }
  ],
  "changedFiles": [
    "本归档/SYNTHESIS.md",
    "本归档/SKILL-REVISION.md",
    "本归档/SKILL-DRAFT.md"
  ],
  "testsAddedOrUpdated": [],
  "commandsRun": [
    {
      "command": "python3 validation for Markdown links/citations/YAML/frontmatter/protected hashes",
      "result": "passed",
      "summary": "All three deliverables validated; 13 cited source rows; candidate is under 500 lines."
    },
    {
      "command": "python3 snapshot comparison and git diff --cached --name-only",
      "result": "passed",
      "summary": "Only authorized changes; ten protected artifacts unchanged; no staged files."
    }
  ],
  "validationOutput": [
    "PASS: 3 deliverables; local links; 13 cited source rows; Markdown fences/whitespace; portable YAML frontmatter; candidate <500 lines; only authorized documents changed; protected files unchanged.",
    "PASS: exact pre-existing target changes are SYNTHESIS.md and SKILL-REVISION.md; SKILL-DRAFT.md is new; ten protected artifacts byte-identical; no staged files."
  ],
  "residualRisks": [
    "Candidate is not installed.",
    "README.md remains stale because it was outside the authorized three-file scope.",
    "No new 指定读者 reading test was performed."
  ],
  "noStagedFiles": true,
  "diffSummary": "Replaced the theory synthesis and revision plan with task-first, 指定读者-default guidance and added a standalone candidate skill.",
  "reviewFindings": [
    "no blockers in the authorized three-file rewrite"
  ],
  "manualNotes": "No production code or installed skill was modified."
}
```
