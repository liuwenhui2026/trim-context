# SPEC: 复刻 trim-context 仓库

> 本文件是一个完整的复刻指南，供 LLM 从零生成整个 trim-context 仓库的所有内容。
> 遵循此 spec 可产出功能等价、结构一致的开源仓库。

---

## 第一章：项目定位与核心叙事

### 项目名称

`trim-context`

### 一句话定位

Context engineering toolkit — 从开源框架中提炼上下文截断与管理模式，提供可直接使用的 Python 工具包。

### 核心问题

当 LLM Agent 执行长程任务时，上下文窗口必定不够用。开源社区已经有大量框架在解决这个问题，但：

1. **没有横向对比**：各框架的截断策略零散分布在代码里，没有人放在一起比
2. **没有模式提炼**：Sliding Window、摘要压缩、优先级裁剪等是通用模式，但没有从具体实现中抽象出来
3. **没有生产指导**：什么时候该用什么策略？不同场景怎么选型？
4. **没有可复用工具**：每个框架自己实现一套截断逻辑，没法直接用在别的项目里

### 仓库结构

```
trim-context/
├── README.md                        # 仓库总览
├── panorama.md                      # 全景图 v2
│
├── approaches/                      # 框架实现分析（6个框架）
│   ├── claude-code-compact.md       # Claude Code 4级渐进压缩
│   ├── hermes-autocompact.md        # Hermes Autocompact + Skill沉淀
│   ├── nanobot-dream.md             # Nanobot Dream双层记忆
│   ├── langgraph-trim.md            # LangGraph trim_messages
│   ├── mem0.md                      # MEM0 V3 记忆管道
│   ├── letta-memgpt.md              # Letta/MemGPT 三层虚拟内存
│   └── comparison.md                # 横向对比
│
├── patterns/                        # 模式提炼（8篇）
│   ├── sliding-window.md            # 滑动窗口
│   ├── summary-compress.md          # 摘要压缩
│   ├── priority-rank.md             # 优先级裁剪
│   ├── rag-trim.md                  # RAG截断策略
│   ├── isolate-fork.md              # 子Agent隔离
│   ├── perception.md                # 感知层（预算监控/阶段感知/实体追踪/噪声识别）
│   ├── verify.md                    # 验证层（压缩一致性/实体归属/反事实评估/误差追踪）
│   └── guard.md                     # 防护层（过度结合抑制/实体污染/噪声过滤/欠召回）
│
├── production/                      # 生产实践（4篇）
│   ├── decision-flow.md             # 选型决策树
│   ├── rag-context-pipeline.md      # RAG+上下文 Pipeline
│   ├── cost-analysis.md             # 成本分析
│   └── pitfalls.md                  # 踩坑指南（10个坑）
│
└── toolkit/trimmer/                 # Python 工具包（实现方式不限，功能对等即可）
```

---

## 第二章：写作风格与规范

### 语言

