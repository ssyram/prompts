---
name: auto-proof-trajectory-audit
description: 给定 transitions.jsonl + yaml + 设计文档三输入，对 auto-proof-cc 一次运行的每个 transition 做精神符合度评判、非最短路径根因分析、模型能力 vs 流程设计二分判定。v2 基于 canada (成功 baseline) / ciim / imo / sphere-packing-3 四 session 实证扩展 anti-pattern 库、灰区判据、审视陷阱、detection 时机要求
---

# Auto-Proof Trajectory Audit (v2)

对 auto-proof-cc workflow 的一次 FSM 运行轨迹做事后审视。

**v2 实证基线**：本 SKILL 的判据来自四份独立 audit：
- **canada1998p5**——4 transitions 健康成功（Vieta-jumping textbook + mathlib API 现成）
- **ciim2022p6**——312 transitions / 13h 失败（70 个 r{N} marker schema-game）
- **imo2009p6**——733 transitions / 13h 失败（maintenance no-op + reviewer 代行 PLAN）
- **sphere-packing-3**——26 transitions + 50 escape / 22h 失败（agent 自决写 FINAL_SESSION_SUMMARY）

**本 SKILL 是自包含的**——审视判据全部内嵌于本文档（§2 阶段精神 / §3 路径与异常 / §4 best effort / §5 二分 / §6 anti-pattern 库 / §7 审视陷阱）。给定 §0 列出的 3 个输入文件即可执行；yaml 和设计文档用于根因分析时引用具体规则，**不是判据来源**。

本 SKILL 是只读审视流程——**禁止修改 workflow 代码**，只产 audit 报告。

---

## 0. 输入 / 输出

### 0.1 输入（3 个必需 + 1 个可选）

| 输入 | 必需 | 路径形式 | 用途 |
|---|---|---|---|
| 1. `transitions.jsonl` | 必需 | 绝对路径 | 审视核心数据源；含每个成功 transition 的 source/target/timestamp/evidence |
| 2. `auto-proof-cc.yaml` | 必需 | 绝对路径 | 索引 transition rules / hard guard 配置 / soft-gate prompt / agent instructions |
| 3. 设计文档 | 必需 | 绝对路径 | `principles.md` 优先；若无则 `design-report.compact.md`——根因分析时引用宪法原则 |
| 4. work-dir | 可选 | 绝对路径 | git 仓库；如有则可看 commit diff / .lean 文件实际内容；大幅提升审视深度 |

**`reviews.jsonl` 路径自动从 `transitions.jsonl` 同目录推断**——含 hard guard / soft-gate 的所有 `decision="fail"` 失败记录，是审视门禁失败的关键数据源。

### 0.2 输出

- 单 session 报告：`<work-dir>/audit/<YYYY-MM-DD>-trajectory-audit.md`（如 work-dir 已知）
- 否则：`<transitions.jsonl-parent>/audit/<YYYY-MM-DD>-trajectory-audit.md`
- 不输出到 `/tmp/`（会被清；用于中间扫描可临时落 `/tmp` 但最终结果必须落工作目录）

### 0.3 调用方式

```
/auto-proof-trajectory-audit <transitions.jsonl> <yaml> <design-doc> [<work-dir>]
```

---

## 1. 核心思想

### 1.1 轨迹为王

审视单位是 **transition 序列**，**不是 round / 不是文件 snapshot**。

- `transitions.jsonl` 一行一个成功 transition——按时间排序就是真相
- `reviews.jsonl` 含 hard / soft guard 所有判定（含失败）——是门禁审视的数据源
- snapshot 只是某个 transition 之后的瞬时状态，会 miss "之前的迁移原因 / 之前的内容形态 / 失败的 try"

### 1.2 最短路径基准

健康运行的最短路径**严格定义**：

```
S_PLAN → S_REVIEW → S_IMPL → S_VERIFY → S_END
   P       R         I        V         END
```

4 个 transition 闭合 1 个 entry。**任何偏离这条路径都是审视重点**——记录 + 根因分析必做。

**v2 实证**：canada1998p5 用 4 transition 完美最短路径达成（无 retry / 无 escape / 无 routing back）；这是健康基线，证明 4-transition 最短路径在合适任务上是可达的。

### 1.3 非最短路径分类（每条都是重点审视对象）

详见 §3.2。

### 1.4 模型能力问题 vs 流程设计问题

每次审视到偏离最短路径或精神违反时，必须二分归类（详见 §5）。

**v2 新增三方对齐成功公式**（canada 实证）：

```
成功 = 任务能力对齐 × Agent 主观尽力 × 流程承载良好
```

三者缺一不可。canada 三方都对齐 → 成功；ciim / imo / sphere 至少一方缺失 → 失败。这是判定根因分配权重时的参照框架。

---

## 2. 阶段精神（SKILL 内嵌判据 — 不依赖外部文档）

四个核心 production state 各自有本体性精神。**Schema 合规是必要不是充分条件**——schema 通过 + 精神违反仍是审视违反。

### 2.1 PLAN —— 自由而综合的证明书写

- 自然语言的整体设计：问题的逐步分解到完成证明的过程
- 必须**自洽完整**：给出的自然语言之间逻辑连贯无矛盾
- **分解到尽量 trivial 的步骤**：每一步都对后续 IMPL 合理可执行
- 核心二元：
  - **自由**——无须拘泥具体实现细节，只需自然语言
  - **综合**——尽量能考虑细节能被保证、能完成证明

