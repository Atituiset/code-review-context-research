# Codebase-Memory: Tree-Sitter 知识图谱 + MCP

| | |
|---|---|
| 作者 | Martin Vogel, Falk Meyer-Eschenbach 等（Charité 柏林等） |
| 发表 | arXiv:2603.27277v1（2026-03） |
| 领域 | LLM 代码检索 / 知识图谱 / MCP |
| 本地 PDF | [pdf/2603.27277.pdf](pdf/2603.27277.pdf) |
| 对应正文 | 03 行业调研篇（持久化知识图谱路线，Codebase-Memory 条目） |

## 一句话总结

用 tree-sitter 把代码库解析成持久化知识图谱存进单个 SQLite 文件，通过 MCP 暴露 14 个结构化查询工具给任意 agent：以 90% 的回答质量换取 1/10 的 token 和 1/2.1 的工具调用次数。

## 核心设计

- **三段流水线**：Parse（tree-sitter，66 种语言；Go/C/C++ 额外做 LSP 式类型解析提升调用图精度）→ Build（6 相位、pthreads 并行、内存图缓冲后批量刷入 SQLite）→ Serve（MCP server，亚毫秒查询）。
- **图谱 schema**：函数/类/文件/路由等节点；CALLS/IMPORTS/INHERITS/TESTS/HTTP_CALLS 等边，甚至通过跨服务 HTTP 路由匹配表达微服务架构。
- **调用解析**：6 策略级联（import map → 同模块 → 唯一名 → 后缀 → 模糊），带置信度打分；策略 1–3 解决约 80% 调用。
- **增量同步**：file watcher + XXH3 内容哈希，只重析变更文件。
- **部署形态**：单静态链接 C 二进制、零运行时依赖——与 CodeQL/CPG 类"重量级"方案的核心差异点。

## 评测结论

31 种语言 × 12 类问题，与 grep+读文件的 Explorer agent 对比（同为 Claude Opus 4.6）：

| 指标 | MCP Agent | Explorer Agent |
|---|---|---|
| 质量分 | 0.83 | 0.92 |
| 工具调用/题 | 2.3 | 4.8 |
| token/题 | ~1,000 | ~10,000 |
| 查询延迟 | <1ms | 10–30s |

关键细分：

- 图方法在**枢纽检测、调用者排名、依赖链**上占优（19/31 语言）；函数式语言差距缩到 ~1%。
- Explorer 在**全文上下文和穷尽式 call-site grep** 上仍占优（16/31）——图刻意不存源码行。
- **最弱场景是宏密集的 C 语言（0.58 vs 1.00）：宏不在 AST 里**。这直接印证了本调研"表驱动/宏控架构是通用工具盲区"的判断。

规模上限：Linux 内核（2800 万行）建图 2.1M 节点 / 4.9M 边约 3 分钟；增量重索引提速约 4×。

## 与本项目的关系

- 它就是 03 篇"持久化知识图谱"路线的学术化样本，且与本仓库 codegraph + navmap 组合高度同构：tree-sitter 结构索引 + 领域增强解析（其对 C/C++ 的混合 type-resolution pass 尤其可对照 navmap）。
- "graph for structural queries, file exploration fallback for source-level tasks"的混合结论，可直接引用为 Phase 0~1 架构的外部证据。
- 其安全章节（8 层 CI 审计、供应链防护）对自研 MCP server 的发布流程有参考价值，但属于工程外围。
- 局限：评测由第一作者人工判卷、无第三方复现；宏盲区问题它自己承认但没解决——这正是 navmap 差异化价值的又一佐证。
