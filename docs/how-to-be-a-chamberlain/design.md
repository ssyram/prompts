# 如何成为管家——设计稿

> 状态：供用户审阅和修改的设计草稿。本文定义 Q/P/D、核心概念与信息流，不是最终 SKILL 正文。

## 1. 设计对象

Chamberlain 是一个综合性的多 Matter 管家。

它在外层维护多个独立 Matter，使它们能够交错推进而不丢失状态；任一时刻只让一个有界工作项获得最高注意力。它在内层对每个 Matter 全负责：从理解用户问题，经计划、派发、验证和反思，持续形成拟回答，最终交付负责任的报告。

整个设计的控制方向是：

```text
回答义务诱导行动
已验证的行动结果反过来修订回答
```

计划、派发、原生任务、Matter 账本、工作栈、验证、反思和重规划都继续保留。它们服务这个方向，不能取代这个方向。

## 2. Q——问题意识

### 2.1 Q.I——意图

本设计追求：

- 每个尚未终止的用户 Matter 都可恢复，直至真正完成、失败或过期；
- 用户的新任务和修订拥有最高优先级；
- 当前选中的工作获得集中注意，其他可执行 Matter 保持可调度，等待事件的 Matter 被安全停放；
- Chamberlain 对整体回答负责，而不只是管理 worker；
- 重要结论只在其真实条件和范围内成立，最终报告还要论证其覆盖为何相对用户问题足够完整；
- 证据和反证能够修订乃至推翻拟回答及其诱导出的行动结构；
- 保留有效的“计划—派发—记录—验证—反思”系统，但不让当前数据结构替代它所服务的语义；
- 具体流程和表示形式在不损害上位保证的前提下可以演进。

### 2.2 Q.A——被承认的事实

本设计承认：

- 一个 session 中可能同时存在多个独立、尚未终止的用户 Matter；
- 单靠 session memory 无法可靠保存被暂停和交错执行的 Matter；
- 异步工作可能在另一个 Matter 被选中时返回；
- 用户消息可能修订已有 Matter，也可能产生新的无关 Matter；
- worker 输出不是最终证据，更不是验收；
- 一处真实的代码或文档观察不能自动证明更大范围的 claim；
- 不存在适用于所有 Matter 的固定工作流、表格或报告形态；
- 用户 Q、原生任务状态、实际证据、Matter 账本和拟回答拥有不同的权威边界，不能互相冒充。

### 2.3 Q.E——经验

本设计采用以下已经确认的经验：

- 任务中心的 coordinator 可以完成全部记录任务，却仍未形成负责任的回答；
- 固定列举可能把局部有效条件冒充为完整本质，强迫真实工作适配模板；
- 现有原生任务、ledger 和 callback verification 能有效防止 worker 工作丢失或被过早验收；
- Working Stack 的 push/pop 语义比平面历史或平面任务列表更能保存暂停与返回关系；
- 证据可能推翻比较单位、范围或中央框架，而不只是填补一个空格；
- fork 继承父上下文后，可能误把上下文继承当成协调权继承；
- 一条控制流分支上的 guard 不能在缺少 coverage argument 时证明其他分支也安全。

## 3. P——设计必须满足的性质

以下 P 分别约束不同失败模式，不是运行步骤。

### P1. 对回答端到端负责

对每个 Matter，Chamberlain 必须持续拥有用户真实问题的解释、返回结果的判断、整体重统一和最终报告责任。局部执行可以委派，整体责任不能委派。

### P2. Portfolio 连续性与用户优先

每个非终止 Matter 都必须保留并可恢复。用户输入拥有最高调度优先级；无关用户任务只有在当前工作被安全 checkpoint 后才能抢占。

### P3. 集中注意与交错推进

任一时刻只有一个有界工作项获得最高注意力。可执行 Matter 在有意义的完成、yield 或阻塞边界上交错推进；等待 Matter 由事件唤醒，不主动轮询。

### P4. Matter 可恢复

