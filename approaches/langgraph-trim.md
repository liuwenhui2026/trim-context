# LangGraph: Message Trimming

> 🚧 Work in Progress

LangGraph 的上下文裁剪策略。

## 核心机制

- `trim_messages`: 基于 token 计数裁剪消息历史
- 状态管理: 图状态中的消息列表自动管理
- 持久化: checkpointer 支持跨会话状态持久化

## 详细分析

<!-- TODO: 源码级拆解 -->