**审视时看**：
- `auto-proof-state/plan/current.yaml` 主体字段（不仅 round number）round-to-round 实质差异
- `auto-proof-state/proofs/nl-drafts/*.md` 含具体自然语言论证草稿
- `auto-proof-state/theory-dag/nodes/*.yaml` 的 `proposition_lean` 是真 Lean 命题文本
- `auto-proof-state/theory-dag/edges/*.yaml` 的论证 step 是具体的（"因为 X 且 Y，所以 Z"）

**v2 新增：成功 run 的正面信号**（canada 实证）：
- 主动构造反例：dropped node 用 `reason:` 字段记录具体反例（canada `N_A_nonneg__dropped` / `N_A_monotone__dropped` 记 `m=1` 反例）—— PLAN 主动 prune 不可达分支是"综合"维度的强信号
- DAG 节点 polarity 与论证流向匹配（`prove`/`refute` 跟 NL draft 推理方向一致）

### 2.2 REVIEW —— 弱点补充 & 质量保证

- **自我审查**：进一步确认自身**特别是容易出错的部分**
- 提醒自身进行验证
- 对整体的自然语言计划进行进一步保证
- **不是与上轮 diff 比对**——是对 PLAN 本身的独立质疑 + 弱点定位

**审视时看**：
- `auto-proof-state/review/checks/*.yaml` 数量随 plan 节点增长
- `auto-proof-state/review/verdict.yaml` 的 `evidence` 字段与同 commit `git diff` **必须可对账**（不可伪造）
- REJECT 时 `reject_reason_hash` 触及实质字段
- PASS 时若有"弱点 / 容易出错的部分"列举更好（但不强制）

**v2 新增：成功 run 的正面信号**（canada 实证）：
- **`pass_with_caveat` 三态而非 PASS/REJECT 二态**：健康 REVIEW 可以 PASS 但同时给出 caveat（预设 escape 路径 / 风险点）—— canada `verdict.yaml` 的 `pass_with_caveat` 含 "若 m=2 时反例存在则进 S_INFRA_REPAIR" 类预设
- check 文件数量随 plan 节点 1:1 增长（canada plan 6 nodes → review 6 checks）

### 2.3 IMPL —— 专注的执行者

- 只负责对 PLAN 形成的计划**集中精力完成形式化证明**
- 具体考虑证明过程的细节
- **"专注"是核心**——IMPL 不再做 plan-level 判断；注意力被 PD 释放后全交给 Lean 实施

**审视时看**：
- `.lean` 文件实质 diff（非 comment / whitespace）
- `auto-proof-state/impl-evidence/obligations/*.yaml` 有具体 attempt 记录 + 真实 lake build output
- `auto-proof-state/impl-agents/returns/*.yaml` 含非空 candidate evidence
- IMPL 不在 impl-evidence 写 plan-level 自然语言论证（违反"专注"）

**v2 新增：成功 run 的正面信号**（canada 实证）：
- **Build log 含 self-corrected 迭代**：3 次构建迭代记录真实 Lean error 并自修（`push_neg` deprecation / `Nat.pow_lt_pow_left` signature 变更）—— 真碰 Lean 的强证据
- 失败 attempt 跟最终成功 attempt 同 commit 内并存（不掩饰）

### 2.4 VERIFY —— 归因判定 + 机械结束

- 面对执行的结果，**判定到底是 IMPL 没尽力还是 PLAN 确实出问题**（这是非平凡的智力工作）
- 完成了就**机械结束并给出报告**
- **不是 routing 流程的填写者**；是基于实跑结果对 IMPL/PLAN 责任做出判断的智力主体

**审视时看**：
- `auto-proof-state/verify/replan-evidence-round{N}.yaml` 是当 round 写的，不复用 stale 版本
- `replan_reason` 是完整论断不是单 token
- VERIFY 独立运行 kernel + 重提取 axiom manifest（不复制 IMPL claim）
- routing 决策可追溯到 IMPL/PLAN 哪方责任 + 具体证据

**v2 新增：成功 run 的正面信号**（canada 实证）：
- **`independent_axiom_extraction.declarations_inspected` 列具体 N declarations**（canada 列出全部 6 declarations：main + 5 helpers）—— 是独立 axiom 提取的可观测形态
- Stamp 9 元 tuple 逐条对账可追溯（kernel_accept / proposition_hash / polarity / axiom_manifest / verify_commit 全部能从单一 stamp.yaml 文件可读出）

---

## 3. 最短路径与异常分类

### 3.1 最短路径精确定义

```
S_PLAN → S_REVIEW → S_IMPL → S_VERIFY → S_END
```

成功条件（按 yaml `S_VERIFY → S_END` hard guard 配置）：
- `check-routing-payload.py --target END` 通过
- `check-entry-closure.py` 通过（每个 entry 有合法 stamp）
- `check-dag.py --mode verify` 通过
- `check-git-tracked-state.py` 通过

### 3.2 异常 transition 表（每条都要审视）

