# Isolate Fork: 子任务独立上下文

> 🚧 Work in Progress

不是压缩上下文，而是从一开始就避免上下文爆炸——给子任务独立的上下文空间。

## 思路

主 Agent 把子任务派给子 Agent，子 Agent 有独立的上下文窗口。主 Agent 只拿到子 Agent 的执行结果（摘要），不继承子 Agent 的完整对话。

## 适用场景

- Multi-Agent 协作
- 工具链很长（搜索→读取→分析→总结），中间步骤不需要保留
- 避免工具输出撑爆主上下文

## 优缺点

- ✅ 从根源上控制上下文增长
- ✅ 子任务之间互不污染
- ❌ 子任务结果可能丢失细节
- ❌ 子 Agent 调用有额外成本

## 框架实现

- Claude Code: 子 Agent + Worktree 隔离
- Hermes: Delegate + Parallelize
- Nanobot: Subagent

<!-- TODO: 实现细节对比 -->