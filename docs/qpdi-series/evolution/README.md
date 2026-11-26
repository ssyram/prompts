# QPDI Series — 公开设计演变

## 文档性质

本文记录 `/qpdi` 与 `/qpdi-compose` 的关键设计压力和裁决，不保存逐字对话、完整执行时间线或内部推理。当前运行权威仍是 [`current/qpdi.md`](../../../current/qpdi.md) 与 [`current/qpdi-compose.md`](../../../current/qpdi-compose.md)；本文解释它们为何形成，不另立规范。

## 总体变化

QPDI prompt 从“一个包含理论、流程和文件安排的综合入口”，逐步分成：

- `/qpdi`：静态认知、论证关系与轻量应用；
- `/qpdi-compose`：Q 与任意深度 D 的写作；
- 专项 SCCO prompt：系统性检查与裁决；
- draft：在不丢失新需求的前提下重新压缩 official。

核心模型最终收敛为 `D-frame = P + M + R`，并把 commitment、pre/post、local/global、component/leaf、状态和视角重新放回各自所属概念。

## 1. 单一运行入口

**压力：**项目内 QPDI skill 与全局 `/qpdi` 并存，会形成两套内容和维护漂移。

**裁决：**删除项目局部副本，只保留全局 `/qpdi` 作为运行入口。

**边界：**principled-ai 项目中的理论和设计材料仍可作为研究依据，但不自动成为第二个可调用 skill。

## 2. 静态认知与行动方式分开

**压力：**早期材料把 QPDI 理论、强制流程、文件结构和自动执行混在一起；模型容易为填满形式而偏离用户问题。

**裁决：**`/qpdi` 先陈述静态 Q/P/D/I、Structural Locality、Discover 与 SCCO，再给可被用户覆盖的轻量工作流；不以 FSM 为启用前提。`Optimization` 统一改为 `Optimality`。

**留下边界：**推荐 `/论域 /论证 /锚定 /行动` 作为对齐凭证，但它不是封闭回复格式。

## 3. Compose 从通用认知中分出

**提交锚点：**`6c0638f`（2026-08-12，`refine QPDI prompts and add compose guide`）。

**压力：**“理解 QPDI”和“把 Q/D 写好”具有不同消费者。Q.I 又长期混入任务背景，D 被误设为固定 architecture/detail 两层，Compose 还可能越界写 I。

**裁决：**新增 `/qpdi-compose`：

- Task Overview 与 Q.I 分离；
- Q.I 聚焦 correctness、wanted 与 boundary；
- Q.A/Q.E 分别承担认知命题和经验材料；
- D 可任意深度展开；
- Compose 只写 Q/P/D，不生成 I；
- SCCO 只作为可检查性，不复制完整审查流程。

**不足：**首版 Compose 约 569 行，字段、反模式和 proof/trace 说明重复，写作能力被模板体积遮蔽。

## 4. D/P 关系 re-unification

**提交锚点：**`7552fb7`（2026-08-12，`unify QPDI design model`）。

**压力：**commitment、pre/post、设计规则、component D 和“D 回答 Q”等说法互相并列，尚未形成统一设计本体。

**裁决：**确立：

```text
D-frame = P + M + R
P = 当前 frame 的全部可判定设计性质
M = components + composition
R = Derivation + Satisfaction + 必要的 Optimality
```

Component D 在父层是 M 的组成角色，展开后才成为自己的 D-frame；只有 D.root 直接回答 Q，child 从 parent contract 推出自己的 P。Commitment、pre/post、invariant 和禁止条件收回 P。

**留下边界：**Q.A 与 Q.E 不能合并；二者可以通过 `supported-by` 消除重复存储，但仍分别是认知命题和经验材料。

## 5. 叶子完成粒度与设计限制

**提交锚点：**`c1e150e`（2026-08-20，`more polish`）。

**压力一：**“没有剩余设计问题就是叶子”过于主观，任意深度 D 也缺少可导航的 handoff。

**变化：**Compose 增加 `expands-to | implemented-by`、thin/thick 判断和不同 I 类型的叶子详细度。

**压力二：**局部 D/I 无法履行原 contract 时，容易把设计限制直接写回 Q，或把已接受的 narrowing 错误向下传播。

**变化：**`/qpdi` 增加 actual guarantee 与 parent contract 的逐层比较：本层仍履约则停止，真实变弱才向上重算，D.root 最终与 Q 对账。

**后续压缩：**draft 保留通用展开判据、handoff 和向上递归，未复制按 code/prompt/schema-config 分类的完整清单；这些细节可由专项设计方法承担。

## 6. Whole-level integrity

**提交锚点：**`83e9460`（2026-08-31，`docs(qpdi): add whole-level integrity guidance`）。

**压力：**各 component 分别满足 contract，不足以证明 composition 作为 whole 满足 P；共享前提、接口和 global P 仍可能冲突或漏接。

**裁决：**R.Satisfaction 和 SCCO 必须检查 frame 整体，不能把局部 pass 直接合取成 whole pass；对上 guarantee 也必须可见。