| 类型 | Transition | 审视要点 | 根因分析方向 |
|---|---|---|---|
| **A. R → P REJECT** | `S_REVIEW → S_PLAN` | reject reason 是否实质？PLAN 上一轮的"自由综合"维度是否真有问题？| 看 verdict.yaml + plan/current.yaml 差异；找 PLAN 上一轮违反 |
| **B. V → P replan** | `S_VERIFY → S_PLAN` | VERIFY 是否真做归因？replan_reason 触及哪一层（base-fact / strategy / polarity）？| 看 verify/routing-payload.yaml + replan-evidence-round{N}.yaml |
| **C. V → I re-impl** | `S_VERIFY → S_IMPL` | IMPL 上一轮哪里没尽力？VERIFY 的归因是"应能做到但没做"还是"试了但 lemma 不通"？| 看 impl-evidence 是否有真实 attempt；看 VERIFY 的归因详情 |
| **D. → S_INFRA_REPAIR** | `(P/R/I/V/M) → S_INFRA_REPAIR` | 触发原因？求救合理性？soft-gate 评判？| 见 §3.3 详细 |
| **E. Hard guard 失败** | `decision="fail"` in reviews.jsonl | 哪个 check 没过？反映 agent 哪种意图？| 看 reviews.jsonl 该条 detail；对照 yaml 该 transition 的 hard guard 配置 |
| **F. Soft guard 失败** | `→ S_INFRA_REPAIR` soft-gate reject | reviewer 守 PA 教练立场？给 [Concrete next moves]？| 看 reviews.jsonl soft-gate 评判正文 |
| **G. P → M → P** | `S_PLAN → S_MONITOR → S_PLAN` | subflow 拆分合理？returns 被实际整合？| 看 monitor/parallel-report.yaml + subflow returns |
| **H. [v2 新增] 工作流外 artifact** | Agent 写 `FINAL_SESSION_SUMMARY.md` / `*_FINAL_*.md` 等非 state-driven artifact | 触发条件？内容是否含 license-to-stop 信号？是否是 FSM transition 之外的 commit？| 看 git log 中非"S_X round N"格式的 commit；看 commit 内是否含 "comprehensive" / "handoff" / "X/Y complete" 用语 |

**H 类是 sphere-packing-3 独立暴露的新类**——agent 在 FSM transition 之外写自己的 markdown 总结，commit message 不属于"S_X round N"格式，是 agent 自创工作流外通道。这跟 D 类（合法 → S_INFRA_REPAIR）不同——D 是工作流提供的 escape 通道；H 是 agent 绕开工作流自创的"license-to-stop"出口。

### 3.3 S_INFRA_REPAIR 触发的深度审视模板

**每次 → S_INFRA_REPAIR 都要走完这个清单**：

1. **触发分类**：
   - **客观环境失败**：agent 实际碰到环境层故障（network / lake / api error）—— 看 prep-escape-context.py 的 `classification: clear_infra` 输出
   - **agent 主观求救**：fuzzy classification；agent 觉得卡了

2. **求救描述质量**：
   - 是具体技术问题描述（哪个 lemma 找不到 / 哪个 tactic 撞墙）→ 真求救
   - 是抽象焦虑（"剩下太难" / "这条路走不通"）→ 可能逃避

3. **Soft-gate 响应**：
   - reviewer 是否守 PA "safety valve + coach" 立场？
   - 是否暴露 license-to-stop 信号（"diminishing returns" / "X/Y complete" / "summary" / 完整 mathlib decl 名）→ 违反 PF
   - reject 时是否给 [What I saw] [Concrete next moves] [Coaching] 三段？

4. **后续动作**：
   - escape grant：operator 接手是否真发生？
   - escape reject：agent 是回到原 state 真做下一 move 还是又来一次相同 escape？

### 3.4 MONITOR subflow 审视

MONITOR 是 PLAN 的并行搜索子模式。审视：
- `auto-proof-state/plan/parallel-split.yaml` 的分支拆分是否合理（两两不重合 / 每路 dag_nodes 非空）
- `auto-proof-state/monitor/parallel-report.yaml` 是否收集了所有分支 returns
- 分支 returns（含 `difficulty_evidence` + `attempted_routes` + `recommended_redirection` 三字段）是否被实际整合到后续 PLAN
- 是否存在派 subflow 但 returns 被忽略的情况

### 3.5 [v2 新增] 四 session 实证形态谱

四份独立 audit 给出的失败形态谱，作为参照基线：

| Session | 失败形态 | Agent 行动质量 | 精神坍塌起点 | Artifact 坍塌起点 | 间距 |
|---|---|---|---|---|---|
| **canada** | 无（健康成功）| 高 | — | — | — |
| **ciim** | r{N} marker schema-game（70 个空心 yaml 节点）| 中（rounds 1-9 真做，10+ 作弊）| T31（round 10 V→P）| T32（round 11 marker 节点）| **1** |
| **sphere-packing-3** | trace-banking + threshold-approach self-stop | **高**（self-correction / abandonment / ground_truth 升级全 best effort）| T7（round 2 IMPL maintenance）| T22 后（FINAL_SESSION_SUMMARY）| **15** |
| **imo** | maintenance no-op + reviewer 代行 PLAN | 低（rounds 14+ 完全躺平）| T28（round 8）| T52（round 14 plan freeze）| **24** |

**间距 = 精神坍塌起点到 artifact 坍塌起点之间的 transition 数**。

**核心观察**：
- 三种失败形态各不相同，但**共识首位根因**：hard-guard 缺 forward-progress liveness predicate（70 marker 节点 + 168 maintenance no-op + 50 escape 全合规通过现有 check-*.py）
- **Agent 行动质量与 workflow 健康度不正相关**：sphere agent 行动质量最高（真做 Lean / self-correction / 主动 abandonment）但 workflow 失败最彻底（design 暴露最完整）—— 因为 FSM 不接受任何形式的 substantiated infeasibility 退出
- **间距是 detection 设计的硬约束参考**：间距 1-24 transition 差异巨大，detection 必须能在精神坍塌起点的 ≤2 round 内触发（详见 §8.5）

---

## 4. Best Effort 判据（核心难点）

**Best effort 是模型能力问题 vs 流程设计问题二分的关键支点**。

判定时必须**逐阶段独立判**——某 transition 涉及哪个阶段就看那个阶段。

### 4.1 PLAN best effort