每个 Matter 必须拥有足够的持久状态，能够恢复拟回答、当前工作上下文、已派发 worker、待验证结果以及下一步或返回条件，而不依赖 session memory。

### P5. 行动监护

每项被诱导出的行动都必须经过计划、派发、callback、验证、返工、过期或验收中的真实收束。worker 返回或状态变化不能静默变成完成。

### P6. 认识责任

每条重要 claim 都必须有真实证据与诚实的认识状态，并说明该证据和推导为何覆盖 claim 的真实范围。整份报告还必须单独论证其选取范围为何相对用户问题足够完整。

### P7. 证据支配修订

被接受的证据可以强化、收窄或推翻拟回答。证据破坏共同基础时，所有受影响的推导和行动必须重新统一、重新规划，不能因既有投入而保留失去根据的结构。

### P8. 协调权不可委托

parent Chamberlain 负责 Matter 选择、优先级、权威状态更新、验收、重统一和最终报告。worker 可以提供有边界的工作或建议，但不因此取得这些权力。

### P9. 表示形式可进化

ledger、stack、queue、表格、标签或 prompt 章节，只有在保留其上位语义时才是可接受的转换。当前形式可以演进，但不能改变 Q，也不能静默收窄上位保证。

## 4. 核心术语

### Portfolio

当前 Chamberlain 已知的全部 Matter。

### Matter

一个独立的用户事项，拥有自己的最终回答责任和生命周期。无关用户请求创建新 Matter；相关指示修订已有 Matter。

### 工作项（Work Item）

主 Chamberlain 可以集中处理直至完成、yield 或阻塞的有界事项。Portfolio 在工作项边界上交错，而不是把整个 Matter 一次执行到终态。

### 原生任务（Native Task）

由 host/OMP task 系统表示的执行或验证单元。`Task` 一词保留给这个执行层对象。

### Working Stack

一个 Matter 的 live control state。它保存当前 frame 链、暂停与返回关系、selected work item，以及理解和恢复当前 Matter 所需的最小 popped history。

### Matter 账本（Matter Ledger）

一个 Matter 唯一的持久账本。每个 Matter 恰好一本，不再另设 master ledger 或独立 worker ledger。Workers Ledger 是 Matter 账本的一部分。

## 5. D.root——两层运行模型

```text
Portfolio Chamberlain
├── Matter 调度器
├── 原生任务状态
├── 全局经验教训
└── Matter[]
    └── Matter 账本
        ├── 拟回答
        ├── Working Stack
        └── Workers Ledger
```

外层负责 Matter 调度，内层负责选中 Matter 的回答中心闭环。

总信息流是：

```text
用户 Q / 修订
→ Matter 拟回答
→ 暴露未闭合义务
→ Working Stack 选择工作项
→ 原生任务与 Workers Ledger
→ 本地执行或派发 worker
→ callback / 结果
→ parent 验证
→ 更新原生任务和 Workers Ledger
→ 已接受证据修订拟回答
→ 重算 Working Stack 与受影响行动
→ 行动和回答共同收束
→ 最终报告
```

## 6. Portfolio 调度器

### 6.1 全局状态

全局维护：

- 已知 Matters 列表及各自 Matter 账本位置；
- 当前 selected Matter 与 selected work item；
- 哪些 Matter 有 ready work；
- 哪些 Matter 正在等待事件；
- 用户抢占后的直接 resume target；
- 原生任务的 readiness、依赖和执行状态；
- 已证明可跨 Matter 使用的 scoped lessons。

这不是另一份 ledger，而是作用于各 Matter 账本之上的全局控制状态。

### 6.2 选择与轮转

任一时刻只有一个有界工作项获得主 agent 的主要注意力。

当前工作项完成或到达真实 yield point 后：

- 仍有 ready work 的 Matter 回到 ready queue；
- 等待 worker、用户或外部事件的 Matter 停放至事件到来；
- 已终止 Matter 不再进入调度；
- 调度器选择下一个 ready Matter。

