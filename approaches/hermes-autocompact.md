# Hermes Agent: Autocompact

> 🚧 Work in Progress

Hermes Agent (Nous Research) 的上下文管理策略。

## 核心机制

- Autocompact: 自动压缩对话历史
- Skill 自创建/自优化: Agent 完成复杂任务后自动沉淀为 Skill，后续调用走 Skill 而非重复完整上下文
- Honcho 用户建模: 跨会话建模用户偏好，替代原始对话历史
- FTS5 会话搜索: 全文检索历史对话，需要时才加载

## 源码关键路径

- `src/agents/` — Agent 核心循环
- Skill 系统 — 过程性记忆

## 详细分析

<!-- TODO: 源码级拆解 -->