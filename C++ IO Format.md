---
tags:
  - type/permanent
  - status/in-progress
  - topic/learning
  - attr/concept
---
## 省流

```
#include<iostream>
#include<iomanip>

cout << fixed << setprecision(6) << pi << '\n';

```
## Definition

为了便于日常开发时快速查阅，我们将 C++ `iostream` 的格式化输出技巧、常见避坑点及现代替代方案归纳整理如下：

### 1. 头文件分工（有无参数决定头文件）
* **`<iostream>`**：提供基础流和**无参数**的操纵符（如 `fixed`, `hex`, `boolalpha`, `endl`）。
* **`<iomanip>`**：提供**带参数**的格式化操纵符（如 `setw`, `setprecision`, `setfill`）。

---

### 2. 常用格式化场景速查

| 格式化需求 | 操纵符组合 | 核心代码示例 | 输出效果演示 |
| :--- | :--- | :--- | :--- |
| **保留 $N$ 个有效数字** | `setprecision(N)` | `cout << setprecision(3) << 12.345;` | `12.3`（四舍五入） |
| **保留 $N$ 位小数** | `fixed << setprecision(N)` | `cout << fixed << setprecision(3) << 12.3;` | `12.300`（不足补0） |
| **设置输出宽度与填充** | `setw(N) << setfill(C)` | `cout << setw(4) << setfill('0') << 5;` | `0005`（**`setw` 仅生效一次**） |
| **输出十六进制及前缀** | `hex << showbase` | `cout << hex << showbase << 42;` | `0x2a` |
| **输出 `true`/`false` 文本** | `boolalpha` | `cout << boolalpha << true;` | `true` |

---

### 3. 三大核心“避坑点”

1. **`setw(n)` 的“一次性”特征**：
   大部分操纵符设置后会一直生效，唯独 **`std::setw` 仅对紧随其后的那一个变量生效**。如果有多列数据需要对齐，每一列前都必须重新写一遍 `setw`。
2. **全局状态污染与恢复**：
   诸如 `fixed`、`hex`、`boolalpha` 等操纵符会修改输出流的全局状态。在不需要时，建议及时将其恢复，否则会影响后续所有代码的打印格式：
   ```cpp
   std::cout << std::defaultfloat; // 恢复默认浮点数格式
   std::cout << std::dec;          // 恢复十进制
   std::cout << std::noboolalpha;   // 恢复布尔值为 0/1
   ```
3. **字符/指针打印陷阱**：
   * **字符转 ASCII 码**：`char` 必须强转为整型才能输出数值（`static_cast<int>(ch)`）。
   * **`char*` 打印地址**：直接打印 `char*` 会输出其指向的字符串内容。如果想打印该指针本身的地址，必须将其强转为 `void*`（`static_cast<const void*>(ptr)`），否则容易因为找不到 `\0` 导致程序崩溃。

---

### 4. 现代 C++ 的演进趋势
由于 `iostream` 的流式操纵符存在“修改全局状态”、“语法繁琐”等痛点，现代 C++ 引入了更为简洁安全的格式化方式：
* **C++20**：引入了 `<format>` 库（类似 Python 的格式化语法），如 `std::format("PI: {:.2f}", 3.1415)`。
* **C++23**：引入了 `<print>` 库，支持直接通过 `std::print` 或 `std::println` 打印，不再依赖传统的 `std::cout` 操纵符。

---
## **Related**