# skill-routing — 用哪个 skill 的经验记录

> 不是 skill 规约，是**使用 skill 时的选择经验沉淀**。每次发现选错了 skill 就回这里加一条。
> 这个文件本身不被 `/<name>` 调用——它是给主对话里的 Claude 看的 routing 参考。

---

## principle-derivation v1 vs v2

两个 skill 同骨架（意识 → 原则 → 推导），但姿态相反。

| 场景 | 选 |
|---|---|
| 已收敛分析的**事后整理** | **v1** |
| audit 报告、code review、设计回顾、bug 复盘 | **v1** |
| 引导读者**从零产生独立洞察** | v2 |
| 教学材料、概念入门、新人 onboarding | v2 |
| 自己设计要立结构 / 提原则 | v1 |
| 单层、扁平、原则少 | v1 |
| 多层认识在不同时刻形成，要展示这一过程 | v2 |
| 主体不清楚 / 还在共同探索 | 都不用，先 trial-and-error |

### v1 vs v2 选用案例：audit 报告

**情境**：审计 ciim2022p6 trajectory，3 份独立 reviewer 产出的 audit 报告被要求"用 principle-derivation-v2 语言重写"。

**结果**：审视者**已经知道答案** → v2 的"邀请同行"姿态变成"把事后回顾包装成意识展开"——principle-derivation-v2 §四警告的"用方法替代思考"几乎踩中。具体内容（commit hash / 字段值 / 覆盖检查结论）被叙述节奏遮蔽，决策者读起来反而不知道审视者具体发现了什么。

**判据**：审计 / 复盘这种"已收敛分析的事后陈述"用 v1（直接给结构 + 推导表）。v2 用于"还在展开中" / "邀请同行"。

**记忆点**：v2.md §六已经有这张选用表，**容易在被任务 prompt 推进时忘记查表**。下次启动任何"重写 / 重表达 / 重新组织已有内容"的任务，先回 v2.md §六对一下姿态匹配。

---

## workflow / workflow-audit / hoare-audit

三个 skill 名字都带 workflow / audit，容易混。

### 一句话区别

- **workflow** — 开发**做事**的规范，不审计任何东西
- **workflow-audit** — 审视一份 **GitHub PR** 的多方向并行 review（驳回优先）
- **hoare-audit** — 审视**代码 vs 设计 spec** 的正确性（spec 必需）

### 对比

| skill | 输入 | 输出 | 是否需要 spec | 主体关系 |
|---|---|---|---|---|
| workflow | 任务描述 | 设计 + 代码 + 测试 | 否（自己产生设计文档） | 做事的人 |
| workflow-audit | GitHub PR 编号 | review 报告 | 项目 workflow.md 优先，可降级 | 外部审查（不同主体）|
| hoare-audit | 代码 + 设计 spec | findings + auto-fix | **必需**（"No audit without a spec"） | 自审 / 他审都可 |

### 选用决策

- 在写新功能 → **workflow**
- 别人提了 PR 要 review → **workflow-audit**
- 拿到一份代码，想审"是否满足设计意图" → **hoare-audit**（先确认有 spec；没有 spec → 先建 spec 或拒绝 audit）

### 联系

- workflow 立 spec → hoare-audit 验代码满足 spec
- workflow-audit 跟"做事的人"是不同主体（外部 PR review）
- hoare-audit 跟"做事的人"可以是同一主体（自审）也可以是外审

### 容易混淆点

- **workflow 含的"代码审核"段（§4.2）**不是独立 audit skill——那是开发过程内的自审，完全在 workflow 内做完，不调 audit skill
- **workflow-audit vs ultrareview**：都是 PR review，但 workflow-audit 是单 agent 做 multi-direction；ultrareview 是 multi-agent cloud parallel（user-triggered 计费）。ultrareview 不能由 Claude 自己调起
- **hoare-audit vs trajectory-audit**（如 auto-proof-trajectory-audit）：前者审代码正确性 vs spec，后者审 agent 跑动轨迹是否符合设计精神。不同对象、不同输入

### `/hoare-audit` 与 Lv0–Lv4 的关系

参见 `prompts/meta-principles.md` 的层级定义。

- `/hoare-audit` 工作在 **Lv3 ↔ Lv4**：design report 描述应该做什么，代码 / YAML 是否真的做到了
- **Lv3（方案 / design report）= `/hoare-audit` 的硬底线 spec**；缺则先 `/hoare-design` 还原
- **Lv1 / Lv2 不是 `/hoare-audit` 的输入**；它们是 Lv3 出现「方案本身可疑」时的**仲裁杠杆**。有则减少 Decisional 数量，无则不阻塞 audit（只是更多歧义被推到 Decisional gate）
- **`/hoare-design` 只产 Lv3** descriptive spec，不产 Lv1 / Lv2
- **立 Lv1 / Lv2 是 `/charter-craft` 的工作**，不是 `/hoare-audit` 的工作
- 「函数级 Pre/Post/Invariant」是 Lv3 内部颗粒度选择，不是新的层级

