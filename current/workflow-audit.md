---
name: workflow-audit
description: Multi-direction, disprove-first audit of GitHub PRs with parallel challenge rounds and evidence-gated findings
---

# Workflow Audit

对 GitHub PR 进行多方向并行审计，采用**驳回优先**方法论，所有发现必须经受独立质疑才能成为结论。

本 SKILL 是只读审查流程——**禁止修改代码**，只产出 review 文档。

## 使用方式

```
/workflow-audit PR #<number>
/workflow-audit PR #<number> 重点关注 <area>
```

## 输入

- GitHub PR 编号（或 URL）
- 可选：重点关注区域（如"修复和驳回的部分"）
- 可选：项目的 workflow.md 路径（默认检查 `docs/workflow.md`）

## 输出

- 审计产出目录：`docs/audit/pr<N>-review/`（中间产物）
- 最终 review 文档：`.local/<date>-pr-<N>-review.md`

---

## 反状态清单（必须主动排除）

任何发现进入最终结论前，主审必须逐条寻找并记录以下“反状态”。命中任一反状态时，不得按原严重性输出；必须降级、改写为争议项，或直接驳回。

| 反状态 | 含义 | 必须处理 |
|---|---|---|
| **R1：现象存在 ≠ 缺陷成立** | 代码、测试、日志能证明现象发生，但不能证明它违反需求、设计或用户契约 | 找到被违反的明确 contract；找不到则降级为观察/争议 |
| **R2：已文档化 contract** | 代码注释、设计文档、测试名称/注释明确把该行为定义为当前 contract | 不得直接列为 bug；只能提出“是否改变 contract”的决策项 |
| **R3：测试夹具可达 ≠ 生产可达** | 复现依赖 test-only 构造、直接注入内部队列、mock、不经过公开 API | 标记为 synthetic；代码类严重性最高只能到 Low/Nit，除非补充生产路径复现 |
| **R4：存在 fail-fast / guard / invariant** | 正常入口有预校验、权限门、schema 校验、生命周期约束，阻止问题状态出现 | 必须证明 guard 可被生产路径绕过，否则 REJECTED — not reachable |
| **R5：历史/废弃文本 ≠ 当前文档错误** | 命中的文本位于历史章节、废弃章节、migration note、已标 `[已废弃]` 区域 | 只能要求加更醒目的废弃标记；不得声称当前契约错误 |
| **R6：局部 stale ≠ 全局缺文档** | 某处文本过期，但 README、architecture、devlog、fix doc 或 changelog 已有正确说明 | 降级为局部 cleanup；不得说“未文档化” |
| **R7：弱化发现不得恢复强措辞** | 质疑轮给出 WEAKENED、DISPUTED、INCONCLUSIVE 后，最终文本又用 High/Blocking/必须修 | 禁止；必须保留弱化后的范围和严重性 |
| **R8：流程建议 ≠ PR 阻塞项** | PR scope、devlog、checklist、commit hygiene 等流程改进，未影响当前代码/用户契约 | 默认 Non-blocking；除非项目规则明确要求 blocking |
| **R9：可选文档记录 ≠ 必须修复缺陷** | fix doc/devlog/CLAUDE lesson 缺失，但 commit message、PR body、测试或文档已有 durable record | 降级为 process nit；不得和 runtime/doc contract bug 捆绑 |
| **R10：并发/竞态主张必须给 interleaving** | 声称 race/atomicity bug，但没有具体 A/B 步骤、共享状态、锁边界、可观测后果 | 不得作为代码发现；最多列为待验证风险 |
| **R11：API 文案问题 ≠ 实现问题** | 实现和测试正确，只是 README/tool description/design text 不一致 | 分类必须是文档；不得要求代码 fix 或 runtime regression |
| **R12：作者可反驳点必须预先写出** | 如果 maintainer 可用“这是 contract / 不可达 / 已废弃 / 已在别处说明”一句话反驳 | 主审必须在最终文档中先给出反方和自己的处理；不能留给对方首轮指出 |

---

## 流程概览

```
Phase 1: 数据收集
Phase 2: 三方向并行审计
Phase 3: 发现汇总去重
Phase 4: 质疑轮（驳回优先，≥2 人/发现，fresh context）
Phase 5: 判定聚合 + 分裂判定仲裁
Phase 6: 最新代码复查
Phase 7: 反质疑（存活发现二次检验）
Phase 8: 主审亲自验证
Phase 9: 综合两方证据，生成 review 文档
```

---

## Phase 1 — 数据收集

