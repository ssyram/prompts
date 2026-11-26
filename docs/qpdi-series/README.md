# QPDI Series

本目录记录 `/qpdi` 与 `/qpdi-compose` 为什么存在、当前设计如何成立，以及关键变化由什么实际需求推动。它服务维护、审阅和后续改稿，不复制 prompt 正文，也不构成第二套运行规范。

## 权威边界

| 材料 | 职责 |
|---|---|
| [`current/qpdi.md`](../../current/qpdi.md) | 当前 `/qpdi` 的运行期权威 |
| [`current/qpdi-compose.md`](../../current/qpdi-compose.md) | 当前 `/qpdi-compose` 的运行期权威 |
| [`drafts/qpdi.md`](../../drafts/qpdi.md) | 下一版 `/qpdi` 候选；未获确认前不替代 official |
| [`drafts/qpdi-compose.md`](../../drafts/qpdi-compose.md) | 下一版 Compose 候选；未获确认前不替代 official |
| 本目录 | 需求、设计依据、演变和状态说明 |

若文档与 `current/` 的实际 prompt 冲突，运行行为以 `current/` 为准，同时把冲突视为待修的设计—实现漂移。`evolution/` 只解释形成过程，不能反向取得规范权威。

## 文档

- [`principles.md`](principles.md)：实际需求、使用条件、非目标和成功标准。
- [`architecture.md`](architecture.md)：Q/P/D/I 模型、两份 prompt 的分工、递归设计与交接边界。
- [`evolution/README.md`](evolution/README.md)：从单一 `/qpdi` 到 QPDI Series 的关键压力、裁决和提交锚点。

## 覆盖范围

本系列覆盖：

- `/qpdi` 的静态认知、用户话语拆解、Structural Locality、Discover、re-unification 与 SCCO 基础；
- `/qpdi-compose` 对 Q 和任意深度 D-frame 的写作；
- official 与 draft 的关系；
- 与 SCCO 专项 prompt、skill routing 和 I 的责任边界。

本系列不定义固定执行引擎、角色编排或 FSM，也不把高度定制的 I 写作收进 Compose。

## 当前状态

- official 已包含 Q 的可选整体组织、`D-frame = P + M + R`、设计限制的逐层 guarantee 检查、whole-level integrity 和详细叶子写法。
- draft 以更短的统一模型为骨架，已压缩吸收 whole Q、whole-level Satisfaction、逐层 guarantee narrowing、`expands-to | implemented-by`、I 保持 P/M 和最小 owner 路由。
- draft 仍是候选；是否替换 official 需要内容审阅和代表场景验证，不能仅凭篇幅决定。

## 维护规则

1. 语义变化先指出它解决的真实失败或新需求，再更新相应设计；纯措辞整理无需伪造新需求。
2. 核心定义只保留一个设计说明位置；prompt 中保留运行所需的最小自包含版本。
3. 变化记录写“压力 → 原设计不足 → 新裁决 → 仍成立的边界”，不保存逐字会话和内部执行流水账。
4. official、draft、设计文档和 routing 的职责发生变化时，检查相互引用；未决定的候选明确标为 draft。
5. 不因已有文档而保留无消费者字段，也不以精简为名删除实现或审阅真正需要的语义。