**误区警示**：曾把 Lv0–Lv3 误读为「意图 / 架构 / 接口 / 函数」四级文档粒度。这是错的。Lv 是制宪体系的推导层次，不是文档粒度。

---

## audit 类 skill 的共性："驳回优先 / 先反后反反"

workflow-audit / hoare-audit / charter-craft §4.8 都共享一个机制——**审视者应先挑刺，再 counter-challenge**。

理论根据（charter-craft §4.8.1）：LLM 转换立场就能挑出问题（高 recall），单纯 confirm 几乎不发现问题（低 recall）；counter-challenge 过滤无效挑刺（高 precision）。反两次是经验最优点。

实际项目里：
- audit-principles.md 制宪用了 §4.8 先反后反反
- workflow-audit 直接把多方向并行 challenge 嵌入流程
- hoare-audit 用 spec 作为 ground truth 让 challenge 有可证伪 anchor

**判别**：写任何新 audit 流程，"先反后反反"应该是默认结构，不是 nice-to-have。

---

## schema-matching-agent：认知图式自适应路由

schema-matching-agent 不是传统意义上的 task skill，而是一个**认知姿态调节层**——它在回复前强制执行六对图式维度（抽象↔具体、中心↔边界、融贯↔符合、精确↔召回、压缩↔展开、构成↔调节）的显式判断，然后根据判断结果组织回答。

### 核心价值：纠正大模型默认输出偏置

裸调用大模型时，用户经常"红温"（感到烦躁、无力、被敷衍）——这通常不是因为回答内容错误，而是因为**认知姿态错配**。大模型有强烈的默认偏置：

- 倾向给出**中心性**回答（典型、泛化、教科书式），忽略用户真正关心的边界和例外
- 偏好**融贯**叙事（逻辑自洽、结构漂亮），即使与实际不符
- 压缩为**构成性断言**（"答案是X"），即使问题本身需要探索和调节
- 在**抽象-具体**轴上往往停在中间偏抽象，既不够原则也不够实操

schema-matching-agent 通过每轮强制 `<think_schema>` 判断来对抗这些偏置，显著缓解"答非所问""正确但无用""合理但不真"这三类最消耗用户耐心的体验。

### 适用场景

| 场景 | 为什么适合 | 示例
|---|---|---|
| **多轮对话中用户复杂意图的对齐** | 每轮强制重新判断图式，不会固化到初始姿态；`<steering>` 机制允许用户显式转向 | 用户从"给我概览"逐渐深入到"这个边界 case 怎么处理"，Agent 自动从抽象+中心移向具体+边界 |
| **人文知识、人文思考的通用助手** | 人文问题天然高模糊度、多视角，需要融贯与符合之间灵活切换，构成与调节之间恰当落位 | 讨论一个哲学概念时，Agent 能区分"给出确定解读"和"展示不同学派的张力"两种需求 |
| **各种技术方案的讨论（不涉及极度具体领域的落地细节）** | 方案讨论在精确/召回、压缩/展开、构成/调节三对维度上频繁切换；需要逻辑一致（融贯）但也要可验证（符合） | 架构选型讨论中，Agent 自动识别用户是在"做决策"还是"做探索"，切换构成/调节姿态 |

### 不适用场景

| 场景 | 为什么不适合 |
|---|---|
| 固定格式的执行任务（翻译、数据提取、代码生成） | 图式判断引入不必要的开销，执行型任务需要的是确定性和一致性，不是姿态调节 |
| 极度专业领域的精确诊断（医疗、法律文书） | 这些场景要求"符合+构成"端固定姿态，图式灵活性反而有害 |
| 需要高速批量处理的流水线任务 | 每轮强制 think_schema 有 token 和延迟成本 |

### 选用判据

- **问自己：这个回答的"姿态"比"内容"更重要吗？** 如果是 → 考虑 schema-matching-agent
- **问自己：用户会因为我们"怎么答"而红温，还是因为"答错了"而红温？** 如果是前者 → schema-matching-agent 有价值
- **问自己：这个任务需要每轮重新评估回答方式吗？** 如果不需要（一次性确定就够） → 图式机制是负担，不用

---

## /qpdi、/qpdi-compose 与 SCCO 审查：认知、写作、检查分开

### 一句话区别

- **`/qpdi`** — 理解 QPDI 静态结构，识别用户论域，拆解复合论证，把材料锚定到 Q / D 及 D 内 P，并用 Discover 定位缺口。
- **`/qpdi-compose`** — 专门把材料写成 Q 与任意深度的 D 展开；P 作为 D 内规则一起写；可以按明确授权路径落盘；不写 I。
- **`/scco-recall` / `/qpdi-tribunal` / `/hoare-audit`** — 检查已经形成的产物，不负责代替 compose 起草 Q/D。

