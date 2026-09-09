# 四路写作理论调研：运行记录

- 工作流：`[运行标识已省略]`
- Mission：`[运行标识已省略]`
- 四路均已完成；完整回执已保存至 `runtime/workflow-receipt.json`。
- 脚本：`workflow.js`，启动前validate通过。
- 四个native `researcher`，fresh context，并发上限4、总spawn上限4；无额外综合/审查代理。
- 隔离：共享cwd、只读研究，输出分别由宿主保存；不要求清理已有脏工作区。
- 任务：#81、#82已完成；#83的综合稿与技能修改方案已写成，来源核读与文件检查完成。

| key | 已完成的run | 实际输出 |
|---|---|---|
| openings | `[运行标识已省略]` | [01-openings.md](01-openings.md) |
| multilingual | `[运行标识已省略]` | [02-multilingual.md](02-multilingual.md) |
| comprehension | `[运行标识已省略]` | [03-comprehension-contract.md](03-comprehension-contract.md) |
| modifiers | `[运行标识已省略]` | [04-modifiers.md](04-modifiers.md) |

回执模型均为 `openai-codex/gpt-6-astra:medium`，均为fresh context。它们是分工查资料，不是四次独立实验或投票验证。使用原生完成通知续接，未轮询等待。

用户随后要求不中止、继续工作。交接不构成等待用户再次确认的门槛；结果返回后直接继续#83，但仍不自动修改技能。

## 准备阶段记录

第一次按文本片段选择三个请求时匹配到四条，断言失败，未写出请求摘录或状态文件。查到末条请求在另一分支有同内容记录，改为以`2881de26`祖先链选择`a12ff0d8`、`60be1695`、`2881de26`，核验后保存。这不是subagent启动失败。

`runtime/launch-state.json`保存repo/cwd/branch/HEAD、启动时git status与四个受保护文件hash。启动前检查通过：请求归属、Markdown围栏、工作流大小、受保护文件未改变。

## 首轮综合与验证（#83）

- 父代理读取四路报告，并按原始URL核读10项关键来源的相关内容；具体范围见 [SYNTHESIS.md](SYNTHESIS.md)。
- 子进程fetch缓存ID在主会话返回 `No stored results`；用同一web工具体系按已有URL重取，没有新检索、补派代理或CLI切换。
- 保存 [综合报告](SYNTHESIS.md)、[技能修改方案](SKILL-REVISION.md) 与 [目录](README.md)。未实际修改技能。
- 文档检查包括内部链接、来源编号、围栏和空白；这不是阅读效果实验，也不是自动来源核验通过。
- 四个原研究报告hash与四个受保护文件hash均保持不变。检查记录保存于 `runtime/verification.json`。该记录描述#83当时的版本，不用于证明下述新稿已通过检查。

## 后续批判与三稿修订（#84–#85）

- #84结合已读Occam材料检查任务、方法、推荐和例子的关系；一次只读oracle协助提出批判，工作流为 `[运行标识已省略]`。它不是新一轮写作研究，也不是读者效果实验。
- #85根据用户授权重写综合稿和修改方案，新增完整技能候选。改前五份文件与十个保护文件的hash保存在 before.json（原始本地运行记录，未随归档提供）。
- 首个工作流 `[运行标识已省略]` 因 `SyntaxError: Unterminated string constant (2:211)` 失败，未启动writer。已核对原稿、受保护文件和原有脏工作区状态没有变化；失败现场（原始本地运行记录，未随归档提供）、回执（原始本地运行记录，未随归档提供）及空局部diff均保留。
- 将长任务说明移入 [BRIEF.md](runtime/rewrite-85-993f3496/BRIEF.md)，工作流脚本（原始本地运行记录，未随归档提供）通过validate后，以相同原生协议重试。成功工作流 `[运行标识已省略]`，唯一写作者 `[运行标识已省略]`；成功回执（原始本地运行记录，未随归档提供）。没有切换CLI或前台代理。
- 父代理完整阅读三稿，修正无依据的总优先序、将未知主张自行降为目标、歧义例擅定解释等问题，并补足成文示范。初稿保存在 `runtime/rewrite-85-993f3496/writer-version/`。
- 新交付为 [SYNTHESIS.md](SYNTHESIS.md)、[SKILL-REVISION.md](SKILL-REVISION.md)、[SKILL-DRAFT.md](SKILL-DRAFT.md)。同步更新目录与交接顶部，未改原研究报告、原稿、旧历史总报告、已安装技能及参考文件。
- [本次核对与给定材料试写](runtime/rewrite-85-993f3496/CHECKS.md)记录人工核对及证据边界；本次机器检查与成稿hash保存于 `runtime/rewrite-85-993f3496/verification.json`。不以这些检查声称真实指定读者阅读效果或技能独立增益。

当前待用户审阅候选、决定是否替换实际技能；本轮不继续调研或增加审查门槛。