收集 PR 的全部上下文（方式不限——`gh` CLI、GitHub API、`fetch_content` 抓 PR 页面、手动提供 diff 均可）：

- 所有 commit（按阶段分组：原始提交 vs 修复提交）
- 完整 diff
- PR 描述中声称修复/解决的内容
- 已有的 review comments 和讨论

同时读取项目的参考材料：
- 项目 workflow / 开发规范文档（如存在）
- 项目的 CLAUDE.md / AGENTS.md / CONTRIBUTING.md 中的规范和已知限制

## Phase 2 — 三方向并行审计

同时启动三个独立的审计方向，每个方向使用不同的分析框架。三个方向**并行执行**（`subagent parallel`），互不知道彼此的存在。

### 方向 A：Hoare Logic + Workflow

- 对 PR 中每个函数/模块变更构建 Pre/Post/Invariant
- 验证修复的 postcondition 是否完备
- 检查 workflow.md §5.1 修复方法论合规性
- 重点：逻辑正确性、安全属性、不变式保持
- 参考：`/hoare-prompt`、`/hoare-audit` methodology

### 方向 B：Finegrained Check + Workflow

- 从 PR 变更中提取命题（proposition extraction）
- 跨文件矛盾分析（Cross-file contradiction analysis）
- Workflow 文档优先（doc-first）合规检查
- 重点：文档一致性、API 契约、描述与实现的偏差
- 参考：`/finegrained-check`

### 方向 C：Workflow Pure

- 纯流程审计：对照 workflow.md §1-§7
- 设计阶段、实现顺序、测试分层、修复方法论、文档更新
- 检查 PR 生命周期是否符合项目开发规范
- 重点：是否有 `docs/fixes/` 文档、devlog 是否同步、CLAUDE.md 教训是否回写

### 执行规范

- 每个方向各派一个 agent（推荐使用指定的强模型）
- 使用 `fresh` context（不继承父会话状态，避免确认偏误）
- 每个 agent 收到完整的 PR diff + 相关源文件 + 对应的参考材料
- 产出写入 `docs/audit/pr<N>-review/direction-{a,b,c}.md`

## Phase 3 — 发现汇总去重

收集三个方向的全部发现，执行：

1. **去重**：相同问题被多个方向发现的，合并为一条，标注发现来源
2. **分类**：每条发现标注严重性（Critical / High / Medium / Low）和类型（代码 / 文档 / 流程）
3. **编号**：统一编号 F1, F2, ... 或 W1, W2, ...（W = workflow 流程类）
4. **反状态预筛**：对每条发现套用“反状态清单”R1-R12，记录可能命中的反证方向
5. **只汇总不删除**：任何方向产出的发现都保留，不在此阶段做判断性筛除；但必须带上反状态标签，例如 `possible R2/R3`

## Phase 4 — 质疑轮（核心：驳回优先）

> **原则**：默认立场是怀疑。每条发现必须经受住主动的推翻尝试才能成为可执行项。

对 Phase 3 的每条发现，派出 **≥2 个独立 agent** 进行质疑。

### 质疑者规范

- 使用 `fresh` context（**绝对不能用 fork**，避免继承审计偏见）
- 质疑者**不看原始审计报告**，只看具体的单条 claim
- 每个质疑者收到：
  - 具体主张（文件、函数、描述、严重程度）
  - 相关源文件（让质疑者自己读代码）
  - 显式指令：**"你的任务是找出这条发现错误的理由。寻找反驳它的证据。保持对抗性。必须逐项检查反状态 R1-R12：是否只是现象存在、是否已文档化为 contract、是否 test-only 可达、是否被 guard/invariant 阻止、是否只是历史/废弃文本、是否被其他文档缓解。"**

### 判定选项

- **UPHELD**：尝试推翻但未能成功，说明尝试了哪些角度；不得只证明“现象存在”，还必须证明“违反当前 contract 且生产可达”
- **WEAKENED**：部分正确但被夸大，附证据
- **REJECTED — misreading**：原审计者误读了代码
- **REJECTED — mitigated**：问题被其他机制缓解
- **REJECTED — not reachable**：不变式阻止了该状态
- **REJECTED — context mismatch**：威胁模型不匹配
- **INCONCLUSIVE**：无法决定

### 分组策略

- 将发现分成多组，每组 3-5 个发现
- 每组派一个 agent 负责质疑
- 确保每条发现至少被 2 个不同组的 agent 覆盖
- 最多 6 个质疑 agent 并行
- 产出写入 `docs/audit/pr<N>-review/challenge-{1..6}.md`