**留下边界：**whole-level integrity 是 Satisfaction 与 SCCO 的应用，不新增第五个 SCCO 维度，也不宣称 Q 或整体形态绝对完备。

## 7. Q 的可选整体组织

**提交锚点：**`9ea5b57`（2026-09-01，`docs(qpdi): clarify optional Q organization`）。

**压力：**条目化 Q 可精确引用，却可能让读者看不出这些条目为何共同出现；简单添加“最高 idea”又会伪造统一、压平多个不可还原 Q.I。

**裁决：**材料有真实 whole–part 关系时，可增加非权威 whole view；它呈现 Q.I 的展开、限制或张力，并把 Q.A/Q.E 放回实际语境。没有真实中心时保留并列。只有用户确认整体表达本身定义规范方向时，它才进入 Q.I。

**留下边界：**发生史可以解释目标为何显著，不能缩窄后来确认的更一般追求；whole view 也不能复制一份 Q。

## 8. Occam draft

**状态：**[`drafts/qpdi.md`](../../../drafts/qpdi.md) 与 [`drafts/qpdi-compose.md`](../../../drafts/qpdi-compose.md) 是尚未替换 official 的候选。

**压力：**official 已覆盖真实需求，但静态定义、写作说明、目录路由、反模式和 I 类型清单共同增长。直接删短会丢失 whole Q、guarantee propagation 和 leaf handoff；直接合并 official 又会恢复膨胀。

**当前变化：**

- local/global 收回 `P.scope`；
- Q.A 与 Q.E 以 support 边去重；
- R 明确 Derivation、whole-level Satisfaction 和条件性 Optimality；
- actual guarantee 作为 Satisfaction 结果逐层向上比较，不另立本体；
- I 具体化 M 并保持 P；
- Discover 增加纯 R 缺口；
- whole Q、`expands-to | implemented-by` 与最小 owner 路由被压缩保留；
- 固定 `docs/design/` 和按 I 类型的长清单未进入核心 draft。

**未决：**draft 需要代表场景和独立内容审阅后，才能决定是否替换 official。篇幅下降本身不是通过条件。

## 9. 拒绝以“三观”解释 Q

**压力：**Q.I/Q.A/Q.E 的结构边界已经清楚，但仍需说明为什么它们共同构成人的不可委托问题意识，而不只是材料分类表。

**被拒候选：**曾尝试把 Q 解释为“三观”在具体问题中的体现，并把 Q.I、Q.A、Q.E 分别联系到价值、世界认知和经验。历史核验表明，这三个日常概念来源异质、作用范围重叠，也不构成同层、互斥或穷尽的哲学划分；其中“人生观”尤其横跨规范、认知与经验，无法与 Q.E 对应。

**裁决：**从 active prompt 和设计中删除该类比。Q 改由问题结构自身说明：意识指向当前问题，Q.I 定义判断方向，Q.A 承载认知前提，Q.E 承载经验依据。Intentionality 只说明对象指向，不与 Q.I 的规范意图混同。

**边界：**本节只保存拒绝理由，防止以后因三项数量相同而重新引入；“三观”不再构成 QPDI 的定义、依据或写作方法。

## 稳定下来的区分

| 容易混同 | 当前区分 |
|---|---|
| Task Overview / Q.I | 前者导航，后者定义规范方向 |
| Q.A / Q.E | 被采纳命题 / 经验材料 |
| P / commitment / pre-post | 后两者是 P 的内容或表达形式 |
| Local / global P | 同一 P 的不同 scope |
| D-frame / component / leaf | D 是 P/M/R；后两者是相对角色或展开状态 |
| R.Optimality / SCCO.Optimality | 对象级论证 / 对论证充分性的检查 |
| Contract / actual guarantee | 上游要求 / 当前 M 实际可证明性质 |
| Whole view / 正式 Q | 阅读组织 / 可被引用的权威条目 |
| 内容确认 / 事实验证 / 写入授权 | 三条独立状态轴 |
| D / I | 实现无关的性质与模型 / 具体 realization |

## 被明确拒绝的终点

- 以“三观”或其他日常人格分类为 Q.I/Q.A/Q.E 奠基；
- 把 QPDI 绑定成固定 FSM 或强制阶段；
- 把 D 固定为 architecture/detail 两层；
- 让每个 child D 直接重新回答 Q；
- 把 commitment、pre/post 或未规定空间立成 P 的并列本体；
- 用局部 component pass 代替 whole-level Satisfaction；
- 让人接受设计限制自动变成 Q.I 修改；
- 在 Compose 内生成 I 或复制完整 SCCO tribunal；
- 固定所有项目使用仓库根 `docs/`；
- 只因更短就把 draft 宣布为最小充分。

## 后续变化如何记录

新增阶段只记录能够改变现行设计判断的内容：

```text
暴露的实际压力：
原设计为何不足：
新增、收窄或删除的义务：
它被吸收到哪个既有概念：
仍不成立或未验证的范围：
实现 / 提交证据：
```

一次措辞润色、单次模型误读或过程工具故障，不自动形成永久设计阶段。只有它暴露稳定失败并改变需求、P/M/R 或运行 prompt 时才进入本文。
