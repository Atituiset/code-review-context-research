# AST/CST/CFG 深度调研

两篇报告，入门与进阶分开读：

- **[01-概念篇-从AST到PDG.md](01-概念篇-从AST到PDG.md)** — 代码分析技术的深度阶梯：
  文本 → CST（tree-sitter）→ AST（Clang）→ CFG → ICFG/Call Graph → PDG → Value-Flow，
  以及 Datalog 流派。零编译原理背景可读。
- **[02-实践篇-五工具深度对比与检视工具链.md](02-实践篇-五工具深度对比与检视工具链.md)** —
  基于源码与提交历史勘察的五个项目（mcp-language-server / codegraph / navmap /
  vul-llvm / CodeFuse-Query）对比、分工与集成建议。面向资深工程师。
- **[03-行业调研篇-代码上下文提取与AI检视.md](03-行业调研篇-代码上下文提取与AI检视.md)** —
  业内六条路线（零索引 agentic 搜索 / Repo Map / Embedding RAG / LSP-for-agents /
  持久化代码知识图谱 / CPG 静态分析）+ 商业 AI 评审产品的上下文工程 +
  学术研究线，附全部来源链接。
- **[04-工程实践篇-一线团队AI代码检视实战.md](04-工程实践篇-一线团队AI代码检视实战.md)** —
  生产系统一手复盘：Snap CodePal+Code Search、DoorDash 评审 agent+DashBench、
  Unblocked 组织记忆、国内业务级评审实践、GitHub 官方（Blackbird/Copilot）与
  开源生态扫描（gh API 实测），含 12 条跨团队共识 checklist。
- **[05-落地建议篇-生产路线与验收标准.md](05-落地建议篇-生产路线与验收标准.md)** —
  综合前四篇的最终落地建议：目标架构、Phase 0~3 分阶段路线（含出口标准）、
  验收指标、风险登记册、十条军规。**决策与执行只看这一篇即可。**
- **[06-评测篇-AI代码检视系统评测方法学.md](06-评测篇-AI代码检视系统评测方法学.md)** —
  评测怎么做：公开基准全景（CR-Bench/c-CRAB/Qodo/Martian/Juliet/SWE 系）、
  基准构建三种方法学（回溯/注入/回放）、指标设计（Hit 定义/分层 recall/四信号）、
  LLM-as-judge 校准、映射到本架构的五层评测方案与陷阱清单。