### 当前路由

| 场景 | 当前选择 |
|---|---|
| 理解 QPDI、判断用户在谈哪类内容、分析 Q/D/P 关系 | `/qpdi` |
| 从用户复合话语保全论证，再整理、起草或落盘 Q | `/qpdi-compose` |
| 从 Q 起草设计，或把一个 D 节点继续展开成下一层 D | `/qpdi-compose` |
| 重组已有 Q/D/P 文档，补清 reasoning、作用域和 trace | `/qpdi-compose` |
| 对大对象做 SCCO 高召回找漏 | `/scco-recall` |
| 对已有产物做 SCCO 对抗裁决与问题分流 | `/qpdi-tribunal` |
| 用强 spec 审代码是否满足设计 | `/hoare-audit` |

### `/qpdi-compose` 的边界

- 写作对象是 **Q + D 展开链**；P 属于 D 内部，不作为独立顶层产物。
- D 不固定为“architecture + detail”两层。大型项目可以是 `D.root → D.1 → D.2 → … → D.n`；每层回答其上层承诺、拥有自己的 reasoning/P，并继续向下展开。
- 不提供 I 模板，不生成代码、配置或实现规约；I 高度依赖具体项目，只作为设计的下游边界被提及。
- SCCO 不是 compose 的主体。compose 只需让 proof 写得可检查，并做最小写作自检；正式召回、攻击、反驳和裁决交给 SCCO 审查 skill。
- 推荐的 Q/D 文件结构、展开层数和回答格式都可被用户指令覆盖；关键是内容类型、论证关系和 trace 正确。
- 默认只给 candidate 草案；写入文件需要明确授权，写入授权不等于用户确认 candidate Q。

### 关键误区

- 不把 QPDI 当完整工程 workflow；它首先是认知与论证结构。
- 不把 D 文档粒度固定成两级；“顶层 / 详细”只是小型项目的常见坍缩。
- 不让 compose 承担完整 SCCO 审查，也不让审查 skill 反过来替代 Q/D 写作。
- 不因设计对 I 有下游影响，就把用户的设计讨论改述成 I 任务或自动扩大修改授权。

---

## `/occams-razor`：压缩已经形成的对象

`/occams-razor` 用于已有对象 `X` 的最小充分压缩：先恢复 `X` 所在的有效目标链与实际语境，再递归删除能够证明无必要贡献的内容。它不是通用 minimalism，也不负责创造目标、补造缺失内容或替用户选择非等价方案。

| 当前需求 | 路由 |
|---|---|
| 已有设计、研究、流程或表达，需要去掉无贡献内容且保持正确与完整 | `/occams-razor` |
| 主要任务是发现遗漏、错误或未覆盖分支 | `/scco-recall`，再由 `/qpdi-tribunal` 等判断 |
| 需要新建或重写 Q/D、选择方案、补足机制 | `/qpdi-compose` 或相应设计流程 |
| 要通过实际评测持续优化一个 prompt | `/prompt-iter` |

使用 `/occams-razor` 时，根层 `X` 通常由用户指定；不得让无关 LLM 历史、已判错或放弃框架、过程便利结构和未获承认的 AI 发明改写它的目标。若无法建立 `X` 的实际需求语境，停止裁剪并提出最小异议。裁剪后必须执行其内置复查循环；无需另起 reviewer 或报告工件。

---

## /prompt-iter：什么时候用它"调提示词"

**情境**：要把一个已存在的提示词（distiller / agent system-prompt / 工具描述 / 任何 prompt）"测着改到更好"，且有 eval 或能跑真 agent 的手段。

- **用 `/prompt-iter`**：目标是**打磨一段已存在的提示词**——数据驱动 bake-off（基线 → 缺口 → 候选 → 确定性度量+多轮抽样 → 回归护栏 → 选赢家/诚实搁置）。固化了"判官单次不可信(sd≈2)、确定性度量破噪音、信息保全增量改不重写、并发不串行、测两层(产物+下游行为)、回归护栏、吓人回归先去噪"这些坑。有条件配 `/pi-consult` 当迭代顾问（**闭环回灌实验结果**再问第二轮）。
- **不是 `/qpdi` / `/workflow`**：那两个是"从零设计+实现一个功能"的开发流程；`/prompt-iter` 只管"一段已有提示词的测试迭代"这一件事。
- **不是 `/pi-consult` 单用**：`/pi-consult` 是"派多模型出思路/复核"，是 `/prompt-iter` 里取候选的一个环节，本身不含度量/对拼/回归护栏。

---

## code-bug-reasoning：讲清/判真假一个【具体 code bug】