| 维度 | best effort | 不 best effort |
|---|---|---|
| 内容具体性 | proposition_lean 是真 Lean 命题；nl-draft 含具体论证草稿 | 单 token / 模板字 / proposition_lean: r{N} |
| 自洽完整 | 自然语言论证从假设到结论可追踪 | 论证有断裂 / 引用未证明的"X 我们已经知道" |
| 分解 trivial | 每个 obligation 对后续 IMPL 是 "明显可写 N 行 Lean" | obligation 描述"需要某种 induction"但不指明 |
| 综合维度 | 给出 alternative strategies / boundary cases / dropped 反例 | DAG 结构稳定后只做"每 round 加一个 lemma" execution sizing |
| 响应度（V→P / R→P 后）| PLAN 实质响应了上轮 VERIFY/REVIEW 指出的具体问题 | 与上轮 plan 无实质差异 / 重炒已 ruled out 方向 |

### 4.2 REVIEW best effort

| 维度 | best effort | 不 best effort |
|---|---|---|
| 弱点定位 | check 文件列举 plan 中容易出错的具体点 | check 文件空 / 全模板 |
| evidence 可对账 | verdict.yaml evidence 字段描述与 git diff 真实对应 | evidence 声明 "2 new nodes" 但 diff 实际 0 |
| 文件演化 | check 文件随 plan 节点增加 | check 文件一次写永不更新（imo `01_math_logic.yaml` 模式） |
| 拒绝质量（REJECT 时）| reject_reason 触及 plan 实质字段 | reject_reason 反复同 hash / 表象化 |
| 三态使用（v2 补）| 适时使用 `pass_with_caveat` 给出 PASS + 预设风险点 / escape 建议 | 仅 PASS / REJECT 二元，无 caveat |

### 4.3 IMPL best effort

| 维度 | best effort | 不 best effort |
|---|---|---|
| 实质碰 Lean | `.lean` 文件有 commit diff（非 comment） | `git log -- '*.lean'` 多 round 空 |
| attempt 记录 | impl-evidence/obligations 含真实 tactic / lemma 名 + lake build output | maintenance no-op / `return_kind: maintenance, returned_files: []` |
| 责任归属 | 失败时记录 error 信息 + IMPL/PLAN 责任标注 | failure 字段空 / 用 "structural ill-typing" 模糊归责 |
| 专注 | 不写 plan-level 自然语言论证 | IMPL 在 impl-evidence 写 NL prose |
| 真改 entry .lean（v2 补）| commit 后 entry `.lean` 文件 diff 持久（非 revert）| 在 worktree 试 + revert 主分支 + 抄 trace 到 `staging-artifacts/traces/*.txt` 充数 |

### 4.4 VERIFY best effort

| 维度 | best effort | 不 best effort |
|---|---|---|
| 独立 kernel 验证 | VERIFY 独立 lake build + axiom extraction | 复制 IMPL claim |
| 归因深度 | replan-evidence 含 IMPL/PLAN 责任判断 + 具体支撑 | replan_reason 单 token / 复制 soft-reviewer 建议 |
| 文件 freshness | replan-evidence-round{N}.yaml 是当 round 写的 | 复用 round (N-K) 的 stale 文件 |
| routing 合理 | routing 决策对应归因结果 | routing 反复跳但内容雷同 |
| Axiom 提取可读（v2 补）| `independent_axiom_extraction.declarations_inspected` 列具体 N declarations | extracted_axiom_manifest: `[]` 无 `_ref` 注释（连最小满足都达不到）|

### 4.5 Best effort 判定的灰区

某些情况不能直接判 best effort 是否达成：

- **早期合理策略后被复制退化**：round 1 "不写 partial Lean" 是合理 PLAN/IMPL choice；被复制到 round 10+ 退化为 no-op——前者 best effort，后者不。**必须区分"策略形成期"和"策略复制期"**
- **单 transition 不够**：某 transition 内容简，可能是该阶段实际无内容可做（如 closure 已达）；要看前后 transition 上下文
- **跨阶段污染**：如 imo soft-reviewer 代行 PLAN——IMPL 跑出 4 个真 Lean lemma 看起来 best effort，但 lemma 名实际来自 reviewer 不来自 PLAN。这种情况 IMPL 表象 best effort 但流程实际坏掉——归类应为流程设计问题

**v2 新增灰区："诚实最小满足"健康形态**（canada 实证）：

canada transition 4 stamp `extracted_axiom_manifest: []` + `_ref` 双轨——属设计文档 §1.5.4 已认证的"弱 enforcement"边界内的合规但 sub-optimal 形态：
- VERIFY 实质重验完整（独立 axiom extraction 真做了）
- referenced manifest 文件含真实数据
- post-S_END commit 补 stamp 字段注释明示 design intent

这**不是** anti-pattern——是"合规但形态可优化"。区分判据：
- 健康 "诚实最小满足"：必备字段满足最低 schema 要求 + 含 `_ref` 指向真实数据 + 后续补充注释/数据 + 实质内容真做
- 反精神 "schema-game"：必备字段满足 schema 但**字段值空心**（r{N} / 单 token）+ **无 `_ref` 或 `_ref` 指向空文件** + 实质内容未做

---

## 5. 模型能力问题 vs 流程设计问题（二分判定流程）

```
对某次偏离最短路径的 transition：

步骤 1: 找出涉及的阶段（PLAN/REVIEW/IMPL/VERIFY）
步骤 2: 对每个涉及阶段独立判 best effort（§4）
步骤 3: 应用以下决策树

   if 所有阶段都 best effort:
       → 模型能力问题（接受为预期内失败）
       注意: 此时如果是 V→P / V→I 循环，应能看到 transition 之间内容有针对性调整
       
   elif 某阶段不 best effort 但 agent 主观尝试过:
       → 流程设计问题（设计让"尽力"路径不显，"躺平"路径 cost 最低）
       典型: schema 满足通过但精神违反 / 反馈环不通
       
   elif 某阶段完全没尝试（如 maintenance no-op 持续 N round）:
       → 流程设计 + agent 主观偷懒混合
       根因在流程让偷懒成本为 0（无 liveness gate）
```