## Phase 5 — 判定聚合

对每条发现，汇总所有质疑者的判定：

| 情况 | 处理 |
|---|---|
| 全部 UPHELD | → 执行反状态复核；若无 R1-R12 命中才进入 Phase 7 |
| 全部 REJECTED | → 丢弃，记录到已拒绝列表 |
| 判定分裂（有分歧） | → 派仲裁 agent 做 tiebreak，然后进 Phase 7，标记"争议" |
| 全部 WEAKENED | → 调整严重程度/范围后进 Phase 7；最终文本不得恢复原强度 |

### 分裂判定仲裁

对判定分裂的发现，派一个额外 agent（`fresh` context）做仲裁：
- 收到双方的判定和证据
- 要求给出最终判定 + 理由
- 产出写入 `docs/audit/pr<N>-review/tiebreak-<finding-id>.md`

## Phase 6 — 最新代码复查

在质疑轮结束后，检查目标仓库是否有新的 commit（PR 可能在审计期间更新）。

对每条存活发现，在最新代码上重新验证：
- **已解决**：新 commit 修复了该问题 → 标记为 resolved
- **仍存在**：问题在最新代码中仍然存在 → 继续
- **部分解决**：部分修复 → 更新发现描述

## Phase 7 — 反质疑

对每条通过 Phase 5 且在 Phase 6 中仍存在的发现，再启动**一个 fresh agent** 做最终推翻尝试。

反质疑者收到：
- 原始 claim
- 质疑者的判定摘要（尝试了哪些推翻角度）
- 该 claim 已命中或可能命中的反状态标签（R1-R12）
- 指令："之前的质疑者试图推翻此发现但未成功。做最后一次尝试。寻找他们遗漏的角度。尤其检查：现象是否只是被测试锁定的 contract；复现是否绕过公开 API；文档命中是否位于废弃/历史章节；是否已有其他文档缓解。"

- 反质疑者也无法推翻 → **确认成立**
- 反质疑者找到有效推翻 → 标记为**争议项**

产出写入 `docs/audit/pr<N>-review/counter-challenge-{1..N}.md`

## Phase 8 — 主审亲自验证

主审（你自己）**亲自阅读源码**验证每条确认发现的关键证据：

- 对每条发现，用 `read` 工具读取实际文件和行号
- 声明："我在 `file:line` 验证了 [具体证据]" 或 "我无法验证 [主张]"
- **不能仅依赖 agent 报告**

### 反状态复核（主审必做）

主审亲验时必须为每条存活发现填写：

```markdown
#### 反状态复核
- R1 现象存在≠缺陷成立：<是否命中，证据>
- R2 已文档化 contract：<是否命中，证据>
- R3 测试夹具可达≠生产可达：<是否命中，公开 API/生产路径复现>
- R4 guard/invariant 阻止：<是否命中，入口校验/不变式证据>
- R5 历史/废弃文本：<是否命中，章节上下文>
- R6 其他文档缓解：<是否命中，交叉引用>
- R7 弱化后是否恢复强措辞：<检查结果>
- R8/R9 流程建议是否误列 blocking：<检查结果>
- R10 race 是否有 interleaving：<步骤>
- R11 文档问题是否误列代码问题：<检查结果>
- R12 maintainer 一句话反驳：<最强反驳 + 主审回应>
```

任何代码类发现若无法给出**公开 API 或真实运行路径**复现，只能进入 Disputed/Non-blocking，不得进入 Blocking。

### 证据标准

**所有发现**必须先通过反状态复核：
- 不能把“行为存在”当成“缺陷成立”。
- 不能把“已文档化 contract”当成“代码 bug”。
- 不能把“test-only 夹具可达”当成“生产可达”。
- 不能把“局部 stale”当成“全局缺文档”。

**代码类发现**必须具备：
- 具体的测试用例、PoC 脚本或逐步复现步骤
- "理论上可能发生"不够——要展示它发生

**文档类发现**必须具备：
- 确切的错误文本引用（file:line）
- 三重确认文档中没有其他部分缓解该问题（changelog、免责声明、交叉引用）
- 如果存在缓解措施，降级为 nit，除非缓解对文档主要使用模式明显不足

## Phase 9 — 生成 Review 文档

将所有存活发现编纂为结构化 review 文档。

### 文档结构