`code-bug-reasoning` 是 `principle-derivation-v2` 的**代码场景特化**——姿态沿用 pd-v2，加三个代码专属机制：① 大局先行+逐层定位（每层"大局定位→钉行号→回指大局"）；② **调用链契约传播**（后条件标异常·危险操作前条件标非-UB·击穿点=bug·符号枚举表治误报/漏报）；③ 可达性裁定（链闭合？+ 触发合理可达？→ 真 live/条件/退化/理论/假）。

### 和相邻 skill 的分工

| 场景 | 选 |
|---|---|
| 推导一个**设计/原则/概念**（任何领域）| **principle-derivation-v2**（或 v1，按事后/邀请）|
| 讲清/判真假一个**已锁定的具体 code bug**（机制+可达性）| **code-bug-reasoning** |
| 拿代码**对照 design spec** 查是否满足意图（需 spec、找偏离、可 auto-fix）| **hoare-audit** |
| 一份 GitHub PR 的多方向 review | **workflow-audit** |
| bug 还没定位、大海捞针阶段 | 都不是——先定位；code-bug-reasoning 管"锁定嫌疑段后如何讲清+判真假" |

### 关键判别

- **vs pd-v2**：主体是"一段代码里的一个缺陷"且要**沿数据流传播前后条件找击穿点** → code-bug-reasoning；主体是抽象设计/原则 → pd-v2。code-bug-reasoning **不复述** pd-v2 姿态、只引用。
- **vs hoare-audit**：hoare-audit 是**有 spec 的 code-vs-spec 审计循环**（hunting + findings + fix）；code-bug-reasoning **无需 spec**、针对**单个已疑似缺陷**做"讲清 + 判真假 + 定可达性"。前者找 bug、后者论 bug。
- **双重身份**（写在 skill §六）：它也是**逐函数契约分析（verify-program / FM-Agent / CCodeAnalyze 那类 per-function Pre/Post+SP）缺的那条边——跨函数异常传播 pass——的规格**。逐函数分析"有节点无边"、系统漏跨函数涌现 bug（实证：某 run 逐函数写了 `GetSumOfQuerySeq` 的 Post、也标了内存尺寸依赖，却没连"int32 返回值溢出成负→击穿尺寸检查"这条边，被端到端 PBT 抓到）。要补这条边 → 用 code-bug-reasoning §二当 pass 规格。

### 由来（防"用方法替代思考"）

这个 skill 是从一次真实 bug 分析里**被用户逐轮逼出来的**：先犯"惜墨如金→读者没模型→loophole"，补出"大局先行+逐层定位"；再犯"单操作数脑补→误报 + 不传另一操作数异常→漏一个必要条件"，补出"契约传播+符号枚举"。**所以它的反模式（skill §五）不是想象的、是踩过的**——用时对着 §五自查最省事。

---

## /scco-recall vs /qpdi-tribunal vs /hoare-audit：召回层与判断层

三者不是竞品，是同一审查体系的两层：

- **scco-recall** — 召回层：互盲正交低阈值扇出产**候选池**，只捞不判（finder 有意 spec-盲）；强制含跨文件/消费者角度
- **qpdi-tribunal** — 判断层：counter→judge 对抗式精度 + SCCO 分流（可修 / 需裁决 / 伪问题）；拿 spec 滤"有意决策"类噪音
- **hoare-audit** — 重型判断层：spec 门 + ≥2 disprove-first challenger + 收敛循环 + 每条确认发现钉 PBT

判别：

- 一大坨 diff / 大文档要审、怕漏 → 先 **scco-recall** 再 tribunal / hoare-audit
- 已有候选或单条主张要判真假 → 直接 **tribunal**（单个 code bug → code-bug-reasoning）
- 正确性关键 + 要收敛到不动点 + spec 在手 → **hoare-audit**（可用 scco-recall 当其 Step 2 召回前端）
- **禁**：把 scco-recall 的候选池直接当结论对外报——它未经精度层

由来：一次 PR 审查中，max-recall code-review（10 镜头扇出）在已被 5 轮 hoare-audit 收敛的代码上仍捞出 15 条有效发现，最值钱的两条在**未改动的消费者代码**里；复盘发现其 10 镜头全部塌进 SCCO 四维（约定合规也是——约定即要求，属对规则层的 Sound）。优势不在新维度，在**召回程序 + 找/论分层 + 消费者范围规则**，遂抽成 scco-recall、并把这三样回补进 tribunal。

---

## 记入此文件的触发条件

- 用错了 skill / 用错了 skill 变体（如 v1/v2）发现的经验
- 两个看似可换的 skill 实际有清晰分工的情况
- skill 名字相似但定位不同的情况
- 通用项目工作流里反复证实的"在 X 场景该用 Y skill"判别