- **approaches/ 和 patterns/**：中文为主，术语用英文。代码示例和技术术语不翻译。
- **production/**：中文为主。
- **toolkit/**：英文（代码、注释、README、pyproject.toml）。
- **仓库根目录 README.md**：中文。

### 语气

- **技术叙事，不端着**。像跟同事聊技术方案，不像写论文。
- **信息密度高**。每句话都有信息增量，不说废话，不凑字数。
- **用表格和代码说话**。能用表格对比的不要写段落，能用代码示例的不要写描述。
- **有观点**。不害怕下判断——"LangGraph 不内置摘要，你需要自己实现"、"没有一个框架有 Verify"。
- **自嘲和留白可以共存**。示例里可以出现 `)—(` 这样的颜文字表示尴尬，但不要过度。
- **不要客套**。不加"值得注意的是"、"需要指出的是"这类零信息前缀。

### 格式规范

- Markdown，兼容 GitHub 渲染
- 表格用 `|:----|` 左对齐，数字列右对齐
- 代码块标注语言（```python / ```bash / ```text）
- 每篇文档以一句引用开头（`>` 格式），点出这篇文档要解决的核心问题
- 章节用 `##`，子节用 `###`，不用 `#`
- 中英文之间加空格（`Python 框架`，不是 `Python框架`）
- 代码示例可以运行，不是伪代码

### 禁止事项

- 不出现任何公司名称、产品代号、花名、人名
- 不出现医疗、金融等特定行业的患者/用户数据
- 替换所有实例化名：用通用的"用户"、"Agent"、"助手"代替
- 不泄密：框架分析基于公开的开源代码和文档，不包含非公开信息

---

## 第三章：approaches/ — 框架分析规范

每篇框架分析文档必须包含以下结构：

```markdown
# [框架名]：[一句话概括其上下文策略]

> [一句引用，暴露核心洞察]

## 核心机制

[这个框架怎么处理上下文溢出，2-3段概述]

## 详细实现

### [关键组件1]
[具体分析，包含代码片段]

### [关键组件2]
...

## 关键设计决策

| 决策 | 选择 | 理由 |
|------|------|------|

## 优劣势

### 优势
- ...

### 劣势
- ...

## 与其他框架的对比

[简短对比，1-2段]
```

### 6 个框架的分析要点

#### 1. Claude Code Compact (`claude-code-compact.md`)

- **核心**：4 级渐进压缩流水线：Trim → Dedup → Collapse → Autocompact
- **关键机制**：
  - Trim：按 token 阈值裁剪旧消息
  - Dedup：去重相似内容
  - Collapse：合并短消息
  - Autocompact：子 Agent 生成对话摘要
- **缓存感知**：SYSTEM_PROMPT_DYNAMIC_BOUNDARY 分离静态/动态前缀
- **附件消息模式**：动态内容通过 attachment 注入，不改主消息前缀
- **自动恢复**：compact 后自动恢复最近编辑的 5 个文件
- **KV Cache 约束**：前缀必须 byte-identical，否则缓存失效
- **Token 计数**：`ceil(chars/4) + 3` 每条消息开销

#### 2. Hermes Agent (`hermes-autocompact.md`)

- **核心**：Autocompact + Skill 生命周期
- **Skill 生命周期**：auto-create → auto-optimize → reuse
- **Honcho 用户建模**：替代对话历史，用结构化画像存储
- **FTS5 全文检索**：历史 Skill 的快速召回
- **Agentskills.io**：开放标准兼容

#### 3. Nanobot Dream (`nanobot-dream.md`)

- **核心**：Dream 双层记忆——短期 context + 长期 skill 发现
- **ClawHub**：Skill 市场，社区分享
- **设计哲学**：极简 autocompact，不过度设计
- **Dream 过程**：压缩当前 context → 提取可复用 skill → 下次直接加载

#### 4. LangGraph (`langgraph-trim.md`)

- **核心**：`trim_messages()` 纯函数 + `pre_model_hook`
- **trim_messages 参数**：strategy="first"/"last", start_on, include_system, token_counter
- **add_messages reducer**：支持 RemoveMessage 和 REMOVE_ALL_MESSAGES
- **pre_model_hook + llm_input_messages**：不修改 state，只影响 LLM 输入
- **不内置摘要**：checkpoint 存完整快照，截断由用户自己决定
- **近似 Token 计数**：`ceil(chars/4) + 3`

#### 5. MEM0 (`mem0.md`)

- **核心**：V3 记忆管道——纯 ADD + MD5 哈希去重
- **V1 vs V3**：V1 用 LLM 做 ADD/UPDATE/DELETE，V3 简化为纯 ADD + 哈希
- **三路检索**：semantic + BM25 + entity boost
- **外部注入模式**：记忆系统独立于 LLM 调用链
- **硬编码限制**：PAST_MESSAGE_TRUNCATION_LIMIT=300
- **Procedural Memory**：从对话中提取可操作的知识

#### 6. Letta/MemGPT (`letta-memgpt.md`)

- **核心**：三层虚拟内存——Core / Recall / Archival
- **Core Memory**：in-context Blocks，Agent 可用 core_memory_replace 直接编辑
- **Recall Memory**：对话历史 DB，支持搜索
- **Archival Memory**：pgvector 向量存储，archival_memory_insert/search
- **Agent 自治**：Agent 自己决定何时读写内存，通过工具调用
- **90% 水位线**：自动触发 Core Memory 部分驱逐
- **Sleeptime Agent**：后台 Agent 异步整理记忆
- **Block 数据结构**：label / description / limit / read_only

#### 7. 横向对比 (`comparison.md`)

- **核心概念映射表**：6 框架 × 7 概念维度
- **三个光谱对比**：压缩精细度、信息丢失风险、实现复杂度
- **适用场景推荐表**：6 个场景 × 推荐框架
- **Token 消耗对比表**：含正常/压缩触发/额外/节省
- **一句话选型**：甩手掌柜/自己掌控/Agent自治/长期记忆/快速上手

---

## 第四章：patterns/ — 模式文档规范

每篇模式文档必须包含：

```markdown
# [模式名]

> [一句引用]

## 问题

[这个模式解决什么问题，为什么需要它]

## 方案

### [策略1]
[具体实现，包含代码示例]

### [策略2]
...

## 选型指南

| 场景 | 推荐策略 | 理由 |
|------|---------|------|

## 反模式

[常见错误做法]
```

### 8 篇模式文档要点

#### 1. sliding-window.md
- LangGraph trim_messages 兼容
- 系统消息保留、从尾部向前填充、preserve_recent 保底
- 代码示例用 Python
- 反模式：无差别丢弃、丢失系统消息

#### 2. summary-compress.md
- LLM 摘要作为压缩手段
- 摘要质量漂移问题（递归摘要的累积误差）
- 不递归摘要原则：原始 → 摘要只做一次
- 成本：1 次额外 LLM 调用
- 摘要 Prompt 设计要点

#### 3. priority-rank.md
- 评分维度：类型优先级 × 0.6 + 时间近度 × 0.4
- 默认优先级表：system=10, user=8, assistant=5, tool=2
- 自定义优先级的使用场景
- 与 sliding_window 的选择条件

#### 4. rag-trim.md
- 4 种策略：top_k / dynamic / decay / layered
- 每种策略的适用场景和数据流
- RAG 结果排列策略：正序/倒序/交替
- "狼来了"问题：过多低质 RAG 如何稀释注意力
- 代码示例用 trim-context API

#### 5. isolate-fork.md
- 子 Agent 模式：独立上下文、结果汇报
- 任务分解原则：按上下文边界切分
- 与 Sliding Window 的互补关系
- 子 Agent 结果注入主上下文的方式

#### 6. perception.md — 感知层
- 预算监控：token 用量、趋势外推、ETA 计算、分组件监控
- 任务阶段感知：关键步骤识别（二八法则）、阶段切换时上下文重构、关键步骤零误差策略
- 实体追踪：实体状态表、主体消解（中文省略主语问题）
- 噪声识别：信号可信度评估、信噪比监控、渐进式过滤（4 pass）
- 架构图：感知层 → 感知报告 → 决策层

#### 7. verify.md — 验证层
- 压缩一致性检查：关键事实提取 → 校验 → 补救（追加到摘要尾部）
- 事实类别优先级表：data/constraint > decision > status > process > emotion
- 摘要质量评分：5 维度加权
- 实体归属校验：实体归属矩阵、压缩 Prompt 实体约束
- 反事实评估：代理指标（引用率/纠正率/追问率）、A/B 测试、离线回放
- 多步误差追踪：误差预算分配、关键步骤零容忍、检查点对比
- 架构图：执行层 → 验证层（通过/补充/重做）→ 防护层

#### 8. guard.md — 防护层
- 过度结合抑制：四维相关性门控（topic_match / entity_match / time_valid / safety）
- 相关性门控代码示例（Python dataclass）
- 抑制审计日志
- 实体污染检测：实体边界标记、跨实体推理守卫
- 噪声过滤：6 种噪声类型及策略、信噪比监控（SNR > 0.7 目标）、渐进式 4-pass 过滤
- 欠召回策略：场景容忍度矩阵（医疗/金融/编码/闲聊/RAG）、置信度标注（高/中/低）、渐进式揭示
- 架构图：串联管道——过度结合抑制 → 实体污染检测 → 噪声过滤 → 欠召回策略

---

## 第五章：production/ — 生产实践规范

### 1. decision-flow.md — 选型决策树

- 三级决策树：上下文规模 → 任务特征 → 功能需求
- 5 个典型场景推荐
- 框架选型表：场景 × 推荐框架
- 升级信号：什么时候该从简单方案换到复杂方案

### 2. rag-context-pipeline.md — RAG+上下文 Pipeline

- 完整 pipeline：Monitor → Decide → Execute → Verify
- Token 预算分配比例和调整策略
- 3 个关键决策：何时检索、检索多少、如何截断
- 反模式清单：塞满 RAG、忽略缓存、按需但不预取

### 3. cost-analysis.md — 成本分析

- 直接成本：输入/输出 token 单价对比
- 隐藏成本：缓存失效、重复调用、摘要 LLM 调用
- 按场景的成本估算表
- "截断 ≠ 省钱"的核心论点

### 4. pitfalls.md — 踩坑指南（10 个坑）

每个坑的结构：问题描述 → 典型场景（ASCII 对话流）→ 症状 → 解法

**10 个坑**：
1. 幻觉（Hallucination from Missing Context）——截断丢关键事实
2. 注意力稀释（Attention Dilution）——lost-in-the-middle
3. 信息冲突（Context Clash）——摘要与新增信息矛盾
4. 工具输出丢失——截断切掉工具返回值
5. 摘要漂移——递归摘要累积误差
6. 缓存断裂——截断改变消息前缀
7. 过度结合（Over-binding）——模型强行关联不相关历史
8. 实体归属丢失——压缩后"谁的信息"模糊
9. 记忆过期不失效——旧记忆未标记过期
10. 多步累积误差——长链路指数级误差累积

末尾附诊断清单（10 项 checkbox）。

---

## 第六章：全景图 (`panorama.md`)

7 层架构全景图：

```
感知层（When & What）→ 决策层（Decide）→ 执行层（How）→ 验证层（Verify）→ 防护层（Guard）
```

### 感知层
- 预算监控、任务阶段感知、实体追踪、噪声识别

### 决策层
- 四级触发：L0 预防 / L1 阈值 / L2 事件 / L3 紧急
- 关键步骤识别
- 上下文重构

### 执行层
- 压缩：sliding_window / summary / semantic
- 选择：priority / RAG / attention / on-demand-expand
- 隔离：sub-agent / context-fork
- 持久化：checkpoint / skill / virtual-memory / memory-expiry
- 缓存：KV cache optimization

### 验证层
- 压缩一致性、实体归属、反事实评估、多步误差追踪

### 防护层
- 过度结合抑制、实体污染检测、噪声过滤、欠召回策略

### 必须包含的表格

1. **层间流片规则**：感知报告 → 决策 → 执行 → 验证 → 防护 → LLM
2. **开源实现现状表**：每层 × 有实现 / 部分实现 / 无实现
3. **问题域映射表**：生产问题 → 对应层 → 对应技术

---

## 第七章：仓库根 README.md 规范

结构：
1. 一句话标题 + 定位
2. 问题背景：为什么需要上下文工程
3. 仓库结构树
4. 核心内容概览：
   - 6 框架分析
   - 8 模式提炼
   - 全景图 v2（7 层）
   - 10 个坑
   - Python 工具包
5. 快速开始：pip install + 3 行代码示例
6. 选型速查表
7. 贡献指南（简短）

---

## 第八章：交付检查清单

复刻完成后的验收标准：

- [ ] 目录结构与 spec 完全一致
- [ ] 6 篇框架分析文档，每篇包含核心机制/详细实现/设计决策/优劣势
- [ ] 8 篇模式文档，每篇包含问题/方案/选型指南/反模式
- [ ] 4 篇生产实践文档
- [ ] 1 篇全景图，含 7 层架构和 3 张表
- [ ] 无公司名、医疗数据、花名、人名
- [ ] 中文文档语言风格一致：信息密度高、有观点、用表格和代码说话