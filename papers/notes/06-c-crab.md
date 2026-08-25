# c-CRAB: 把人类评审反馈变成可执行测试

| | |
|---|---|
| 作者 | Yuntong Zhang, Zhiyuan Pan 等（NUS / 浙大 / SonarSource） |
| 发表 | arXiv:2603.23448v3（2026-04） |
| 领域 | AI 代码检视评测 |
| 本地 PDF | [pdf/2603.23448.pdf](pdf/2603.23448.pdf) |
| 对应正文 | 06 评测篇（c-CRAB 条目、基准构建"回溯法"代表）；05 落地篇 |

## 一句话总结

评测代码评审不看评论相似度、也不只靠 LLM-as-judge，而是把人类评审意见系统性地转成可执行测试：评审 agent 的意见交给 coding agent 改码，改完能通过测试才算这条评审"抓住了人类提出的问题"。现有全部工具加起来只能解约 40%。

## 为什么相似度指标不行

论文的动机案例极具说服力：

- python-telegram-bot PR 上，人评和 Codex 指出**同一个越界访问问题**但措辞完全不同——BLEU-4 = 0.00、ROUGE-L = 7.02、embedding 相似度仅 54.59。文本相似度系统性低估有效评审。
- posthog PR 上，人评要求把 ArrayField(CharField) 换成 TextField（根治），Claude Code 建议加大 max_length（缓解）——LLM-as-judge 认为两者等价，但只有前者的修改能让 fail-then-pass 的测试通过。**对齐人类意图包含修复方案的正确性**，这是 judge 区分不了的；且 judge 输出有随机性、不可复现。

## 构建流水线（671 PR → 184 实例 / 234 条带测试的评审）

1. **评审过滤**：人工标注 100 条金标集 → 迭代 LLM prompt 分类器，只留"具体、可行动、客观可验证"的意见。
2. **执行环境构建**：自动生成 Dockerfile + 安装脚本；历史 commit 依赖漂移时由隔离环境里的 coding agent 审计并钉死版本。
3. **NL→测试**：给定评审意见 + diff hunk + before/after 两个版本的完整文件，LLM 合成"before 失败、after 通过"的测试；不满足就用执行报错反馈循环修（最多 3 次）。
4. **coding agent 校验**：用人类评审意见指导 Claude Code (Sonnet-4.6) 改码，必须能让测试通过才收进基准——保证评测失败归因于被测评审的质量而非 coding agent 的能力。人工抽检一致性 84%。

测试分两类：behavioral 17.9%（运行代码断言行为）、structural 82.1%（检查源码模式/API 面）。

## 结果

| 评审工具 | 总评论 | Overall pass |
|---|---|---|
| Claude Code | 1336 | **32.1%** |
| Devin | 1344 | 24.8% |
| PR-Agent | 524 | 23.1% |
| Codex | 324 | 20.1% |
| （四工具并集） | — | 41.5% |
| Human（构造性上界） | 234 | 100% |

关键解读：人工抽检显示这些工具的评论 78–88% 本身是有用的——低通过率不是评论质量差，而是 **agent 关注点与人类评审系统性错位**（如 Codex 在 Testing 类 +23.7pp、PR-Agent 在 Design 类 +16.3pp）。这指向人机协作评审而非替代。

## 与本项目的关系

- 06 篇"基准构建三方法学"中**回溯法**的最佳范本：从真实人类反馈出发、以动态程序行为为 oracle，规避了 LLM-as-judge 的校准难题。
- "评审质量 = 能否引导出可通过测试的修复"给了我们一个可移植的验收动作：navmap 上下文增强后的评审意见，可以用同样的 fail-then-pass 协议做 A/B 验证。
- 与 CR-Bench 互补：CR-Bench 以缺陷为中心（judge 打分），c-CRAB 以人类关注点为中心（测试裁决）。
