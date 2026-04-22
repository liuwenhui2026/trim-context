# trimmer: 通用截断工具

> 🚧 Work in Progress

框架无关的上下文截断工具，按模式调用。

## 使用

```python
from trimmer import Trimmer

trimmer = Trimmer(
    max_tokens=8000,
    strategy="sliding_window",  # sliding_window / summary / priority / rag_trim
)

trimmed_messages = trimmer.trim(messages)
```

<!-- TODO: 实现 -->