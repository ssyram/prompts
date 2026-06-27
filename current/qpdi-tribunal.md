Use the `qpdi` skill and read `references/tribunal-scco.md` if needed.

Review target:
$ARGUMENTS

Run a lightweight tribunal-style review. Do not claim to execute the QPDI FSM unless explicitly asked to work on the FSM implementation.

Do this:

1. Identify the review target and likely QPDI layer.
2. Check SCCO:
   - Sound — correctness and contradiction.
   - Complete — missing required coverage.
   - Concise — duplicated, stale, bloated, or historical residue.
   - Optimization — simpler or stronger organization.
3. Challenge likely problems.
4. Counter weak findings and discard pseudo-issues.
5. Classify valid findings:
   - can directly fix;
   - needs user decision;
   - blocked by missing material.

Answer in this format:

```text
审查对象：
S — Sound：
C — Complete：
C — Concise：
O — Optimization：
有效问题：
伪问题：
可直接修：
需要用户裁决：
```
