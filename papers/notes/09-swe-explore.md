# SWE-Explore: 仓库探索能力的行级评测

| | |
|---|---|
| 作者 | Shaoqiu Zhang, Yuhang Wang 等（上海交大等） |
| 发表 | arXiv:2606.07297v1（2026-06） |
| 领域 | coding agent 评测 / 代码定位与检索 |
| 本地 PDF | [pdf/2606.07297.pdf](pdf/2606.07297.pdf) |
| 数据 | github.com/Qiushao-E/SWE-Explore-Bench；HF: SWE-Explore-Bench |
| 对应正文 | 06 评测篇（SWE-Explore 条目）；03 行业调研篇 |

## 一句话总结

把"仓库探索"从端到端解题中剥离出来单独评：给定 issue + 仓库，explorer 返回固定行数预算下的**排序代码区域列表**；ground truth 从多条独立成功轨迹的读取行为中蒸馏。结论：agentic 探索远超传统检索，但**文件级命中已很强、行级覆盖才是瓶颈**。

## 方法

- **数据**：SWE-bench Verified/Pro/Multilingual 中有 ≥2 条强模型（GPT-5.4、Gemini-3-Pro、Sonnet-4.6 等）成功轨迹的实例，共 848 例、10 语言、203 仓库。
- **Ground truth 蒸馏**：收集轨迹中所有可解析为 (文件, 行区间) 的读取动作（view/cat/grep -n 命中），跨轨迹**求交集**得保守核心集，LLM 提炼补充承重区域，作者逐例人工审计。每例平均 4.3 文件 / 4.7 区域 / 1578 行核心上下文。
- **指标族**：行级 Precision/Recall、HitFile/HitRegion、行预算版 nDCG@B、首次有效命中 FUH、上下文效率（预测行落在 core∪opt 的比例）、噪声率。
- **外部效度桥**：一次性验证——只给 explorer 输出的上下文让固定 patcher 解题，证明指标与下游修复率相关：Context Efficiency 相关性最高（Pearson r=0.950），Rec@100 最强秩相关（ρ=0.845）。

## 关键结果

- **agentic explorer 完胜检索基线**：BM25/TF-IDF/Potion 各指标接近随机；所有 agent 类 explorer 大幅领先。多步交互式探索是必要条件，单发词法/向量检索到不了同一量级。
- **瓶颈在行级 recall，不在精度**：通用 agent HitFile ~0.64–0.68、nDCG@500 ~0.87–0.94 都很高，但 Recℓ 只有 0.14–0.19——找到对的文件早，但漏掉大量决定性行区间。
- **换更强的模型不解决瓶颈**：同一 Mini-SWE-Agent 换 6 个 LLM，文件命中始终显著强于行召回——需要的是更好的探索机制而非更大的模型。
- 五个主流 coding agent（Claude Code/Codex/OpenHands/Mini-SWE-Agent/AweAgent）的探索画像惊人相似：复杂 harness 对探索子问题没有本质增益，简单 explorer 接口即可研究此问题。
- CoSIL 精度与行召回最均衡（Prec 0.581 / Recℓ 0.788），但上下文效率偏低（0.544）。

## 与本项目的关系

- 为 codegraph/navmap 这类结构化检索提供了理想的对照评测台：我们的工具输出正是"排序区域列表"，K=5 协议可直接复用来量化 navmap 相对 agentic grep 的增量。
- "行级覆盖是瓶颈"直接支撑 05 篇目标架构中"精确检索是底座"的判断——agent 不缺找文件的能力，缺把证据行一次给全的能力，这正是预建索引的价值区。
- 其 trajectory-grounded 标注法也可反哺我们构建内部评测集。
