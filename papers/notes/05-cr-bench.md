# CR-Bench / CR-Evaluator: 评审 agent 的信噪比评测

| | |
|---|---|
| 作者 | Kristen Pereira, Neelabh Sinha 等（Nutanix） |
| 发表 | arXiv:2603.11078v1（2026-03，preprint） |
| 领域 | AI 代码检视评测 |
| 本地 PDF | [pdf/2603.11078.pdf](pdf/2603.11078.pdf) |
| 对应正文 | 06 评测篇（CR-Bench 条目）；04 工程实践篇（精确率优先共识的证据） |

## 一句话总结

把 SWE-Bench 的真实缺陷系统性转成"评审时可发现的 PR 缺陷"基准（584 例，人工核验子集 174 例），并提出 usefulness rate 和 signal-to-noise ratio 两个新指标——揭示评审 agent 的核心权衡：逼它多找 bug 就多产噪声。

## 基准构建

- **转换算法**：对每条 SWE-Bench 实例，git blame 找到引入缺陷行的 commit → GitHub API 反查唯一关联 PR（多 PR 则丢弃）→ LLM 判定"该 bug 在评审时是否可通过逻辑检查/边界测试发现"（不可发现则丢弃）→ 问题陈述改写成评审评论 → 按 category/impact/severity 打标。
- **分类学**：category 用 Beizer 分类（Structural 占 verified 子集 79.9%）；impact 用 ISO/IEC 25010；severity 分低中高（verified 子集 93.1% 为 Medium+）。
- 定位差异：只收**客观缺陷**（功能/性能/可靠性/安全回归），风格类问题明确排除——那应该交给 linter 而不是昂贵的 LLM 推理。

## CR-Evaluator 指标体系

LLM-as-judge 把每条生成评论零样本三分类：Bug Hit / Valid Suggestion / Noise。由此导出：

- Recall = hits / 总 bug 数；Precision = hits / 总评论数；F1。
- **Usefulness Rate** = (hits + valid suggestions) / 总评论数 —— 认可缺陷之外的建设性意见。
- **SNR** = (hits + valid) / noise —— 开发者信任的核心代理；高 recall 靠高音量堆出来的 agent 会在此现形。

## 关键结果（GPT-5.2 / GPT-5-mini，单发 vs Reflexion）

| Agent | Recall | Precision | Usefulness | SNR |
|---|---|---|---|---|
| Single-shot GPT-5.2 | 27.0% | 3.6% | **83.6%** | **5.11** |
| Reflexion GPT-5.2 | **32.8%** | 5.1% | 66.1% | 1.95 |
| Single-shot GPT-5-mini | 18.4% | 3.5% | 74.3% | 2.89 |
| Reflexion GPT-5-mini | 27.6% | 3.2% | 47.7% | 0.91 |

- Reflexion 把 recall 抬约 6pp，但 SNR 从 5.11 砍到 1.95——**小模型在迭代追问下幻觉放大**（SNR 崩到 0.91）。
- 所有配置的 memory 类 bug recall = 0（需运行时 trace）；Usability/Functional Suitability 类偏弱（需要 PR diff 之外的环境知识）——封闭上下文评审的系统性短板。
- 高严重度缺陷 recall 明显更高：agent 是"重大回归的安全网"，不是 nit-picker。

## 与本项目的关系

- 06 篇评测方案学的直接输入：usefulness/SNR 应进入我们的验收指标集；"评估必须含良性样本、看 trace 不只看分"在此得到量化支撑。
- "精确率优先"的代价曲线第一次被画出来：每提升 1pp recall 的噪声成本可以度量——这为 05 篇 Phase 出口标准的阈值设定提供了参照系。
