# Stack Graphs: Name Resolution at Scale

| | |
|---|---|
| 作者 | Douglas A. Creager, Hendrik van Antwerpen（GitHub） |
| 发表 | EVCS 2023（Eelco Visser 纪念研讨会），arXiv:2211.01224v3 |
| 领域 | 程序分析 / 名称绑定 / 代码导航 |
| 本地 PDF | [pdf/2211.01224.pdf](pdf/2211.01224.pdf) |
| 开源实现 | github/stack-graphs、tree-sitter/tree-sitter-graph |
| 对应正文 | 01 概念篇（CST→名称绑定层）、03 行业调研篇（LSP-for-agents 路线、stack graphs 引用） |

## 一句话总结

GitHub 用 stack graphs 在 forge 规模（数百万仓库、PB 级代码）上做精确代码导航：把名称绑定编码为图中的路径，通过"文件级增量 + 查询时惰性解析 + partial path 预计算"三件套同时保住类型相关查找能力和增量性。

## 要解决的问题

软件 forge 上的代码导航与本地编辑器完全不同量级：

- **规模**：PB 级代码历史、每分钟数千次 push、任意历史版本可查。
- **延迟目标**：查询 <100ms；索引目标是在用户从 git 客户端 Alt-Tab 到浏览器之前完成。
- **语言长尾**：Linguist 收录 500+ 语言，不可能逐个写语义分析器。

现有方案都不满足：LSP server 是交互式 sidecar，无法支撑 forge 的并发与语言矩阵；LSIF 虽可批处理但现存实现不支持文件级增量——新 commit 一来就要全仓重分析。

## 核心方法

### 从 scope graphs 到 stack graphs

1. **Néron scope graphs**：名称绑定 = 图中路径。每个文件是孤立子图（只有 root 节点共享），**天然 file-incremental**——但无法表达类型相关的查找（如 `B().x` 需要"暂停"当前查找去先解析 `B` 的返回类型）。
2. **van Antwerpen scope graphs**：支持类型相关查找，但方式是把中间查找**急切地**固化到图中，产生跨文件的边——彻底破坏了文件增量的可分解性。最坏情况下改一个文件要重建整个程序的图。
3. **Stack graphs（本文）**：关键洞察是维护一个**显式的符号栈**保存挂起的中间查找：
   - 节点四类：root / scope / push symbol (↓x) / pop symbol (↑x)；
   - 遇到 reference/push 就把符号压栈，遇到 definition/pop 要求栈顶匹配并弹出；
   - 路径起点是 reference、终点是 definition、栈空 → 一个完整的名称绑定；
   - 中间查找在**查询时惰性执行**，从而既支持类型相关查找又保留文件增量。

### 工程三板斧

- **file-incremental**：每个文件的子图独立构建、独立存储。git 的 Merkle tree 天然提供内容寻址的 blob id，未变更文件的分析结果直接跳过。与 delta incrementality（依赖追踪+失效重算）对比：后者跨文件边导致同一文件版本要按 commit 存多份，存储成本不可行。
- **partial paths**：把查询时的路径搜索部分搬回索引时。对每个文件预计算所有"文件内或到达 import/export 点"的部分路径（带前置/后置符号栈条件），查询时只需拼接 partial paths。这是论文最重要的性能优化。
- **声明式图构造语言 tree-sitter-graph**：语言专家用 pattern match CST → 图 gadget 的规则描述一种语言的绑定逻辑，每种语言写一次；纯语法分析，无需项目方配置、无需调用不可信的构建过程。

## 结果

自 2021 年 11 月起在生产运行，分析 GitHub 上所有 Python 仓库（公开+私有）的每次提交。核心解析算法语言无关、只实现一次；全部开源，外部语言社区可自助接入。

## 与本项目的关系

- 这是 01 篇"CST 层能回答什么"的最佳实证：纯 tree-sitter 语法层 + 声明式绑定规则，就能做到精确的跨文件名称解析，不需要编译、不需要 clangd 式全量索引。
- 其"失败模式显式"特性（路径找不到就返回空，绝不静默漏报）正对应本调研弃用 clangd 索引的理由；codegraph 的 tree-sitter 预建索引思路与此同源。
- partial paths 的索引时/查询时工作划分，对 navmap 增量更新的设计有直接借鉴价值。
