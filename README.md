# code-review-context-research

**AI 代码检视的上下文工程调研**：从 AST/CST 概念到生产落地的完整研究报告。

面向大规模 C/C++ 代码库（宏控、表驱动架构）的 AI 辅助代码检视场景，回答四个问题：

1. 代码分析技术（AST / CST / CFG / ICFG / PDG）的深度阶梯是什么，各层能回答什么？
2. 我们自有的五个工具各处于哪一层，如何分工？
3. 业内（Snap / DoorDash / GitHub / aider / Serena / 开源图谱生态）是怎么做的？
4. 生产怎么落地，效果怎么评测？

## 文档导航

按顺序阅读，每篇独立成文、可单独分享：

| 篇目 | 内容 | 读者 |
|---|---|---|
| [01-概念篇：从 AST 到 PDG](01-概念篇-从AST到PDG.md) | 代码分析技术的深度阶梯：文本 → CST（tree-sitter）→ AST（Clang）→ CFG → ICFG → PDG → Value-Flow，及 Datalog 流派 | 入门可读 |
| [02-实践篇：五工具深度对比与检视工具链](02-实践篇-五工具深度对比与检视工具链.md) | mcp-language-server / codegraph / navmap / vul-llvm / CodeFuse-Query 的源码级勘察、横向对比、生产决策（agent 场景弃用 clangd 索引，codegraph+navmap 替代） | 资深工程师 |
| [03-行业调研篇：代码上下文提取与 AI 检视](03-行业调研篇-代码上下文提取与AI检视.md) | 业内六条路线：零索引 agentic 搜索 / Repo Map / Embedding RAG / LSP-for-agents / 持久化知识图谱 / CPG 静态分析；商业产品与学术线 | 全体 |
| [04-工程实践篇：一线团队 AI 代码检视实战](04-工程实践篇-一线团队AI代码检视实战.md) | Snap CodePal+Code Search、DoorDash 评审 agent+DashBench、Unblocked、GitHub 官方（Blackbird/Copilot）、开源生态实测（gh API）；12 条跨团队共识 | 全体 |
| [05-落地建议篇：生产路线与验收标准](05-落地建议篇-生产路线与验收标准.md) | 目标架构、Phase 0~3 分阶段路线（含出口标准）、验收指标、风险登记册、十条军规 | **决策与执行入口** |
| [06-评测篇：AI 代码检视系统评测方法学](06-评测篇-AI代码检视系统评测方法学.md) | 公开基准全景、基准构建三种方法学（回溯/注入/回放）、指标设计、LLM-as-judge 校准、五层评测方案 | 评测负责人 |

**快速路径**：决策者与落地执行直接读 05；评审或汇报需要证据时按 05 附录的出处索引回溯到 01-04/06。

## 核心结论（TL;DR）

- Clang AST 与 tree-sitter CST 是编译流水线不同阶段的产物：前者回答"是什么"（语义），后者回答"长什么样"（结构）；选型只看三个问题——结论属于哪一层、代码能否编译、控制流是否藏在数据里。
- clangd 全量索引在生产检视场景的根本问题是**失败模式静默**（索引不完整 → 漏报无信号）；codegraph（tree-sitter 预建索引）+ navmap（libclang 领域导航图）组合的失败是显式可见的。
- 表驱动分发（消息分发表/注册式/状态机）是所有通用图工具的盲区，需要领域知识显式化——这是 navmap 的差异化价值，业界几乎无对应物。
- 一线生产系统的共识：上下文质量 > 模型能力；发现与验证分离；精确率优先；精确检索是底座、语义层是插件；评估必须含良性样本且看 trace 不只看分。

## 姊妹项目

本调研解决"**评审时 agent 能看到什么、看多准**"（上下文层）。
配套的 [agent-reviewer](https://github.com/Atituiset/agent-reviewer) 解决
"**评审何时发生、如何强制发生**"（工作流层：SDD 嵌入式评审门禁 + ANDM 团队记忆飞轮）。

两层在 reviewer subagent 的输入处汇合：agent-reviewer 的输入四元组
（diff + spec + 规则 + 团队记忆）覆盖 Intent 与 Conversation，
**Environment（代码结构上下文）由本调研的 codegraph + navmap 方案供给**
（对应 02 篇生产决策与 05 篇 Phase 0~1）。

## 论文库

正文引用的学术文献与行业报告已归档至 [papers/](papers/INDEX.md)：原文 PDF（`papers/pdf/`）+ 逐篇中文解读（`papers/notes/`），索引见 [papers/INDEX.md](papers/INDEX.md)。

## 说明

- 调研时间：2026-08；所有外部论断附来源链接，可逐条核查。
- 文中"我们的工具链"指：[mcp-language-server](https://github.com/Atituiset/mcp-language-server)（fork）、[codegraph](https://github.com/Atituiset/codegraph)（fork）、[navmap](https://github.com/Atituiset/navmap)（自研）、vul-llvm（本地 fork，上游 [thebesttv/vul-llvm](https://github.com/thebesttv/vul-llvm)）、[CodeFuse-Query](https://github.com/Atituiset/CodeFuse-Query)（参照）。
