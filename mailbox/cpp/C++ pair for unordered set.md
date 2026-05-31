---
tags:
  - type/permanent
  - status/in-progress
  - topic/learning
  - attr/concept
---

## Definition

``` cpp

#include <iostream>
#include <unordered_map>
#include <utility>

// 自定义 pair 的哈希结构体
struct PairHash {
    template <class T1, class T2>
    std::size_t operator () (const std::pair<T1, T2>& p) const {
        auto h1 = std::hash<T1>{}(p.first);
        auto h2 = std::hash<T2>{}(p.second);
        // 经典的 Boost hash_combine 算法
        return h1 ^ (h2 + 0x9e3779b9 + (h1 << 6) + (h1 >> 2));
    }
};

int main() {
    // 将 PairHash 作为第三个模板参数传入
    std::unordered_map<std::pair<int, int>, std::string, PairHash> grid_map;
    grid_map[{1, 2}] = "Starting Point";
    
    std::cout << grid_map[{1, 2}] << std::endl;
    return 0;
}
```

---
## **Related**