```markdown
# PR #<N> Review: <title>

审计方法：三方向并行审计 + 驳回优先质疑 + 反质疑 + 主审验证
审计基准：commit <hash>
审计日期：<date>

## 概要

<一段话总结 PR 的目标、审计结论、需要关注的重点>

## 阻塞性问题（Blocking）

### F<N>: <title> [严重性]

**问题**：<描述>
**证据**：<file:line 引用 + 具体内容>
**测试用例 / 复现步骤**：<代码类必须有；必须说明是否经过公开 API/真实运行路径，不能只用 test-only 注入>
**反状态复核**：<列出 R1-R12 的命中情况；命中项如何降级/改写>
**审计路径**：<哪个方向发现 → 质疑结果 → 反质疑结果 → 主审验证>
**建议修复**：<具体建议>

## 非阻塞性建议（Non-blocking）

### F<N>: <title> [严重性]
<同上格式，但明确标注为建议而非要求>

## 争议项（Disputed）

### F<N>: <title>

**正方（问题存在）**：<论点 + 证据>
**反方（问题不存在/已缓解）**：<论点 + 证据>
**主审评估**：<哪方更有理，但不做最终裁决>

## 已解决（Resolved）

<被新 commit 修复或被质疑者成功驳回的发现列表>

## 审计方法说明

<三方向各审了什么、质疑轮的统计数据>
```

### 关键原则

- **争议项如实呈现双方**：不以多数票解决，呈现论点让 PR 作者/maintainer 判断
- **不删除任何发现**：被驳回的发现记录在"已解决"section，保持审计完整性
- **措辞精确**：每个判断性措辞都要有证据支撑，不做主观评价
- **PR-ready 收敛**：如果用户要求把审计结论整理成 issue/PR statement，只能保留维护者无法用 R1-R12 快速驳回的可执行项；争议项、弱发现、流程偏好不得伪装成必修项。

---

## 硬性约束

1. **只读**：整个流程中**禁止修改任何源代码**。只产出 review 文档。
2. **fresh context**：所有质疑/反质疑 agent 必须使用 `fresh` context，不能用 `fork`。避免继承审计偏见。
3. **≥2 独立质疑**：每条发现至少被 2 个独立 agent 质疑。没有经过质疑的发现不能进入最终 review。
4. **证据门控**：代码类发现必须有测试用例/PoC；文档类发现必须三重确认无缓解。缺少证据的发现降级为 nit。
5. **汇总不删除**：Phase 3 汇总时只能合并重复项，不能基于主观判断删除发现。
6. **争议交由人类**：质疑者和反质疑者意见分裂的，如实呈现双方证据，不做裁决。
7. **主审亲验**：Phase 8 中主审必须亲自读源码验证，不能仅依赖 agent 报告。
8. **反状态门控**：Phase 3/5/7/8 必须显式检查 R1-R12。任何命中反状态的发现不得按原强度进入最终结论。
9. **生产可达门控**：代码类 Blocking 必须通过公开 API 或真实运行路径复现；test-only 注入、mock、内部构造只能支持争议/低优先级发现。
10. **Contract 门控**：若行为被代码注释、设计文档或测试明确锁定为当前 contract，不得称为 bug；只能提出是否改变 contract 的决策项。

## 与其他 Prompt/Skill 的关系

本 SKILL 编排整体流程。各 Phase 中可引用以下参考材料（如项目中存在）：

| 参考 | 用途 | 用于 Phase |
|---|---|---|
| `/hoare-prompt` | Hoare Logic 分析框架 | Phase 2 方向 A |
| `/hoare-audit` | 审计方法论（驳回优先） | Phase 2 方向 A, Phase 4 判定体系 |
| `/finegrained-check` | 命题提取与矛盾分析 | Phase 2 方向 B |
| `/workflow` | 项目开发流程规范 | Phase 2 全部方向 |

不要求这些参考材料全部存在。如果缺少某个参考，对应方向简化为通用审计。

## 典型执行示例

```
用户：/workflow-audit PR #31 重点关注修复和驳回的部分

Phase 1：收集 PR 数据 → 12 commits, 14 files
Phase 2：派 3 个 agent 并行 → 产出 direction-{a,b,c}.md
Phase 3：汇总 23 条发现，去重后 18 条
Phase 4：派 6 组质疑者 → 4 条 REJECTED, 8 条 UPHELD, 6 条分裂
Phase 5：6 条分裂发现派仲裁 → 3 UPHELD, 2 REJECTED, 1 INCONCLUSIVE
Phase 6：git fetch → 3 条被新 commit 解决
Phase 7：8 条存活发现派反质疑 → 6 确认, 1 争议, 1 REJECTED
Phase 8：主审读源码验证 6+1 条
Phase 9：生成 .local/5-1-pr-31-review.md
```
