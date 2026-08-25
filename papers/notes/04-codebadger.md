# codebadger: CPG × LLM 的语义导航

| | |
|---|---|
| 作者 | Ahmed Lekssays（QCRI） |
| 发表 | ACM SVM '26 Workshop（2026-04）；arXiv:2603.24837v1 |
| 领域 | 程序分析 / 漏洞挖掘 / MCP |
| 本地 PDF | [pdf/2603.24837.pdf](pdf/2603.24837.pdf) |
| 开源 | github.com/lekssays/codebadger |
| 对应正文 | 03 行业调研篇（CPG 静态分析路线、codebadger 条目）；02 实践篇（vul-llvm 同类对照） |

## 一句话总结

把 Joern 的 Code Property Graph 引擎用 MCP 封装成高层语义工具（切片、污点追踪、数据流），让 LLM 不必生成 CPGQL 查询就能做跨过程安全分析——在 8000 方法的代码库上找到了 libtiff 未公开缓冲区溢出，并一次生成 libxml2 CVE-2025-6021 的正确补丁。

## 要解决的三个错配

1. token 限制装不下整个仓库（8000 方法 ≈ 数亿 token）；
2. embedding 检索捕捉不了跨过程数据流（"函数 A 的返回值流入函数 B 的 buffer 写"对向量不可见）；
3. LLM 写不出正确的 CPGQL——训练语料里几乎没有这种 DSL，多跳遍历经常幻觉 API。

## 设计

- **会话制架构**：`create_cpg_session` → Joern 生成 CPG（按源码哈希+语言缓存，Docker 隔离，Redis 管状态）→ 后续高层工具调用。
- **核心思想：给工具不给查询语言**。程序切片（backward slicing 可缩减 90% 代码量）、污点流分析（PDG 前向传播 + 路径数上限）、边界检查定位、调用图提取等，都封装为 LLM 可直接调用的语义操作。
- 这与人类分析师的工作方式同构：不线性读文件，而是 summarize → 定位 source/sink → 追 flow → 验证模式。

## 三个用例

1. **GGML 全库内存安全审计**（8667 方法 / 19.8 万调用）：54 个分配点、300+ 危险操作被定位；发现无界 alloca、分配大小整数溢出、未检查指针算术等问题。
2. **libtiff 未披露漏洞**：从 `col_offset` 无验证的指针算术出发反向追踪数据依赖 + 边界检查时序（检查在使用之后，TOCTOU 模式），确认缓冲区溢出并生成 ASAN 复现的 exploit。
3. **libxml2 CVE-2025-6021**：数据流分析 + 切片后一次生成与维护者修复高度一致的正确补丁。

## 与本项目的关系

- 是 vul-llvm 最直接的可比对象：同为"重语义图 + LLM"路线，但 codebadger 走 MCP 服务化 + 高层工具抽象，vul-llvm 更偏 LLVM value-flow 分析。两者共同证明：**发现与验证分离、精确检索做底座**的路线在安全场景成立。
- "不让 LLM 生成 DSL 查询、只暴露语义操作"是一个重要的接口设计原则——navmap/navmap-mcp 的工具面设计可直接借鉴。
- 局限：纯 case study，无量化基准；CPG 构建成本（Joern 解析大仓库的时间）没有报告——这正是 03 篇对比表里 CPG 路线"重量级"的短板。
