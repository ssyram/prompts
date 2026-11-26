# skill-maker：参考基线与分析环境

**现在直接采用 ZW 提供的 prompt-base，不等待完整 Q 或测试框架，也不先另造一套 maker。**

当前工作目录：`~/workspace/ai-tools/prompts/`。原来的 auth-survey 目录只作为源材料来源；本项目不恢复那条工作线。

## 立即使用

[可直接加载的原版基线](baseline/schema-closure-prompt-generator/SKILL.md)与[上游原文副本](zw-reference/general-prompt-base.md)字节一致。保留上游名称 `schema-closure-prompt-generator`；`skill-maker` 是本项目的名称，不是假定已注册的命令。

在 Pi 中可以仅为一次启动加载它，不改全局配置：

```bash
cd ~/workspace/ai-tools/prompts
pi --skill docs/skill-maker/baseline/schema-closure-prompt-generator/SKILL.md
```

随后在会话中输入：

```text
/skill:schema-closure-prompt-generator
```

再说明要生成或调整哪个领域的提示词，提供现有文本、实际任务、明确反馈和仍有效的要求。原版从 Phase 1.1 的角色探询开始，依次要求确认画像、诊断和映射，再生成与自检；这里没有替你提前做这些判断。

也可直接让已有会话读取上述 SKILL.md 全文，并明确要求按该技能处理你的下一条需求。只提文件名不等于已加载正文。

本次只准备文件和使用说明，未启动这套生成流程，未运行效果实验。以上加载方式依据本机 Pi `docs/skills.md`；没有声称已经验证模型会遵守它。

## 从哪里开始研读

1. 先读原文 `Role`：它把生成器定位为“把领域需求转为可执行约束”，而非提供成品套用模板。
2. 顺着 `Workflow` 看输入如何经过探询、诊断、映射、生成、自检；哪些判断由你确认。
3. 再看七层方法论怎样落实到具体约束、输出格式和检查项。原文的“不提供填空式模板”针对生成器套用成品模板，并不取消为具体任务定制结构化输出。

共同研读和综合由用户主导。助手负责材料、问题定位、版本对照和后续测试，不把本次直接采用写成方法已被证明有效。

## 当前方向与后续顺序

目标是让合作的 AI 实际注意并执行任务要求。先处理漏做、不执行，再区分推理质量和结构质量。过程中的结构化填写用于把对应判断纳入注意，不要求最终交付物一律变成表格。

参考材料与共同理解 → 共同打磨 maker → 由 maker 生成 Occam、reflect-and-proceed、recalibration 等修订文本 → 用生成文本执行案例 → 分别检查是否执行、结果质量及失败环节。

案例准备可以并行，但不是使用当前基线的前置门槛。后续生成的原始版本与人工改过的版本必须区分；后来的用户反馈留作评价，不能提前塞给被测模型。

## 已准备的材料

- [ZW 参考与研读入口](zw-reference/README.md)：八条建议原文、上游生成器、反思改写稿与原稿对照。
- [问题意识草案](principles.md)：区分目标、工作前提和经验；是供你核对的候选，不是已定案 Q。
- [三组案例环境](cases/README.md)：三份基线、四个迁移用例，输入与评价分开，保存原始消息定位。

这批案例是已知反馈的回归/迁移材料，不是独立、未见样本的效果证明。下一步可以直接读或用原版，再共同分析怎样调整。安装、发布和效果实验不在本轮范围内。

## 来源与进度

- [上游 GitHub 文件](https://github.com/Gnc0/prompts/blob/tool/prompt-is-winning-so-much/general-prompt/general-prompt-base.md)
- [固定版本与校验信息](zw-reference/prompt-base-source.json)：分支 `tool`，提交 `c1a5ba7efb4f718663f5781445a5808bcffd2aa2`，取回日期 2026-09-11。
- 原文与采用版均为 27,663 字节，SHA-256：`3cd9ef15775f871956e448a09253af0a7bb10a6c5c9d5262271d802a2154093d`。
- 全局进度维护在 `~/MY-TODO.md`，不另建 auth-survey 局部 TODO 作为主入口。