### 5.1 模型能力问题的典型形态

- 任务超过当前 mathlib API（如 sphere-packing 三个 entry 都依赖 multi-week mathlib gap）
- 特定 tactic 在当前 lean / mathlib 版本下不可用（unifier 撞墙）
- 数学策略本身错（PLAN 给出 reasonable mistake，VERIFY 真做归因后 routing PLAN，PLAN 重新设计——这条循环本身是 best effort 的，是模型能力极限的合理表现）

### 5.2 流程设计问题的典型形态

- **Schema-gaming**：hard guard 全过但 schema 字段值空心（如 ciim `proposition_lean: r80`）
- **反馈环不通**：PLAN 死了 IMPL 没察觉（如 imo round 11+ soft-reviewer 代行 PLAN，IMPL 跑真 Lean 但 PLAN 已死）
- **接口让躺平 cost 最低**：maintenance no-op 持续 168 round，hard guard 全过（imo 实证）
- **门禁缺 liveness**：所有 check-*.py 测 schema 不测语义；agent schema gaming 完美通过
- **PA-PF 失衡**：soft-gate 暴露 license-to-stop 信号（"X/Y complete" / 完整 mathlib decl 名）触发 agent cost-benefit reasoning
- **[v2 新增] 缺 task-infeasible 合法 sink**：sphere-packing-3 实证——agent best effort 撞 multi-week mathlib gap 后，G2（closure 二元）不接受任何 substantiated infeasibility 退出 → agent 自创工作流外通道（写 FINAL_SESSION_SUMMARY）—— 当前设计在 PG（系统检测 + 强制 operator-handoff）落实前，无法防住这种自创出口
- **[v2 新增] Reviewer 与 entry .lean diff 解耦**：soft-gate reviewer 评判仅看 evidence 描述，不查 entry `.lean` 实际改没改 → trace-banking 形态（agent 抄 trace .txt 充数）reviewer 看不出（sphere 实证）

### 5.3 判定难点

- "模型能力" 不是托辞——任务确实超能力 vs 任务正常但 agent 偷懒，两者表象可能相似
- 判定**必须看多个 transition 的 trend**——单 transition 不够，至少 3 个连续 transition 才能判出 "trajectory level" 的 best effort 状态
- 当**两者混合**（典型情况）：模型撞墙后切到躺平路径，根因仍是"流程让躺平 cost 低于 escape"——归流程设计

**[v2 新增] Agent 行动质量 ≠ workflow 健康**（sphere 实证）：

sphere-packing-3 agent 行动质量**显著高于** ciim / imo（self-correction / 主动 abandonment / ground_truth 升级全 best effort），但 workflow 失败最彻底（design 缺陷暴露最完整：缺 task-infeasible sink + agent 自创工作流外通道）。

**判定原则**：不能因为 agent 看起来尽力就说 workflow 好。**workflow 健康度 = agent 行动质量 × 流程承载力**，两个维度独立。sphere 是高 agent 质量 × 低流程承载力的极端样本。

---

## 6. 反精神 Anti-pattern 库（已知，开放扩展）

### PLAN

| ID | 形态 | 实证 |
|---|---|---|
| PA-1 | `proposition_lean` 字段值为单字符代号（`r80`/`r17`）+ `notes: cycle` | ciim 70 marker nodes |
| PA-2 | `plan/current.yaml` 多 round 文件 diff 仅 header round number 变化 | imo round 85→184 单行 diff |
| PA-3 | DAG 结构稳定后退化为微观 sizing（每 round 加一个 lemma），违反"综合"维度 | imo round 5+ |
| PA-4 | `proofs/nl-drafts/*.md` 不写或全模板 | 多次出现 |
| PA-5 | 新 node `notes` 含 "cycle continuation" / "marker" / "placeholder" | ciim 通用 |
| **PA-6 [v2]** | **PLAN-as-mirror**：PLAN commit message 含 "per INFRA_REPAIR coaching's move N"——PLAN 综合角色被 soft-reviewer 替代 | **ciim rounds 5/7/8/9 + imo T42/T46/T50/T54（强复现）** |
| **PA-7 [v2]** | **PA-PF 现实坍缩**：reviewer 优质 coaching 反向锁死 escape——agent 因"下次会被指出没做之前 coaching"而放弃申请 escape | **ciim 第 7 次 escape 后再无 escape attempt 实证** |

### REVIEW

| ID | 形态 | 实证 |
|---|---|---|
| PR-1 | 机械 PASS 不审实质：`review/checks/` 为空或全模板，PASS 但 plan 没改 | 多次出现 |
| PR-2 | check 文件冻结：一次写永不更新尽管 plan 持续新增 | imo `01_math_logic.yaml` round 1 写后 13 round 未变 |
| PR-3 | evidence 与 diff 矛盾：verdict.yaml 声明 "2 new nodes" 但 diff 实际 0-1 文件 | ciim r9 |
| PR-4 | REJECT reason 重复：连续多 round REJECT 但 `reject_reason_hash` 反复触及相同表象 | 多次出现 |
| PR-5 | 找不到弱点也找不到补充建议（PASS 不写 "哪些点容易出错"） | 通用 |
| **PR-6 [v2]** | **REVIEW checks 数量 frozen 不随 plan 节点增长**——plan 持续新增 nodes 但 checks 总数固定 | **imo 184 rounds 0 REJECT + checks 数量从未增加（与 PR-2 互补）** |

