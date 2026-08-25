# Tracing Technical Debt by AI-Based Correlation Agent（Nokia 案例研究）

| | |
|---|---|
| 作者 | Satu Grönroos, Tuula Hietanen, Anni Salmi, Katja Turtiainen |
| 发表 | Laurea 应用科学大学学士论文，2025-11，46 页 |
| 领域 | 技术债管理 / agentic AI / 行业案例 |
| 本地 PDF | [pdf/theseus-greptile-survey.pdf](pdf/theseus-greptile-survey.pdf) |
| 对应正文 | 03 行业调研篇（作为"Theseus 论文调研"引用于 Greptile 对比行）；04 工程实践篇背景 |

> 注：本仓库正文将其用作 Greptile 等工具的调研出处；论文本体是 Nokia 委托的技术债诊断 agent 概念研究。

## 一句话总结

Nokia 出题、Laurea 团队用 Design Sprint 完成的概念验证：现有工具各自孤立地看代码或文本，缺少把代码结构数据与 commit message、文档等分散文本上下文关联起来的"相关性 agent"——原型 CoCo 验证了对话式人机协同维护工作流的可行性。

## 内容要点

- **问题定义**：技术债根因诊断需要跨数据孤岛——代码分析工具（CodeScene/SonarQube）只看代码，Git 提交信息、原开发者注释、设计文档、需求文档散落各处无人关联。
- **工具对标**：CodeScene（热spots/代码健康）、SonarQube（规则/质量门禁）、**Greptile**（AI 全仓理解与评审）、Powerdrill（数据问答）——结论是没有任何现成工具同时结合代码分析与上下文信息。这是正文 03 篇 Greptile 行的引用来源。
- **Agentic vs Generative**：多步维护工作流（定位根因→关联证据→提出方案→人工确认）需要 agentic AI 的多步推理能力，单轮生成不够。
- **Human-in-the-Loop 工作流**：Correlation Agent 主动评估代码库状态 → 在触点汇报发现与建议摘要 → 用户校验/纠偏 → agent 学习继续。三阶段开发者动作流程：接收与排序 → 验证与计划 → 实施与交互。
- **原型 CoCo**：对话界面 + 相关性分析，结论是对话式交互显著降低调查时间、改善开发协作。

## 与本项目的关系

- 定位是**行业侧写而非学术证据**：它代表大型电信企业对"上下文关联型检视"的真实需求陈述——Intent 与 Conversation 维度（issue、commit message、历史决策）必须与 Environment（代码结构）融合，这正对应本调研与 agent-reviewer 的两层汇合点。
- 其 Human-in-the-Loop 三阶段流程可作为 05 篇 Phase 2+ 中"评审意见分发与人确认"环节的参考模板。
- 方法论上属于概念设计（Design Sprint + 3 次访谈 + 原型 demo），无量化评测，引用时应注明证据等级低于其余各篇。
