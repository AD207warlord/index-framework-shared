# Index Framework Shared

设计端(Claude副脑) + 执行端(Claude Code) + 用户(99) 三方协作的同步桥梁。

## 结构

```
shared/       ← 双方共享的定稿文档
for-review/   ← 等待99 review的草稿
from-design/  ← 设计端交付的内容
```

## 原则

- 只放脱敏内容，不放敏感信息
- 设计端通过 raw URL 读取
- 本地通过 git push/pull 同步
- 99 做最终裁决
