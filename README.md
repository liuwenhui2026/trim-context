# trim-context

> 上下文超了怎么办？各开源框架的截断策略 + 生产选型

LLM 的上下文窗口再大也有上限。多轮对话、工具调用返回、RAG 检索结果——迟早会爆。

各框架怎么处理？生产怎么选？这里给出答案。

## 上下文工程 vs RAG

RAG 解决的是"从外部知识中选出最相关的放进上下文"——这是 **Select** 问题。

但上下文工程比 RAG 大得多：

```
上下文工程（Context Engineering）
  ├── Write    → Prompt 设计、Memory 结构化存储
  ├── Select   → RAG 检索、动态工具加载        ← RAG 在这里
  ├── Compress → 摘要、截断、Token 剪枝         ← 本仓库聚焦
  └── Isolate  → 状态隔离、子 Agent 独立上下文
```

截断恰好是 RAG 和上下文工程的交汇点——RAG 检索到 20 条结果，上下文只放得下 5 条，怎么截？截完之后跟对话历史怎么排？进来之后什么时候该丢？

## 里面有什么

| 目录 | 内容 | 状态 |
|:-----|:-----|:-----|
| `approaches/` | 各开源框架的截断实现源码级拆解 | 🚧 |
| `patterns/` | 截断模式提炼（滑窗/压缩/优先级/隔离） | 🚧 |
| `production/` | 生产选型：你的场景该用哪种 + RAG 管线设计 | 🚧 |
| `toolkit/` | 通用截断工具 + 效果评测 | 🚧 |

## approaches/ — 各框架怎么做的

| 框架 | 截断策略 | 核心机制 | 详见 |
|:-----|:---------|:---------|:-----|
| Claude Code | 4 级渐进压缩 | 裁剪→去重→折叠→摘要，压缩后自动恢复最近 5 文件 | [claude-code-compact.md](approaches/claude-code-compact.md) |
| Hermes Agent | Autocompact | 自动压缩对话历史，Skill 自创建/自优化 | [hermes-autocompact.md](approaches/hermes-autocompact.md) |
| Nanobot | Dream 两级记忆 | 短期 context + 长期 skill 发现 | [nanobot-dream.md](approaches/nanobot-dream.md) |
| LangGraph | Message Trimming | 基于 token 计数裁剪消息历史 | [langgraph-trim.md](approaches/langgraph-trim.md) |
| MEM0 | 长期记忆管理 | 跨会话记忆提取与召回 | [mem0.md](approaches/mem0.md) |
| MemGPT/Letta | 虚拟上下文管理 | 分页内存，类似操作系统内存管理 | [letta-memgpt.md](approaches/letta-memgpt.md) |

**横向对比**：[comparison.md](approaches/comparison.md)

## patterns/ — 截断模式提炼

从各框架实现中提炼出的通用模式，框架无关：

| 模式 | 思路 | 适用场景 |
|:-----|:-----|:---------|
| [Sliding Window](patterns/sliding-window.md) | 留头留尾丢中间 | 多轮对话，中间轮次不重要 |
| [Summary Compress](patterns/summary-compress.md) | 旧消息摘要替代原文 | 长会话，历史信息需要保留语义 |
| [Priority Rank](patterns/priority-rank.md) | 按重要性保留 | 混合内容（对话+工具+检索），需要精挑 |
| [RAG Trim](patterns/rag-trim.md) | RAG 检索结果的截断策略 | RAG 管线，检索结果超窗口 |
| [Isolate Fork](patterns/isolate-fork.md) | 子任务独立上下文 | Multi-Agent，避免上下文污染 |

## production/ — 生产怎么选

| 文档 | 解决的问题 |
|:-----|:----------|
| [decision-flow.md](production/decision-flow.md) | 你的场景该用哪种截断策略？决策流 |
| [rag-context-pipeline.md](production/rag-context-pipeline.md) | RAG + 上下文工程完整链路设计 |
| [cost-analysis.md](production/cost-analysis.md) | 截断 ≠ 省钱？各方案 Token 成本对比 |
| [pitfalls.md](production/pitfalls.md) | 踩坑：截断导致的幻觉、遗忘、冲突 |

## toolkit/ — 拿来即用

| 工具 | 说明 |
|:-----|:-----|
| [trimmer/](toolkit/trimmer/) | 通用截断工具，框架无关，按模式调用 |
| [benchmarks/](toolkit/benchmarks/) | 各截断方案的效果评测 |

## 为什么做这个

上下文工程已经是 2026 最热的 AI 工程话题——Shopify CEO 说它将取代 prompt engineering，北大发了学术论文，LangChain 写了专题博客。

但现状是：**理论很多，实战很少。**

- Awesome-Context-Engineering 收了上百篇论文，但没人对比各框架的截断实现
- Claude Code 的 4 级压缩被拆解了，但开源框架怎么做的没人讲
- 概念定义一堆，但"我的 RAG 系统上下文爆了该选哪种策略"没人回答

这个仓库补的就是这个缺：**各框架源码级对比 + 生产选型 + 可直接用的工具。**

## 快速开始

```bash
git clone https://github.com/liuwenhui2026/trim-context.git
cd trim-context

# 看各框架怎么处理截断
cat approaches/comparison.md

# 找适合你的截断策略
cat production/decision-flow.md

# 用通用截断工具
pip install -e toolkit/trimmer/
```

## 参考

- [Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering) — 上下文工程学术综述
- [Agent-Skills-for-Context-Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) — 14 个上下文工程 Skill
- [How Claude Code Works](https://github.com/Windy3f3f3f3f/how-claude-code-works) — Claude Code 源码拆解
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Nous Research 自进化 Agent
- [Nanobot](https://github.com/HKUDS/nanobot) — 港大超轻量 Agent

## License

MIT