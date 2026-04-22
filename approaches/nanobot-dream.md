# Nanobot: Dream 两级记忆

> 🚧 Work in Progress

Nanobot (HKUDS) 的上下文管理策略。

## 核心机制

- Dream: 两级记忆系统（短期 context + 长期 skill 发现）
- 基于计数的记忆管理: token-based 长期记忆
- Autocompact: 自动压缩对话上下文

## 源码关键路径

- `nanobot/agent/memory.py` — 记忆系统
- `nanobot/agent/autocompact.py` — 自动压缩

## 详细分析

<!-- TODO: 源码级拆解 -->