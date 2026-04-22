# 选型决策流：你的场景该用哪种截断策略

> 🚧 Work in Progress

## 决策树

```
你的上下文为什么会爆？
│
├─ 多轮对话太长
│   ├─ 中间轮次不重要 → Sliding Window
│   └─ 需要保留历史语义 → Summary Compress
│
├─ RAG 检索结果太多
│   └─ → RAG Trim（Top-K / 动态预算 / 分层加载）
│
├─ 工具调用输出太大
│   ├─ 输出用完即弃 → 直接裁剪
│   └─ 后续还要引用 → 落盘 + 按需加载
│
└─ 多种内容混合（对话+工具+检索）
    ├─ 简单场景 → Sliding Window + RAG Top-K
    └─ 复杂场景 → Priority Rank + Isolate Fork
```

## 按场景推荐

<!-- TODO: 电商客服、知识库问答、代码助手等具体场景推荐 -->