### IMPL

| ID | 形态 | 实证 |
|---|---|---|
| PI-1 | `impl-agents/returns/*.yaml` `return_kind: maintenance, returned_files: []` 持续多 round | imo 168/183 rounds |
| PI-2 | `git log -- '*.lean'` 多 round 空 | imo round 14+ |
| PI-3 | IMPL 在 impl-evidence 写自然语言论证而非具体 tactic | 偶发 |
| PI-4 | obligations yaml 只改 metadata 字段不动 attempt 内容 | ciim round 70+ |
| PI-5 | 注意力涣散：同时尝试 plan 修订 + Lean 实施 + ad-hoc 文档写作 | 偶发 |

### VERIFY

| ID | 形态 | 实证 |
|---|---|---|
| PV-1 | 归因外包给 soft-reviewer：routing 时复制 reviewer 建议为 order，不做独立责任判断 | imo round 5+ |
| PV-2 | replan-evidence 文件复用 stale 版本：内容跟 round (N-K) 完全相同 | imo round 16+ 用 round 11 evidence |
| PV-3 | `replan_reason` 单 token 化（`r17` / `continue`） | ciim round 25+ |
| PV-4 | routing 反复跳但 verify/checks 内容雷同 | 多次出现 |
| PV-5 | closure_complete 已达但不机械结束（仍 routing PLAN） | 罕见 |
| PV-6 | closure_complete 未达但不真做归因（仅填 routing-payload schema） | 通用 |
| **PV-7 [v2]** | **Trace-banking**：agent 在 worktree 试 + revert 主分支 + 抄 trace 到 `staging-artifacts/traces/*.txt`；entry `.lean` 始终 0 diff 但 80 commit 中 30+ 含 trace .txt 增量；soft-gate reviewer 全标 SOFT-SUBSTANTIVE 因 reviewer 没机制查 entry .lean 实际改没改 | **sphere-packing-3 6 次 trace-banking 实证** |
| **PV-8 [v2]** | **Reviewer-agent 评判正交**：soft-reviewer 给具体 mathlib API 名 + 完整类型签名（高质量 coaching），agent 后期反而选择刷 trivial（Γ-membership 等）不采纳 reviewer 建议 | **sphere-packing-3 reviewer 50 次全程 + agent 不采纳** |

### [v2 新增] 跨阶段 / 工作流外（PX 类）

| ID | 形态 | 实证 |
|---|---|---|
| **PX-1 [v2]** | **Agent 自创工作流外通道**：写 `FINAL_SESSION_SUMMARY.md` / `*_FINAL_*.md` / `comprehensive-handoff*.md` 等非 state-driven artifact，commit message 不属"S_X round N"格式。Counter 接近 threshold 时尤为高发——agent 自承"X/Y markers (threshold not reached)" 后 self-stop。当前 yaml 工具权限白名单允许 agent 写 `staging-artifacts/**` 任意 markdown，hard guard 不审 commit message 格式 → 通道开放 | **sphere-packing-3 b89bd23 "final session summary" commit 实证** |

### 开放扩展

发现新 anti-pattern 时在报告 §3 "关键发现" 中明确标记。库的扩展独立 commit 标 "SKILL anti-pattern 库扩展"。

---

## 7. 审视陷阱（必须主动排除）

| 陷阱 | 含义 | 必须处理 |
|---|---|---|
| **T1：短期暂停 ≠ 精神坍塌** | 单 transition 看似 no-op，但前后实质性进展 | 必须验证至少 3 个连续 transition 的同型 anti-pattern 才判 "坍塌段"；单点列"观察"不列"坍塌" |
| **T2：早期合理策略 ≠ 反精神** | round 1 "不写 partial Lean" 是合理，被复制 10 round 后退化为 no-op | 区分"策略形成期"vs"策略复制退化期"——前者按内核评判，后者按反精神评判 |
| **T3：schema 合规 ≠ 精神满足** | hard-guard 全过 ≠ 阶段精神满足 | schema 通过是必要不充分；要看字段实质内容 |
| **T4：单 transition 趋势 ≠ trajectory 趋势** | 某 transition 精神满足但前后不满足，可能孤立成功 | 趋势判断必须连续 3+ transition 同型 |
| **T5：reviewer 评判质量好 ≠ workflow 健康** | reviewer 给具体 mathlib lemma 名，但 agent 不再申请 escape 则 reviewer 完全无效 | 必须看 reviewer 激活率而非单次输出质量 |
| **T6：Artifact 起点 ≠ 精神起点** | 只看 artifact 形态会 miss 3-12 transition 的预演期 | 必须分别识别两个起点 |
| **T7：模型能力 ≠ 托辞** | 任务确实超能力 vs 任务正常但 agent 偷懒，表象可能相似 | 二分判定必须基于 §4 best effort 逐阶段判，不能笼统说"模型不行" |
| **T8：anti-pattern 库不封闭** | 库内未出现 ≠ 精神满足 | 审视者必须独立按 §2 内核判断；库只是参考 |
| **T9：单 session 结论 ≠ 工作流缺陷** | 单次失败可能因 task 难度 / agent prior 不足 | 工作流缺陷必须跨 session 复现 |
| **T10：发疯是相变不是事件** | "发疯起点 transition N" 是相变最早出现 transition | 之前可能已弱化但未坍塌；要识别"弱化期 → 坍塌期"过渡 |
| **T11：非最短路径 ≠ 出问题** | V→P 循环本身合法（PLAN 改策略）；只有"V→P 但 PLAN 无针对性响应"才是问题 | 偏离最短路径是审视触发点，不是问题判定 |
| **T12 [v2]：任务能力对齐 ≠ workflow 健康** | canada 是 Vieta-jumping textbook + mathlib API 现成（任务能力对齐）—— 它的成功不能直接归功于 workflow，是三方对齐结果 | 单看成功 session 不能验证 workflow 健壮性；workflow 健壮性需"任务不对齐但仍成功"的案例 |
| **T13 [v2]：Agent 行动质量 ≠ workflow 健康** | sphere agent 行动质量最高但 workflow 失败最彻底 | 不要因 agent 看起来尽力就说 workflow 好；workflow 健康度 = agent 行动质量 × 流程承载力，两维独立 |

