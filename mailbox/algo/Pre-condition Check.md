---
tags:
  - type/permanent
  - status/in-progress
  - topic/learning
  - attr/concept
---

## Definition
边界前置校验（Pre-condition Check）
套路：在进入任何局部的、高风险的模式解构/匹配前，先进行“宏观”边界检查。
代码示例 (C++)：
code
```C++
// 坏习惯：边匹配边担心越界
for (int i = 0; i < text.size(); ++i) {
    if (text[i] == 'A' && text[i+1] == 'B' && text[i+2] == 'C') { ... } // 极易在末尾越界
}

// 好习惯：先剪枝，再安全匹配
int limit = (int)text.size() - 3;
for (int i = 0; i <= limit; ++i) {
    if (text[i] == 'A' && text[i+1] == 'B' && text[i+2] == 'C') { ... } // 绝对安全
}
```

---
## **Related**