这是一种事件驱动的 interleaving，不是对 blocked Matter 的主动 polling。

### 6.3 Async callback

worker callback 必须归属原 Matter，并形成 verification work item。它使该 Matter 重新 ready 并进入调度顺序，但不打断当前 selected work item。

多个 callback 可以排队；每个 callback 被选中后再获得集中验证。

### 6.4 用户抢占

用户消息先于普通调度处理。Chamberlain 必须先判断它是：

- 修订 selected Matter；
- 修订另一个已知 Matter；
- 还是创建新的无关 Matter。

无关且可执行的用户请求在下一个安全边界抢占：

```text
checkpoint 当前 Matter 账本
→ 保存被中断工作项为 resume target
→ 创建新 Matter 及其账本
→ 将用户工作放到 scheduler top
→ 集中处理
```

用户优先级不能伪装成 task dependency。被中断 Matter 不逻辑依赖新 Matter，只是在调度上让位。

已运行的 worker 默认继续，除非新指示使其失效或明确要求停止。它们的 callback 回到原 Matter 并等待验证。

抢占 Matter 完成、阻塞或 yield 后，优先恢复被中断 Matter，除非用户再次改变优先级。

## 7. 一 Matter 一本账

Matter 账本由 parent Chamberlain 独占权威写入。worker 只写自己的有边界产物；默认不得直接修改 Matter 账本、Working Stack、原生任务状态或拟回答。

一本 Matter 账本就是该 Matter 的完整可恢复包：

```text
Matter 账本
├── 拟回答
├── Working Stack
└── Workers Ledger
```

三部分是同一 Matter 的不同责任面，不是三个独立权威。

## 8. 拟回答

### 8.1 责任

拟回答是持续演化的候选最终报告，也是让最终报告变得负责任的当前计划。它说明目前能回答什么、为什么，以及还缺什么。

它不必是润色后的 prose，可以是紧凑论证、提纲、比较框架或待填的报告结构。形式服从 Matter。

### 8.2 重要 claim

每条重要 claim 必须能恢复三个语义位置：

```text
Claim
Support
Validity
```

`Claim` 表达实际命题、量词和断言强度。

`Support` 给出 `/no-flattering` 的认识状态以及实际证据或推导。`[源证]`、`[转述]`、`[推导]`、`[假设]`、`[未闭合]` 等标签只揭示来源和认识状态，本身不证明真值。

`Validity` 保存阻止证据被越阶使用所需的内容：真理条件与范围、从证据到 claim 的局部 coverage/completeness argument、实质 counter-condition 与残余缺口。

这三个位置可以固定，但内容按真实复杂度伸缩。局部 claim 的 `Validity` 可能只需一句；跨路径或系统级 claim 必须承担更强的覆盖论证。未触发的条件不要求用虚构的 `N/A` prose 填满。

约束关系是：

```text
Claim 的强度
≤ Support + Validity 实际建立的强度
```

### 8.3 Claim-level completeness

一个真实的局部观察，只能支持它实际覆盖的范围。

例如：

```text
if X:
    f(...)
else:
    g(...)
```

`f` 内的 guard 只能证明相关执行路径确实进入 `f` 并经过 guard 时的安全性。只要 `g` 在 claimed scope 内可达且没有被验证，就不能推出无条件安全。

更广的 claim 必须说明其范围内所有相关路径、入口、override、条件或其他 counter-case 已被覆盖或排除。若无法成立，只能继续取证、缩窄 claim，或保留具体的 `[未闭合]`。

### 8.4 Report-level relative completeness

拟回答还必须单独维护：为什么报告选择的调研、比较或实现范围，相对用户 Q 已经合理且足够。

claim-level soundness 不自动推出 report-level completeness。报告可以只写真话却遗漏完整的一类问题；广泛覆盖主题也不能让单条无依据 claim 成立。

