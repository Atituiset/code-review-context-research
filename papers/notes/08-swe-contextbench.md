# SWE-ContextBench: 编码 agent 的上下文学习评测

| | |
|---|---|
| 作者 | Jiayuan Zhu, Junde Wu 等（Oxford / Edinburgh / TUM / NUS 等） |
| 发表 | arXiv:2602.08316v3（2026-05） |
| 领域 | coding agent 评测 / 上下文与记忆复用 |
| 本地 PDF | [pdf/2602.08316.pdf](pdf/2602.08316.pdf) |
| 对应正文 | 06 评测篇（SWE-ContextBench 条目）；03 行业调研篇 |

## 一句话总结

现有基准把每个任务当独立事件，测不出"agent 能否复用既往经验"。SWE-ContextBench 通过 issue/PR 的真实引用关系构造 1,476 个任务（1,100 经验 + 376 相关，51 仓库、9 语言），显式度量上下文复用对准确率、耗时、token 成本的影响。

## 基准构造

- 底座：SWE-Bench Lite / Multilingual / Verified。
- **经验任务池**：跑 Claude Code (Sonnet 4.5) 记录完整求解轨迹（工具调用、文件导航、推理步骤），存为可检索的上下文池。
- **相关任务发现**：人工核查六类引用关系——多 issue 同 PR 解、PR→issue 引用、PR→PR 引用、issue→issue、issue→PR、多重引用；再做一轮递归扩展捕捉二阶依赖，共得 376 个相关任务。
- 另发布 Lite 版（300+99 任务，仅 Python）降低评测成本。
- 指标三维：**准确率变化**（相对无上下文基线）、**时间效率**、**成本效率**（token 为代理）。测试验证沿用 FAIL_TO_PASS + PASS_TO_PASS 双集合。

## 关键结果

- Oracle summary（给对的那份 ~217 token 摘要）显著提升多数 agent：GPT-5.3 Codex 解题率 22.60% → 23.94%，Claude Sonnet 4.5 → 23.40%；难任务上最慢实例 runtime 降超 60%。
- **自由检索几乎无收益甚至负收益**：Free Summary 比 Oracle Summary 低 12.12pp，而 Free Context 只比 Oracle Context 低 1.01pp——完整轨迹冗长但容错高，摘要精炼但"喂错就带偏"。
- 记忆框架对比：Supermemory 最优（resolved 30.30%），Mem0 垫底（24.24%）；LangMem 检索命中率最高（73.34%）但端到端未披露同水平优势——**检索命中率与最终收益不成正比**。
- 自由检索的 top-1 命中率仅 18–36%：当前 agent 的"何时信任检索结果"能力是瓶颈。

## 与本项目的关系

- 给 05 篇 Phase 路线里的"团队记忆飞轮"（姊妹项目 agent-reviewer 的 ANDM 层）提供了量化依据：正确选择的先验上下文能同时买来准确率和效率，错误选择则是负资产——**检索之后的信任/适配机制必须与检索本身同等设计**。
- "摘要轨迹 vs 完整轨迹"的权衡结论可直接用于 reviewer subagent 记忆条目的粒度决策。
