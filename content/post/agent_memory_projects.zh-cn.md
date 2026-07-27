---
title: "Agent Memory 开源项目调研报告"
date: 2026-07-27T07:49:30+08:00
draft: false
tags: [ai, agent]
categories: [tech]
---

![为什么 Agent 需要记忆](https://cdn.mahaoliang.tech/2026/20260727080218337.png)

随着 AI Agent 从单轮对话走向长周期、多会话、多协作的复杂场景，"记忆"是制约 Agent 智能化水平的关键瓶颈。2026 年，Agent Memory 已发展为一个独立的技术领域，涌现出多个开源项目。本报告针对以下 5 个项目进行深度调研，并横向对比业界知名方案。

调研范围：[OpenViking（字节跳动/火山引擎）](https://github.com/volcengine/OpenViking)、[TencentDB-Agent-Memory（腾讯云）](https://github.com/TencentCloud/TencentDB-Agent-Memory)、[MemOS（MemTensor）](https://github.com/MemTensor/MemOS)、[hebb-mind（afx-team）](https://github.com/afx-team/hebb-mind)、[PowerMem（OceanBase）](https://github.com/oceanbase/powermem)。

---

## 项目逐一分析

### [OpenViking](https://github.com/volcengine/OpenViking)

![OpenViking](https://cdn.mahaoliang.tech/2026/20260727080939146.png)

**定位**：面向 AI Agent 的"上下文数据库"（Context Database），由字节跳动火山引擎开源。

**核心理念**：摒弃传统 RAG 的扁平向量存储，采用"文件系统范式"统一管理 Agent 所需的记忆（Memory）、资源（Resources）和技能（Skills）。开发者像管理本地文件一样构建 Agent 的大脑。

**关键架构特性**：

OpenViking 最突出的设计是 L0/L1/L2 三层分级上下文加载机制。L0 是抽象层（极简摘要），L1 是概览层，L2 是完整细节层。Agent 按需加载，仅在需要时深入下层，从而大幅节省 token 开销。实测数据显示，在长程对话任务中，相比 LanceDB 基线，任务完成率提升 15-17%，输入 token 成本降低 92-96%。

检索方面，OpenViking 采用"目录递归检索"——先通过目录层级定位，再结合语义搜索，实现比扁平向量数据库更精准的上下文获取。同时提供检索轨迹可视化，开发者可以清晰观察 Agent 检索了什么、为什么选择这些内容，解决了传统 RAG 的"黑盒"问题。

会话管理上，OpenViking 支持自动会话压缩和长期记忆提取，每次会话结束后自动将重要信息沉淀到用户记忆和 Agent 记忆目录中，实现"越用越聪明"。

**技术栈**：Python + Rust（CLI 和核心组件），支持 Volcengine Doubao、OpenAI、Codex、Kimi、GLM 等多种 VLM 提供商，Embedding 支持火山引擎、OpenAI、Azure、Jina、Ollama、Voyage、DashScope 等十余种。

**生态集成**：提供 OpenClaw 插件、MCP Server、CLI 工具、Python SDK，以及桌面端 OpenViking Helper（macOS/Windows）。

**评价**：这是目前 Star 数最高的 Agent Memory 项目之一。其"文件系统范式"和分层加载设计在理念上非常先进，直接对标了传统 RAG 的核心问题。字节跳动的工程能力保证了项目的成熟度和可维护性。不过其复杂度也较高，部署需要 Python 3.10+、Rust 工具链和 C++ 编译器，对轻量级场景有一定门槛。

---

### [TencentDB-Agent-Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory)

![TencentDB-Agent-Memory](https://cdn.mahaoliang.tech/2026/20260727081125379.png)

**定位**：腾讯云开源的 Agent 记忆插件，口号是"Agents remember, Humans innovate"。

**核心理念**：拒绝扁平存储，拥抱分层与符号化。其架构建立在两个支柱上——"符号化短期记忆"和"分层长期记忆"。

**关键架构特性**：

短期记忆方面，TencentDB-Agent-Memory 采用 Mermaid 符号图来压缩任务状态。传统做法中，工具调用日志动辄占据数万 token，而该项目将冗长的日志卸载到外部文件（refs/*.md），只保留轻量级的 Mermaid 任务图在上下文中。Agent 在符号图上推理，需要验证细节时通过 node_id 回溯原始文本。这种方式在 SWE-bench 测试中将 token 使用量降低了 33%，在 WideSearch 任务中降低了 61.38%，同时任务通过率分别提升了 9.93% 和 51.52%。

长期记忆方面，采用语义金字塔结构：L0 Conversation（原始对话）→ L1 Atom（原子事实）→ L2 Scenario（场景块）→ L3 Persona（用户画像）。Persona 层承载日常偏好，仅在需要细节时下钻到 Atom 层。底层数据持久化在数据库中支持全文检索，上层以人可读的 Markdown 文件存储，保证高信息密度和白盒可检查性。

全链路可追溯是其另一亮点：从高层抽象到底层证据有确定性的回溯路径，避免了不可逆的有损压缩。

**技术栈**：TypeScript/Node.js，默认使用 SQLite + sqlite-vec 后端，零配置即可运行。支持 OpenClaw 和 Hermes Agent 两种集成方式。

**评价**：该项目在"短期记忆压缩"上有独到创新，Mermaid 符号图是一个非常聪明的设计——既保留了结构化信息，又极大压缩了 token。分层长期记忆的设计也比较成熟。但项目 Commits 仅 100，说明还处于相对早期阶段，社区生态和长期维护能力有待观察。

---

### [MemOS](https://github.com/MemTensor/MemOS)

![MemOS](https://cdn.mahaoliang.tech/2026/20260727081256629.png)

**定位**：面向 LLM 和 AI Agent 的"记忆操作系统"（Memory Operating System），由 MemTensor 团队开发。

**核心理念**：将记忆视为操作系统的核心功能，统一 store/retrieve/manage，使 Agent 具备上下文感知和个性化能力。

**关键架构特性**：

MemOS 提供统一的 Memory API，支持添加、检索、编辑和删除记忆，记忆结构化为图谱（Graph），可查看、可编辑，而非黑盒向量存储。原生支持多模态记忆——文本、图片、工具调用轨迹和用户画像可在同一记忆系统中被检索和推理。

Multi-Cube 知识库管理是其特色之一：可将多个知识库作为可组合的"记忆立方体"管理，支持隔离、受控共享和跨用户/项目/Agent 的动态组合。这对于多租户和多 Agent 协作场景非常有价值。

MemScheduler 实现异步记忆摄取，毫秒级延迟，保证高并发下的生产稳定性。记忆反馈与修正功能允许用户用自然语言修正、补充或替换已有记忆，形成持续演化的记忆体系。

在本地插件方面，memos-local-plugin 2.0 支持四层自演化记忆：L1 traces（执行轨迹）→ L2 policies（策略规则）→ L3 world models（世界模型）→ crystallized Skills（技能），全部本地存储，零云端依赖。

**Benchmark 表现**：LoCoMo 88.83，LongMemEval 89.20，PersonaMem v2 40.58，HaluMem 80.91，SWE-Bench 38.46。在 OmniMemEval（涵盖 14 个商业记忆产品的统一评测）中处于领先位置。

**技术栈**：Python（34.7%）+ TypeScript（58.3%），自托管依赖 Neo4j + Qdrant，本地插件使用 SQLite。支持 Docker 一键部署。

**评价**：MemOS 是目前理念最"宏大"的项目——真正在做"记忆操作系统"而不仅仅是一个记忆插件。多模态支持、Multi-Cube 隔离、四层自演化等设计都展现了学术深度。1969 次 Commits 和 10.2K Stars 说明项目有持续投入和社区认可。不过其部署复杂度也最高（Neo4j + Qdrant），学习曲线较陡。

---

### [hebb-mind](https://github.com/afx-team/hebb-mind)

![hebb-mind](https://cdn.mahaoliang.tech/2026/20260727081408865.png)

**定位**：受神经科学启发的 Agent 记忆框架，名称来源于心理学家 Donald Hebb 的"赫布学习规则"——"一起激活的神经元会连接在一起"。

**核心理念**：模拟人脑的记忆生命周期——编码（Encode）→ 回放（Replay）→ 巩固（Consolidate）→ 遗忘（Forget）。

**关键架构特性**：

hebb-mind 的记忆循环分为四个阶段，精确对应神经科学原理。编码阶段模拟海马体 CA1 区，新记忆进入工作记忆收件箱（mem_hippocampus）。回放与巩固阶段模拟慢波睡眠中的尖波涟漪，Agent 将记忆分类到对应分区、解决冲突、在知识图谱中写入标签，默认为每日 18:00 自动执行。检索阶段模拟 CA3 的模式补全，采用三路混合检索（向量 + 关键词 + 图谱），综合考虑时效性、重要性和相关性进行评分。遗忘阶段模拟突触修剪和艾宾浩斯遗忘曲线，基于访问次数和重要性设置动态 TTL，被忽视的记忆逐渐淡出。

标签知识图谱（Tag Knowledge Graph）是其核心数据结构：共同出现的标签之间建立连接，共现次数越多连接越强。检索时沿连接遍历，部分线索即可补全整个模式——这正是赫布学习规则的计算实现。

零外部依赖是其重要卖点：`pipx install hebb-mind` 一条命令即可运行，SQLite 存储、sentence-transformers 本地嵌入、NetworkX 图谱，仅在使用 LLM 巩固功能时需要 API Key。

**Benchmark 表现亮眼**：LoCoMo R@10 达 95.75%（使用 bge-large-1024 + reranker），LongMemEval R@10 达 99.4%，MemBench 总体 Hit@5 达 94.6%，在困难类别（noisy、conditional、post_processing）上大幅领先 MemPalace。

**生态集成**：支持 Claude Code hooks、Codex 集成、Agent Sync（同步 Claude Code 和 Codex 的历史会话）、MCP Server、REST API、Python SDK，以及内置 Web Console。

**评价**：这是一个"小而美"的项目，Star 数不高但技术创新性极强。神经科学隐喻不是噱头——遗忘机制、动态 TTL、巩固回放等设计确实解决了许多项目忽略的"记忆老化"问题。Benchmark 成绩非常出色，尤其是 LoCoMo 95.75% 和 LongMemEval 99.4% 的检索准确率。零依赖的极简部署也是显著优势。但项目 Star 数仅 41，社区活跃度和长期可持续性存疑。

---

### [PowerMem](https://github.com/oceanbase/powermem)

![PowerMem](https://cdn.mahaoliang.tech/2026/20260727081508266.png)

**定位**：OceanBase 团队开源的 AI 记忆插件，口号"Accurate, Agile, Affordable"。

**核心理念**：将对话、行为和反馈转化为结构化的长期记忆，可搜索、可更新、可衰减、可跨 Agent 共享。

**关键架构特性**：

PowerMem 最核心的创新是"两层经验 + 技能蒸馏"（Experience + Skill Distillation）。Experience 层记录具体执行经验，Skill 层从多次经验中蒸馏出可复用的工作流和 SOP。`memory.distill_all()` 一个 API 调用即可触发全量蒸馏。这种自演化机制让 Agent 不仅记住事实，还能学会做事的方法。

混合检索开箱即用：向量检索、全文检索、图谱检索和时效性信号在统一的记忆层中协同工作，无需额外拼接。艾宾浩斯遗忘曲线机制为每条记忆设置基于访问频率和重要性的衰减策略，保持记忆库的活力。

多 Agent 隔离方面，PowerMem 支持 scope 机制，不同 Agent 可以有独立的记忆空间，也可以选择共享。用户画像功能支持自动从交互中提取和更新用户偏好。多模态支持文本、图片和音频。

**Benchmark 表现**：LOCOMO 准确率 87.79%（相比基线 52.9% 提升 65.9%），搜索 p95 延迟 1.44s（基线 17.12s，降低 91.6%），token 消耗从 26K 降至 ~0.9K（降低 96.5%）。AppWorld 任务通过率从 24% 提升至 39%（提升 62.5%）。

**技术栈**：Python，存储支持 OceanBase（原生混合检索）、嵌入式 seekdb、PostgreSQL/pgvector、SQLite。LLM 支持 Anthropic、OpenAI、Azure、Gemini、Qwen、DeepSeek、Ollama 等。提供 Python SDK、HTTP Server、MCP Server、CLI（pmem）、Web Dashboard，以及面向 Claude Code、Cursor、VS Code、Codex、Windsurf、GitHub Copilot、OpenClaw 等主流工具的一键集成。

**评价**：PowerMem 在工程完成度上令人印象深刻——从 SDK 到 CLI 到 Dashboard 到各种 IDE 插件，集成面非常广。OceanBase 作为分布式数据库的底层能力为存储层提供了可靠性保障。两层蒸馏机制是差异化亮点。不过项目 Star 数 757 相对偏少，可能是 OceanBase 团队在 Agent Memory 领域知名度还不够的原因，而非项目质量问题。

---

## 横向对比

| 维度                  | OpenViking            | TencentDB-Agent-Memory | MemOS                   | hebb-mind                    | PowerMem               |
| --------------------- | --------------------- | ---------------------- | ----------------------- | ---------------------------- | ---------------------- |
| Stars                 | 26.8K                 | 9K                     | 10.2K                   | 41                           | 757                    |
| 背后的组织            | 字节跳动/火山引擎     | 腾讯云                 | MemTensor               | afx-team（独立）             | OceanBase（蚂蚁集团）  |
| 核心隐喻              | 文件系统              | 分层金字塔             | 操作系统                | 神经科学（赫布）             | 经验蒸馏               |
| 记忆分层              | L0/L1/L2              | L0-L3 + Mermaid 符号图  | L1-L3 + Skill           | 编码→回放→巩固→遗忘          | Experience + Skill     |
| 检索方式              | 目录递归 + 语义         | 混合（RRF 融合）        | 图谱 + 混合               | 三路混合（向量 + 关键词 + 图谱） | 向量 + 全文 + 图谱 + 时效    |
| 遗忘机制              | 无显式设计            | 无显式设计             | 无显式设计              | 动态 TTL（艾宾浩斯）          | 艾宾浩斯衰减           |
| 自演化                | 会话自动提取          | 场景→画像聚合          | 四层自演化              | 巩固 + 冲突解决                | 两层蒸馏               |
| 多模态                | 图片理解（VLM）       | 未提及                 | 文本 + 图片 + 工具轨迹 + 画像 | 纯文本                       | 文本 + 图片 + 音频         |
| 部署复杂度            | 高（Python+Rust+C++） | 低（SQLite 零配置）     | 高（Neo4j+Qdrant）      | 极低（pipx 一条命令）         | 中（seekdb 嵌入式可选） |
| Benchmark LoCoMo      | 未公布具体值          | 未公布                 | 88.83                   | R@10: 95.75%                 | 87.79%                 |
| Benchmark LongMemEval | 未公布                | 未公布                 | 89.20                   | R@10: 99.4%                  | 未公布                 |
| Token 节省             | 降低 92-96%            | 降低 33-61%             | 降低 35.24%              | 未公布                       | 降低 96.5%              |
| 许可证                | 开源                  | 开源                   | Apache 2.0              | MIT                          | Apache 2.0             |

---

## 业界知名 Agent Memory 项目补充

![Agent Memory 项目](https://cdn.mahaoliang.tech/2026/20260727081559084.png)

除上述 5 个项目外，以下项目在 Agent Memory 领域同样具有重要影响力：

**Mem0**（mem0ai/mem0）—— 目前 Star 数最高（~61K）的通用记忆层，Y Combinator S24 孵化，2025 年 10 月融资 $24M。2026 年 4 月发布的新算法在 LoCoMo 上达到 92.5 分，LongMemEval 94.4 分，单次检索 token 消耗仅 ~7K。已集成 21 个框架和 20 种向量数据库。ECAI 2025 论文（arXiv:2504.19413）是领域内重要的学术基准。

**Letta**（letta-ai/letta，原 MemGPT）—— ~18K-21K Stars，受操作系统启发的分层记忆架构（主记忆/外部存储/归档），Agent 主动管理自己的记忆。LongMemEval 得分 83.2%，在"自改进 Agent"场景中表现突出。

**Zep / Graphiti**（getzep/graphiti）—— ~12K-24K Stars，以时序知识图谱（Temporal Knowledge Graph）为核心。LongMemEval 得分 63.8%-71.2%，在需要追踪事实随时间变化的场景中最为适用。

**agentmemory**（rohitg00/agentmemory）—— ~25K Stars，MCP 原生的编码 Agent 记忆，支持 Claude Code、Cursor、Codex 等主流工具。号称减少 60%+ 的重复解释。

**MemPalace** —— ~41K-55K Stars，社区驱动项目，在 LoCoMo 等基准上有稳定表现，被视为"保守但可靠"的选择。

---

## 关键趋势与洞察

![趋势](https://cdn.mahaoliang.tech/2026/20260727081637901.png)

**趋势一：分层取代扁平**。几乎所有新项目都在做记忆分层，无论是 OpenViking 的 L0/L1/L2，TencentDB 的语义金字塔，还是 MemOS 的 Multi-Cube。扁平向量存储难以应对复杂 Agent 场景。

**趋势二：自演化成为标配**。记忆不再只是"存进去再取出来"，而是需要从经验中自动蒸馏出更高层的抽象（技能、策略、世界模型）。PowerMem 的 Experience+Skill 蒸馏和 MemOS 的四层自演化代表了这个方向。

**趋势三：遗忘是被低估的能力**。hebb-mind 和 PowerMem 引入的艾宾浩斯遗忘机制是一个重要创新点。不遗忘的记忆系统会越来越臃肿，检索质量下降。动态 TTL 和基于重要性的衰减是值得关注的方向。

**趋势四：评测标准逐渐统一**。LoCoMo、LongMemEval 和 BEAM 已成为事实上的标准 Benchmark。但各项目公布的指标口径不完全一致（有的报 R@10，有的报准确率），直接比较仍需谨慎。

**趋势五：Token 效率是硬指标**。所有项目都在强调 token 节省——从 OpenViking 的 92-96% 到 PowerMem 的 96.5%。生产环境中，这是实打实的成本。

**趋势六：与编码 Agent 深度集成**。OpenClaw、Claude Code、Cursor、Codex 等编码 Agent 的记忆集成是各项目都要集成的能力，说明编码辅助是当前 Agent Memory 最成熟的落地场景。

---

## 选型建议

![选型](https://cdn.mahaoliang.tech/2026/20260727081704707.png)

如果追求最低部署门槛和快速验证，hebb-mind 的零依赖设计和出色的 Benchmark 成绩值得优先评估，但需考虑社区支持风险。

如果在字节跳动生态内或需要文件系统式的上下文管理，OpenViking 的分层加载和可视化检索是强项，但部署复杂度较高。

如果需要短期记忆压缩能力（长任务场景），TencentDB-Agent-Memory 的 Mermaid 符号图方案非常创新。

如果追求全面的记忆操作系统能力（多模态、多租户、多 Agent 协作），MemOS 的设计最为完整，但需要承受 Neo4j + Qdrant 的运维成本。

如果已在 OceanBase 生态中或需要数据库级别的存储可靠性，PowerMem 的经验蒸馏和广泛 IDE 集成是不错的选择。

如果是通用场景且追求生态成熟度，Mem0 仍然是当前最安全的选择——61K Stars、21 个框架集成、学术论文支撑、融资保障。