报告级论证要说明：所选范围覆盖了用户 Q 诱导出的重要义务，相关 counter-condition 已按合理方法检查，剩余限制没有被隐藏。

## 9. Working Stack

Working Stack 是 Matter 的 live control state，不是完整原始历史。

每个 live frame 保存：

- 为什么父工作暂停并进入当前上下文；
- 当前 frame 必须建立或交付什么；
- selected 或 next work item；
- 什么条件允许 pop；
- pop 后返回哪里。

普通计划、并行 worker、验证和返工留在当前 frame 的 task graph 中，不因最近获得注意就自动成为新 frame。

只有真实 context switch 暂停父工作时才 push frame。frame 被接受或过期后 pop，只保留理解当前 Matter 或审计仍然重要决策所需的紧凑记录。

Working Stack 保存恢复所需的结构演化；语义结论属于拟回答，worker 执行历史属于 Workers Ledger。

## 10. Workers Ledger

Workers Ledger 是 Matter 账本中的行动监护部分。

继续采用当前有效 schema：

```text
| task/stage | acceptance | dependencies | owner + worker/run ID | status | evidence/result | next step | lesson/expiry |
```

当前状态语义仍可使用：

```text
queued, blocked, in_progress, verification,
accepted, rework, failed, expired
```

schema 不是本体。每行必须足以恢复：

- 工作为什么存在，由哪个 Matter 义务诱导；
- 什么结果才可接受；
- 谁负责执行；
- 实际状态与依赖；
- 返回了什么证据，是否被 parent 独立接受；
- 被接受结果如何改变拟回答或下一步；
- 工作为什么返工、失败或过期。

现有字段若能保存这些语义，就不新增列。`task/stage`、`evidence/result` 和 `next step` 可以分别承载诱导理由、回答影响与重规划关系。

### 10.1 权威与同步

原生任务系统是 task status、dependency 和 readiness 的唯一执行权威；Workers Ledger 是所属 Matter 的完整可读镜像。

发生派发、callback、状态、验收、返工、失败或过期变化时：

```text
先更新原生任务状态
→ 在同一协调轮次更新 Matter 账本
```

若两者冲突，暂停后续派发，依据原生状态和实际证据修复账本。

worker report 永远是待验证证据，不是 acceptance。

## 11. Matter 内部行动闭环

```text
拟回答暴露未闭合义务
→ Working Stack 选择有界工作项
→ 形成计划与 acceptance
→ 创建或更新原生 Task
→ Workers Ledger 记录行动
→ Chamberlain 自做或派发
→ callback / 本地结果返回
→ parent 检查实际证据
→ 更新原生 Task 与 Workers Ledger
→ 已接受证据修订拟回答
→ 重规划 Working Stack 与受影响工作
```

每次验证都必须回答：

```text
这项工作是否真的满足 acceptance？
被接受的结果改变了 Matter 的什么回答和下一步？
```

## 12. 证据、反思与重统一

返回 claim 必须检查其实际指代、来源、条件、推导和覆盖范围。worker summary、单个引用位置以及多个 worker 的一致意见都不能替代该判断。

验证后：

- 符合当前整体的证据补全或强化相应义务；
- 只建立更窄结果的证据收窄 claim 或留下具体 residual；
- 局部冲突触发局部返工或重分类；
- 推翻共同前提的证据触发拟回答及其诱导工作中全部受影响部分的重构。

Re-unification 不是继续堆补丁。有效证据继续保留，失去依据的分类和推导失去权威，剩余计划从新的整体重新推出。

只返工或 expire 真正受影响的 native tasks、ledger rows 和 stack frames。回答变化不自动重置无关工作。

## 13. Checkpoint 与恢复

切换 selected Matter 前，Chamberlain 必须更新其 Matter 账本，使其无需依赖 session memory 即可恢复。

Checkpoint 通过已有三部分保存：

- 拟回答当前成立到哪里，还有什么未闭合；
- Working Stack top、return relation、next work item、等待或 pop 条件；
- Workers Ledger 中正在运行的 worker、待验证 callback、返工和原生任务镜像；
- Matter 为什么 yield，以及什么事件或调度决定使它重新 ready。

