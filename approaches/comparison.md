# 横向对比：各框架截断策略

> 🚧 Work in Progress

## 核心概念映射

| 概念 | Claude Code | Hermes | Nanobot | LangGraph | MEM0 | Letta |
|:-----|:-----------|:-------|:--------|:----------|:-----|:------|
| 截断方式 | 4级渐进压缩 | Autocompact | Dream | trim_messages | 记忆提取 | 分页内存 |
| 短期记忆 | 消息历史 | 对话历史 | Context | State | — | 主上下文 |
| 长期记忆 | 文件/记忆 | Skill+Honcho | Skill发现 | Checkpoint | 记忆库 | 外部存储 |
| 压缩策略 | 裁剪→去重→折叠→摘要 | 压缩+Skill沉淀 | 两级记忆 | Token裁剪 | 提取关键事实 | 换入换出 |
| 恢复机制 | 自动恢复5文件 | Skill调用 | Skill发现 | Checkpoint | 记忆召回 | 分页换入 |

## 选型速查

<!-- TODO: 按场景给出推荐 -->