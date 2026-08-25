# 论文索引（Papers Index）

本目录收录调研正文（01~06 篇）引用的学术文献与行业报告：`pdf/` 为原文，`notes/` 为逐篇中文解读。每篇解读含：一句话总结、核心方法/发现、与本项目六篇正文的对应关系。

## 按正文篇目索引

### 01-概念篇 / 02-实践篇（技术阶梯与工具链）

| 解读 | 论文 | 发表 | 本地 PDF | 关联工具 |
|---|---|---|---|---|
| [01 Stack Graphs](notes/01-stack-graphs.md) | Stack Graphs: Name Resolution at Scale (Creager & van Antwerpen) | EVCS 2023, arXiv:2211.01224 | [pdf](pdf/2211.01224.pdf) | codegraph（tree-sitter 路线源头）、对照 clangd/LSIF |
| [04 codebadger](notes/04-codebadger.md) | Bridging Code Property Graphs and Language Models for Program Analysis (Lekssays) | ACM SVM '26, arXiv:2603.24837 | [pdf](pdf/2603.24837.pdf) | vul-llvm 的 MCP 服务化同类 |
| [02 Codebase-Memory](notes/02-codebase-memory.md) | Codebase-Memory: Tree-Sitter-Based Knowledge Graphs for LLM Code Exploration via MCP (Vogel et al.) | arXiv:2603.27277 | [pdf](pdf/2603.27277.pdf) | codegraph + navmap 组合的学术同构体 |

### 03-行业调研篇（检索路线）

| 解读 | 论文 | 发表 | 本地 PDF | 路线归属 |
|---|---|---|---|---|
| [03 Better Call Grep](notes/03-better-call-grep.md) | Better Call Grep: Evaluating and Improving Grep-Like Lexical Retrieval (Wang et al., 浙大) | ISSTA 2026, arXiv:2601.23254 | [pdf](pdf/2601.23254.pdf) | 零索引 agentic 搜索 |
| 01 Stack Graphs（同上） | — | — | — | LSP-for-agents / stack graphs |
| 02 Codebase-Memory（同上） | — | — | — | 持久化知识图谱 |
| 04 codebadger（同上） | — | — | — | CPG 静态分析 |
| [11 Nokia 技术债 agent](notes/11-nokia-correlation-agent.md) | Tracing Technical Debt by AI-Based Correlation Agent — A Case Study of Nokia (Grönroos et al.) | Laurea 学位论文 2025 | [pdf](pdf/theseus-greptile-survey.pdf) | 行业侧写（Greptile 对标出处；证据等级：概念设计） |

> 注：正文中引用的 preprints.org 手稿（202510.0924）下载返回 HTML、无法获取原文，未收录。

### 06-评测篇（基准与方法学）

| 解读 | 论文 | 发表 | 本地 PDF | 方法学定位 |
|---|---|---|---|---|
| [10 NIST SATE V](notes/10-nist-sate-v.md) | SATE V Report: Ten Years of Static Analysis Tool Expositions (NIST SP 500-326) | NIST 2018 | [pdf](pdf/nist-sp500-326.pdf) | 静态分析评测方法学鼻祖；回溯法原型 + 评测边界告诫清单 |
| [05 CR-Bench](notes/05-cr-bench.md) | CR-Bench: Evaluating the Real-World Utility of AI Code Review Agents (Pereira et al., Nutanix) | arXiv:2603.11078 | [pdf](pdf/2603.11078.pdf) | 缺陷中心 + LLM-as-judge；提出 Usefulness/SNR 指标 |
| [06 c-CRAB](notes/06-c-crab.md) | Code Review Agent Benchmark (Zhang et al., NUS/SonarSource) | arXiv:2603.23448 | [pdf](pdf/2603.23448.pdf) | 人类反馈 → 可执行测试裁决；现有工具仅解 ~40% |
| [07 CodeFuse-CR-Bench](notes/07-codefuse-cr-bench.md) | CodeFuse-CR-Bench: A Comprehensiveness-aware Benchmark (Guo et al., 蚂蚁) | arXiv:2509.14856 | [pdf](pdf/2509.14856.pdf) | 仓库级端到端 + 完整性感知；601 例 Python |
| [08 SWE-ContextBench](notes/08-swe-contextbench.md) | SWE Context Bench: A Benchmark for Context Learning in Coding (Zhu et al.) | arXiv:2602.08316 | [pdf](pdf/2602.08316.pdf) | 上下文复用/记忆检索评测；正确摘要↑、错误检索↓ |
| [09 SWE-Explore](notes/09-swe-explore.md) | SWE-Explore: Benchmarking How Coding Agents Explore Repositories (Zhang et al., 上交) | arXiv:2606.07297 | [pdf](pdf/2606.07297.pdf) | 行级探索能力隔离评测；agentic≫检索，行级召回是瓶颈 |

## 按主题速查

**结构化代码表示**：01（名称绑定图）、02（tree-sitter KG）、04（CPG）
**检索路线对比**：02 vs 03 vs 09（图 vs grep vs agentic 探索的三角对照）
**评审质量指标**：05（SNR/Usefulness）、06（测试裁决）、07（完整性感知）
**宏控/盲区佐证**：02（宏密集 C 最弱项 0.58）、03（隐式依赖天花板）
**增量索引设计**：01（file-incremental + partial paths）、02（XXH3 增量重索引）

## 阅读建议

- 只读三篇先看：**01**（为什么 tree-sitter 层能做精确导航）、**09**（探索瓶颈在哪）、**06**（评审怎么评才可信）。
- 做落地决策补 **05**（指标体系）；做 navmap 差异化论证补 **02+03** 的盲区结论。