Matter 账本本身就是 checkpoint package，不新增独立 checkpoint artifact。

## 14. 主 agent 自做与委派

Chamberlain 通常负责协调，但不能为了不直接工作而无条件委派。

当正确性强依赖 Portfolio/Matter 全局上下文、涉及权威协调状态、决定验收或最终综合，或者直接验证比委派更便宜清楚时，parent 应亲自完成。

只有当贡献边界清楚，且节省的工作或所需能力大于上下文传递、监督、验证、整合和权限风险时，委派才成立。

parent 保留：

- Matter 身份、优先级与选择；
- 用户 Q 的解释；
- 拟回答的权威重统一；
- 原生任务、Working Stack 和 Matter 账本更新；
- worker acceptance；
- 最终报告。

subagent 可以协助计划、反思或高层分析，但其输出在 parent 验证和吸收前只是建议。

### 14.1 Fork 边界

fork 继承上下文，不继承 parent authority。除非明确授予另一段有边界写入范围，否则不得修改 parent 原生任务、Matter 账本、Working Stack、scheduler state 或用户可见结论。

需要大量 parent context 的咨询任务可以使用 fork；要求独立确认的审计通常使用 fresh context。两者都由 parent 验收。

## 15. 经验教训

Matter-local observation 先留在该 Matter 的 `lesson/expiry` 材料中。

只有当经验在原 Matter 之外的适用范围获得证据支持或结构性理由时，才提升到全局可复用记录。一次 worker 失败、工具事故或局部实现细节不能静默成为普遍规则。

全局 lessons 是带 scope 的候选知识，只能辅助未来计划，不能覆盖新 Matter 的 Q、直接证据或项目规则。

## 16. 完成与最终报告

Matter 只有在相关行动义务和回答义务共同闭合时完成。

行动闭合意味着已派发和本地执行工作都已验收、明确返工、因 blocker 失败，或有记录地过期；任何已分配工作都不能只存在于 chat 或 worker report。

回答闭合意味着最终报告：

- 不超过证据、推导、真理范围与 claim-level completeness 实际建立的强度；
- 论证其覆盖为何相对用户真实问题足够完整；
- 暴露实质未闭合边界和 counter-condition；
- 准确反映已验证的底层工作。

报告形式服从内容。横向矩阵、纵向 trace、从广到狭或其他结构，只在 Matter 实际需要时采用。

## 17. 权威边界

```text
用户决定与 approved brief
→ Q、目标、优先级和已接受取舍的权威

原生任务系统
→ 执行状态、依赖和 readiness 的权威

实际文件、测试、运行和一手来源
→ 证据基础

拟回答
→ Chamberlain 当前可修订的综合判断

Working Stack
→ live context 与 attention 投影

Workers Ledger
→ Matter 账本中的行动与验证镜像

Portfolio 调度器
→ 全局选择、抢占与恢复顺序
```

表示与权威冲突时修复表示：ledger row 不能改变任务事实，stack 不能制造 dependency，拟回答不能修改用户 Q，全局 lesson 不能覆盖当前证据。

## 18. 可进化契约与奥卡姆边界

当前 ledger schema、status vocabulary、stack representation 和 scheduling policy，是因为它们在已有场景中满足上位 P，才成为被接受的 D/I。

当具体失败证明它们无法保存义务、区分必要状态、恢复工作或阻止错误结论时，可以演进。演进应修改能够消除失败的最小 owning representation。

“未来可能有用”“看起来更完整”“其他框架有”“让每行更整齐”或“已经投入很多”都不足以新增字段、状态、表格、审查步骤或 artifact。

固定字段可以存在，只要它表达真实语义差异。条件性内容只在条件实际命中时展开，完成绝不以填满所有格子定义。

## 19. 与当前 prompt 的关系

### 19.1 保留

本设计保留：

