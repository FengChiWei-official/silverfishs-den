---
tags:
  - type/permanent
  - status/in-progress
  - topic/learning
  - attr/concept
---

## Definition

在算法竞赛（如 CSP, NOIP）中，“给定长度 $N$ 填入 26 个字母，求满足/不满足某些特定子串条件的字符串数量”是一类**极其经典的套路题**。

不需要每次都去头脑风暴状态怎么转移，可以通过一套**“AC 自动机 + 状态压缩 DP”的万能模板**，实现机械化、无脑化套用。

---

### 💡 “无脑套模板”的核心公式
> **AC 自动机** 帮你解决“当前匹配到哪个前缀”的问题；
> **`mask`（状态压缩）** 帮你解决“已经集齐了哪些串/满足了什么顺序”的问题。

通过以下四个步骤，可以轻松拿下此类题目：

---

### 第一步：直接背诵 “万能自动机构建” 模板（不需修改）

在头文件中，放上这套自动构建失配指针的模板。它会自动帮你把“复杂的后缀匹配关系”简化为 `O(1)` 的状态转移数组 `tr[u][c]`。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 998244353;
const int MAX_NODES = 205; // 节点上限：所有要匹配的模式串总长度加起来即可（如 ccf 3 + cspark 6 = 9，定为 205 足够）

int tr[MAX_NODES][26], fail[MAX_NODES];
int match_mask[MAX_NODES]; // match_mask[u] 是一个二进制位，代表节点 u 代表的后缀中，包含哪些目标串
int tot = 0; // 自动机节点计数器

// 无脑插入：s 为要匹配的串，id 为该串的编号（如 ccf 编为 1，cspark 编为 2）
void insert(string s, int id) {
    int u = 0;
    for (char c : s) {
        int idx = c - 'a';
        if (!tr[u][idx]) tr[u][idx] = ++tot;
        u = tr[u][idx];
    }
    match_mask[u] |= (1 << (id - 1)); // 标记此节点匹配到了第 id 个串
}

// 无脑构建失败指针（BFS 模板）
void build_AC() {
    queue<int> q;
    for (int i = 0; i < 26; i++) {
        if (tr[0][i]) q.push(tr[0][i]);
    }
    while (!q.empty()) {
        int u = q.front(); q.pop();
        
        // 关键：自动继承失配指针的匹配状态（例如：匹配了 cspark，自然也匹配了其后缀 ark）
        match_mask[u] |= match_mask[fail[u]]; 
        
        for (int i = 0; i < 26; i++) {
            if (tr[u][i]) {
                fail[tr[u][i]] = tr[fail[u]][i];
                q.push(tr[u][i]);
            } else {
                tr[u][i] = tr[fail[u]][i]; // 快捷转移：失配时直达下一个可行节点
            }
        }
    }
}
```

---

### 第二步：根据题目要求，定制 `mask` 的状态

当每走完一步、到达一个新节点 `v` 后，需要根据 `match_mask[v]`（当前拿到了哪些串）来更新全局的 `mask`。

#### 情形 A：题目要求“不能出现任何一个目标串”
* **无脑设计**：不需要 `mask`。只要下一步走到的节点 $v$ 满足 `match_mask[v] != 0`，就代表完蛋了，直接抛弃不转移即可。

#### 情形 B：题目要求“必须同时包含串 1 和 2”
* **无脑设计**：`mask` 取值 $0 \sim 3$（对应二进制 `00`, `01`, `10`, `11`）。
* **转移逻辑**：
  ```cpp
  int get_next_mask(int mask, int v) {
      return mask | match_mask[v]; // 用按位或，直接把新拿到的串并入已集齐的集合中
  }
  ```

#### 情形 C：题目要求“先包含串 1，后包含串 2”（本题）
* **无脑设计**：设计五个状态：`0`（啥都没有），`1`（只有1），`2`（只有2），`3`（都有，但顺序是2先1后），`4`（都有，且1先2后，成功）。
* **转移逻辑**：
  ```cpp
  int get_next_mask(int mask, int v) {
      int found = match_mask[v]; // 看看在新节点 v 发现了什么 (第 1 位是 ccf，第 2 位是 cspark)
      bool has_1 = (found & 1);
      bool has_2 = (found & 2);
      
      if (has_1) {
          if (mask == 0) mask = 1;
          if (mask == 2) mask = 3;
      }
      if (has_2) {
          if (mask == 0) mask = 2;
          if (mask == 1 || mask == 3) mask = 4; // 发现 2 的时候，只要之前有过 1，就直接进入成功状态
      }
      return mask;
  }
  ```

---

### 第三步：写出无脑 DP 骨架

有了上面的部件，DP 循环完全就是流水线作业。设自动机共有 `tot + 1` 个节点，`mask` 共有 `M_SIZE` 种状态。

```cpp
// 初始化 DP 数组：dp[节点 u][状态 mask]
int M_SIZE = 5; // 本题的 mask 共有 5 种
vector<vector<long long>> dp(tot + 1, vector<long long>(M_SIZE, 0));
dp[0][0] = 1; // 初始状态：处于根节点 0，mask 为 0

for (int i = 1; i <= n; i++) {
    vector<vector<long long>> next_dp(tot + 1, vector<long long>(M_SIZE, 0));
    
    if (is_fixed[i]) { // 如果当前字符固定（比如是 '#' 或其他特定字符）
        // 重置自动机状态至 0（不能跨越边界），但保留当前的匹配 mask
        for (int mask = 0; mask < M_SIZE; mask++) {
            long long sum = 0;
            for (int u = 0; u <= tot; u++) sum = (sum + dp[u][mask]) % MOD;
            next_dp[0][mask] = sum;
        }
    } else {
        // 普通情况，无脑遍历 26 个字母尝试转移
        for (int u = 0; u <= tot; u++) {
            for (int mask = 0; mask < M_SIZE; mask++) {
                if (dp[u][mask] == 0) continue;
                
                for (int c = 0; c < 26; c++) {
                    int v = tr[u][c]; // 走向下一个节点
                    int nmask = get_next_mask(mask, v); // 更新 mask
                    
                    next_dp[v][nmask] = (next_dp[v][nmask] + dp[u][mask]) % MOD;
                }
            }
        }
    }
    dp = move(next_dp);
}
```

---

### 第四步：汇总答案

在 DP 结束后，**无脑累加所有匹配成功的状态的值**。对于本题，任何停留在合法状态（`mask == 4`）的方案，无论自动机目前停在哪个节点，都是正确答案。

```cpp
long long ans = 0;
for (int u = 0; u <= tot; u++) {
    ans = (ans + dp[u][4]) % MOD; // 累加 mask 为成功状态 4 的方案数
}
cout << ans << "\n";
```

### 💡 总结你的考场决策：
以后在考场上遇到任何类似的字符串计数题：
1. **直接默写** 第一步的 `insert` 和 `build_AC` 模板。
2. 在 `main` 函数开头写 `insert("目标串", 1)`、`insert("目标串2", 2)`，然后调一下 `build_AC()`。
3. **想一想 `mask` 应该怎么设计**（通常是 2 位二进制 `0~3`，或者像本题一样写一个简单的 5 状态小分支判断）。
4. **无脑套用第三、四步的循环骨架**，直接提交，大功告成！

---
## **Related**