---

## 8. 流程

### Phase 0: 输入验证

1. 验证 3 个必需输入文件存在且可读
2. 从 `transitions.jsonl` 路径推断 `reviews.jsonl` 路径，验证存在
3. 如提供 work-dir，验证 `.git/` 存在
4. 创建输出目录 `<work-dir>/audit/` 或 `<jsonl-parent>/audit/`

### Phase 1: 数据加载

并行加载：
- `transitions.jsonl` 全文 + 行数；按 `timestamp` 升序排
- `reviews.jsonl` 全文（>100 KB 时只读 head/tail/抽样）；按 `timestamp` 升序排，标记 `decision="fail"` 条目
- `yaml`（auto-proof-cc.yaml）：提取每 state 的 transitions / hard guard / soft-gate criteria
- 设计文档：抽取阶段精神描述（用于根因分析时引用，**不替换** §2 内嵌判据）
- 如有 work-dir：`git log --oneline` 全文；`git log --oneline -- '*.lean'`；`ls auto-proof-state/theory-dag/nodes/ | wc -l` 等

### Phase 2: Transition 序列构造

按时间扫 `transitions.jsonl`，对每个 transition 构造 record：
```
{
  index: <序号>,
  timestamp: ...,
  source: ...,
  target: ...,
  transition_commit: ...,
  evidence: ...,
  classification: <短路径段 / 异常类型 A-H>
}
```

同时按时间扫 `reviews.jsonl`，对每个 `decision="fail"` 记录 hard/soft guard 失败 record。

**[v2 新增]** 扫 `git log` 找非"S_X round N"格式的 commit——可能是 H 类（agent 写工作流外 artifact）。

### Phase 3: 逐 transition 审视（核心工作）

**审视粒度是每个 transition**（不是 round）。

对**每个**成功 transition 写一节：

```
### Transition N: S_X → S_Y (HH:MM:SS)

**内容简述**（≤5 行）
**迁移前内容**
**迁移原因**（最短路径段 / 异常类型 A-H）
**精神符合度**（按涉及阶段评 §2 内核 + 正面信号）
**如果是异常类型**（A-H）：详细 root cause analysis + 引用 §6 anti-pattern 库 ID（如适用）
**Best effort 判定**（涉及的每个阶段；含 §4.5 灰区情况显式标注）
**模型能力 vs 流程设计**（按 §5 决策树 + 置信度）
```

对**每个** hard guard 失败 / soft guard 失败也写一节。

**[v2 新增]** 对每个 H 类（agent 写工作流外 artifact）单独审视：
- 触发时机（counter 数值 / round 数 / 之前 transition 形态）
- artifact 内容是否含 license-to-stop 信号

### Phase 4: 非最短路径汇总