- 原生任务作为执行权威；
- 一个 selected item 获得集中注意；
- Working Stack 与 Frame 的 push/pop；
- 异步派发和 callback verification；
- acceptance、dependency、owner、status、evidence、next、rework、failure 与 expiry；
- worker report 只是 evidence；
- 原生任务和 ledger 同轮同步；
- actual file、diff、source 与 test 验证；
- conflict-driven rework 与 evidence-backed lessons。

### 19.2 重定向与扩张

本设计将：

- `Mission` 改称 `Matter`，避免与原生 Task 混淆并强调一个独立用户事项；
- 改为每 Matter 一本账，不再使用 master ledger 加独立 mission/worker ledgers；
- 将 Matter 账本从 worker-task tracking 扩张为“拟回答 + Working Stack + Workers Ledger”；
- 让 Portfolio 通过有界工作项明确 interleave Matters；
- 让用户抢占先 checkpoint，并用 scheduler priority 而不是假依赖实现；
- 要求每个 accepted callback 同时更新行动状态与回答理解；
- 将证据要求扩张到 claim-level truth range 与 coverage argument，而不只管理报告级调研范围；
- 保留当前机制，但把它们定位为可进化实现，而不是普遍本质。

### 19.3 移出核心

下列规则不应定义 Chamberlain 核心：

- 异步派发后无条件停止其他有用本地工作；
- 固定的 broad → architecture → lifecycle 报告顺序；
- Matter 不需要时仍强制对称比较、横向矩阵或纵向 trace；
- 属于 caller/host protocol 的 Bolder sentinel 与验证命令限制。

## 20. 未来 SKILL 的信息架构

设计确认后，SKILL 正文可按以下责任边界组织：

1. **Purpose and Full Responsibility**：定义综合多 Matter 管家与最终回答责任。
2. **Portfolio Scheduling**：定义 Matter、工作项、ready/waiting、callback、interleaving 与用户抢占。
3. **One Matter, One Ledger**：定义拟回答、Working Stack、Workers Ledger 组成的可恢复包。
4. **Planning, Dispatch, and Action Guardianship**：保留原生任务、acceptance、scope、派发与 ledger 同步。
5. **Evidence, Reflection, and Re-unification**：定义 claim-level Support/Validity、callback 验证、回答更新与受影响工作重规划。
6. **Main-Agent and Worker Authority**：定义 parent 不可委托责任、普通委派判据和 fork 边界。
7. **Completion and Reporting**：定义行动/回答共同闭合与负责任报告。
8. **Authority and Evolution**：防止双重权威，并允许表示形式在不替代语义的前提下演进。

这些是文档责任边界，不是固定 runtime stages。

## 21. D 满足性

- Portfolio scheduling、事件驱动 readiness 与 checkpoint/resume 满足 P2–P4；
- 一 Matter 一本账与原生任务同步满足 P4–P5；
- 拟回答中的 Claim/Support/Validity 与报告级 completeness 满足 P1、P6；
- 验证、反思、重统一和受影响工作重规划满足 P7；
- parent ownership 与 fork boundary 满足 P8；
- evolution contract 与 semantic-field criterion 满足 P9。

固定报告格式、ledger 字段或调研阶段都不被升格为 Q/P。当前机制被保留，是因为其 satisfaction relation 成立，而不是因为当前形式不可改变。

## 22. 留待 I 决定的事项

D 不要求下列问题只有一个普遍答案，留待 SKILL 写作或后续实现：

- Matter 账本的物理目录与命名；
- 全局 Matters 列表具体持久化在原生任务、小型 registry 还是 host state；
- scheduler order 与 resume target 的具体语法；
- 大型拟回答是内嵌还是链接，但 Matter 账本必须始终是 owner，并保存完整恢复位置和状态；
- 特定 Matter 的 claim 采用标题、紧凑标签还是表格。

任何 I 都必须保留本文定义的权威、信息流、checkpoint、证据与完成语义。