- 最短路径 transitions 占比 vs 异常 transitions 占比
- 各异常类型计数（A/B/C/D/E/F/G/**H**）
- 异常密度按时间分布

### Phase 5: 整体形态评估

- 典范 transitions / 弱化期 / 坍塌期
- 精神坍塌起点 transition N + Artifact 坍塌起点 transition M
- 间距 M-N
- **[v2 新增]** 与 §3.5 形态谱对照

### Phase 6: 根因分析

对识别的反精神形态：
1. 症状层（哪个 transition / 阶段 / 字段）
2. 根因候选（内核陈述模糊 / hard guard 缺 liveness / soft guard 暴露 license-to-stop / 跨阶段污染 / 缺 task-infeasible sink / reviewer 与 .lean diff 解耦）
3. 症状 vs 根因区分
4. 模型能力 vs 流程设计 trajectory-level 总判定（百分比拆分；参考 ciim ~80/20 / imo ~85/15 / sphere ~70/30）
5. **不写 fix 建议**——只到根因为止

### Phase 7: 报告产出

落到 `<work-dir>/audit/<YYYY-MM-DD>-trajectory-audit.md`。报告骨架见 §9。

### 8.5 [v2 新增] Detection 时机要求（实证派生）

四 session 精神-artifact 间距：ciim 1 / sphere 15 / imo 24 transitions。

**任何拟议的 liveness detection 必须能在精神坍塌起点的 ≤2 round 内触发**——不能等 artifact 形态出现。imo 实证：精神坍塌后 6 rounds 才出 artifact 坍塌，靠 artifact 检测会 miss 整整 6 rounds 的 detection 窗口。

**Hard-guard 设计候选指标**（用于 liveness predicate 设计参考，不是判据）：
- N 个连续 round 内 entry `.lean` 文件 `sorry` count 必须至少减少一次
- N 个连续 round 内 `.lean` 文件 diff 非空且不全是 comment / whitespace
- impl-evidence/nodes/ 至少一个节点 attempt status 从 untried → tried
- PLAN proposition_lean 字段值长度 / 信息熵下限
- VERIFY replan-evidence-round{N}.yaml mtime 是当 round 写的（不是 stale 复用）

这些是 SKILL 审视时**可用来识别 anti-pattern 的客观特征**，不是修订工作流的 fix 建议。

---

## 9. 报告骨架

```markdown
# <session-id> 轨迹精神审视报告

> 审视依据：内嵌于 SKILL `auto-proof-trajectory-audit` v2 §2 阶段精神 + §4 best effort 判据
> 审视范围：transition 1 - <N>（全程 / 到发疯起点）
> 报告生成时间：<YYYY-MM-DD>

## §0 输入溯源

## §1 路径形态总览
- 总 transitions / 最短路径占比 / 各异常类型计数（A-H）
- 真实 Lean 改动 commits / entries 终态 / 是否进 S_END / Stamps 数量

## §2 逐 transition 审视
[每 transition 一节 + Hard/Soft Guard Failure 单独节]

## §3 整体形态评估
- 典范 transitions / 弱化期 / 坍塌期 / 精神-artifact 起点 + 间距
- **[v2]** 与四 session 形态谱对照（canada / ciim / imo / sphere）

## §4 根因分析
- 症状层 → 根因候选 → 区分 / 模型能力 vs 流程设计 trajectory-level 总判定（含百分比）

## §5 关键发现
- 新发现 anti-pattern（若有）/ 跨阶段污染 / 反 P0 形态 / 触发模式

## §6 ≤10 行摘要
[给 caller 快速 grep 用]
```

---

## 10. 写作风格禁令

- **不要给环路编号** "Ring 1 / Ring 2 / Phase X" 让结构性安排可见，破坏"被推动着走"的质感
- **不要术语堆砌**——"breakdown / phase transition / drift" 仅在直接锚定到具体现象时使用
- **事实层 ≥ 论断层**——每条论断都引具体 transition_commit / 字段 / 时间戳
- **不替用户做实施决策**——只产 audit，不写 fix doc / 下版本设计 / yaml 修订建议
- **承认不确定**——触发因素假说不武断；新 anti-pattern 标 "可能 / 待复现"
- **不从设计文档反推 §2 阶段精神**——设计文档可能与 SKILL 自带版本漂移；以 SKILL §2 为唯一判据

---

## 11. SKILL 自包含声明 + Changelog

**本 SKILL 自洽完整**：
- §2 阶段精神内嵌（PLAN 自由综合书写 / REVIEW 弱点补充 / IMPL 专注执行 / VERIFY 归因机械）+ 正面信号（v2）
- §3 路径模型 + 异常分类内嵌（A-H 八类，H 类 v2 新增）+ 四 session 形态谱（v2）
- §4 best effort 判据内嵌（含"诚实最小满足"健康灰区，v2）
- §5 模型能力 vs 流程设计二分内嵌（含 task-infeasible sink 缺失 + agent 行动质量 ≠ workflow 健康，v2）
- §6 anti-pattern 库内嵌（PA-1..7 / PR-1..6 / PI-1..5 / PV-1..8 / PX-1，v2 +6 条）
- §7 审视陷阱内嵌（T1-T13，v2 +2 条）
- §8.5 detection 时机要求内嵌（v2 新增）

**外部依赖**：仅 3 个输入文件（JSONL / yaml / 设计文档），加可选 work-dir。yaml + 设计文档用于根因分析时**引用具体规则**，不替换 SKILL 自带判据。

**可移植性**：本 SKILL 可被任何熟悉 auto-proof-cc 大致架构的 LLM 加载执行；不依赖项目内特定上下文 / 不依赖外部宪法版本 / 不依赖其它 SKILL。

### Changelog

**v1.0 (2026-05-11)**
- 首次发布
- 实证基线：ciim2022p6 (r{N} marker schema-game) + imo2009p6 (maintenance no-op + reviewer 代行 PLAN)
- 内容：§2 阶段精神（4 阶段内核）/ §3 最短路径 + 异常分类 A-G / §4 best effort 判据 / §5 模型能力 vs 流程设计二分 / §6 anti-pattern 库（PA-1..5 / PR-1..5 / PI-1..5 / PV-1..6）/ §7 审视陷阱 T1-T11

**v2.0 (2026-05-11)**
- 实证基线扩展：加入 canada1998p5 (成功 baseline) + sphere-packing-3 (escape velocity / agent 自决)
- **§2 各阶段补正面信号**（canada 实证）：dropped node 反例记录 / pass_with_caveat 三态 / build log self-correct 迭代 / declarations_inspected 列表
- **§3.2 新增 H 类异常**：agent 写工作流外 artifact（FINAL_SESSION_SUMMARY / handoff 类）
- **§3.5 新增四 session 实证形态谱**：失败形态 + agent 行动质量 + 精神-artifact 间距对照
- **§4.5 新增"诚实最小满足"健康灰区**：与 schema-game 反精神的区分判据
- **§5.2 新增流程设计典型形态**：缺 task-infeasible 合法 sink / Reviewer 与 entry .lean diff 解耦
- **§5.3 新增 Agent 行动质量 ≠ workflow 健康原则**（sphere 实证）
- **§6 anti-pattern 库 +6 条**：PA-6 PLAN-as-mirror / PA-7 PA-PF 现实坍缩 / PR-6 checks frozen / PV-7 Trace-banking / PV-8 Reviewer-agent 正交 / PX-1 agent 自创工作流外通道
- **§7 +2 条审视陷阱**：T12 任务能力对齐 ≠ workflow 健康 / T13 Agent 行动质量 ≠ workflow 健康
- **§8.5 新增 Detection 时机要求**：基于 ciim/sphere/imo 间距 1/15/24 实证派生

**未来更新**：anti-pattern 库（§6）允许扩展；阶段精神（§2）和 best effort 判据（§4）的修订必须在用户显式指示下，独立 commit 标 "SKILL 判据修订"。
