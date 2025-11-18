# 标签系统设计

## 概述
基于现有笔记库的实际结构与内容，设计的四层标签系统：
1. **一级分类**（笔记类型）：概念定义、算法方法、理论基础等
2. **二级领域**（知识领域）：数学、计算机、物理等
3. **三级细分**（具体主题）：代数、网络、机器学习等
4. **属性标签**（笔记特性）：重要度、完成度等

---

## 一级分类标签

| 标签 | 定义 | 示例笔记 | 使用场景 |
|-----|------|--------|--------|
| `#concept` | 基本概念/定义 | Group, Monoid, Semigroup | 定义新的数学/计算机概念 |
| `#principle` | 基本原理/定律 | Law of Gauss, Coulomb's Force | 物理定律、基础原理 |
| `#algo` | 算法/方法 | Paging, Binning, FFT | 解决问题的具体方法 |
| `#technique` | 技术/实现 | Attention, PEFT, RLHF | 工程实现、最佳实践 |
| `#history` | 历史背景/演进 | History of Attention, History of Memory Allocation | 理解发展脉络 |
| `#problem` | 问题/练习 | （待添加） | 实际问题或练习题 |
| `#proof` | 证明/推导 | Normal Form of Provement | 数学证明或推导过程 |
| `#index` | 导航/索引 | LLM Fine-tuning Index | 知识体系入口 |

---

## 二级领域标签（使用斜线分层）

### 数学 (`#math/*`)
```
#math/algebra/abstract       - 抽象代数（群、环、域等）
#math/algebra/linear         - 线性代数（矩阵、向量等）
#math/provement-method       - 证明方法与技巧
#math/analysis               - 分析（极限、导数、积分等）
#math/discrete               - 离散数学（组合、图论等）
```

**现有笔记示例：**
- Group, Monoid, Semigroup, Closure, Identity, Inverse, Addition Group → `#math/algebra/abstract`
- Normal Form of Provement → `#math/provement-method`

### 计算机科学 (`#computer-science/*`)
```
#computer-science/networks   - 计算机网络（TCP/IP, ARP等）
#computer-science/os         - 操作系统（内存分配、分页等）
#computer-science/architecture - 计算机体系结构
#computer-science/dl         - 深度学习（神经网络、模型等）
#computer-science/foundation - 计算基础理论
```

**现有笔记示例：**
- Computer Network, TCP-IP, ARP Protocol → `#computer-science/networks`
- Dynamic Partitioning, Paging, Memory Allocation → `#computer-science/os`
- Attention, PEFT, RLHF, SFT → `#computer-science/dl`

### 物理 (`#physics/*`)
```
#physics/electromagnetism    - 电磁学
#physics/mechanics           - 力学
#physics/thermodynamics      - 热力学
#physics/quantum             - 量子力学
```

**现有笔记示例：**
- Electric Field, Electric Flux, Coulomb's Force, Law of Gauss → `#physics/electromagnetism`
- Dipole Moment, Electric Dipole → `#physics/electromagnetism`

### 机器学习 (`#ml/*`)
```
#ml/training                 - 模型训练方法（SFT, RLHF等）
#ml/arch                     - 模型架构（Attention等）
#ml/peft                     - 参数高效微调（PEFT, LoRA等）
#ml/preprocessing            - 数据预处理（Binning等）
#ml/foundation               - 基础理论（梯度、反向传播等）
```

**现有笔记示例：**
- SFT, RLHF, Full Fine-tuning → `#ml/training`
- Attention, PEFT, Reparameterization-based PEFT → `#ml/arch` 或 `#ml/peft`
- FFT, Gradient, Binning → `#ml/foundation` 或 `#ml/preprocessing`

### 通用 (`#general/*`)
```
#general/thought             - 思想/思考记录
#general/tool                - 工具使用指南
#general/resource            - 资源链接
```

**现有笔记示例：**
- Middle-layer thoughts, Standardizing units → `#general/thought`

---

## 三级实体分类标签

用于标记笔记描述的是什么类型的对象：

| 标签 | 含义 | 示例 |
|-----|------|------|
| `#entity/abstract` | 抽象概念/理论 | Group, TCP/IP Protocol |
| `#entity/concrete` | 具体对象/实现 | Paging Algorithm, Attention Mechanism |
| `#entity/virtual` | 虚拟/计算机概念 | Process, Virtual Memory |
| `#entity/physical` | 物理实体/现象 | Electric Field, Coulomb Force |
| `#entity/method` | 方法/算法 | FFT, Binning |

**现有笔记示例：**
- Group, Computer Network → `#entity/abstract`
- Paging, RLHF → `#entity/concrete`
- Virtual Memory, Process → `#entity/virtual`
- Electric Field, Coulomb's Force → `#entity/physical`

---

## 属性标签（可选，用于个人标记）

| 标签 | 含义 | 用途 |
|-----|------|------|
| `#important` | 重要概念 | 标记核心笔记 |
| `#wip` | 工作中 | 笔记还在补充完善 |
| `#review` | 待审视 | 笔记需要再次验证或优化 |
| `#solved` | 已解决 | 问题已得到解答 |

---

## 实际标签应用示例

### 示例 1：群论笔记
```markdown
#concept #math/algebra/abstract #entity/abstract

## 定义
- 一个拥有逆元的 [[monoid]]
```

### 示例 2：分页算法笔记
```markdown
#algo #computer-science/os #entity/concrete #technique

## 定义
将原始内存分成固定长度的块...
```

### 示例 3：注意力机制笔记
```markdown
#technique #computer-science/dl #entity/concrete #ml/arch

## 概述
自注意力机制的实现与应用...
```

### 示例 4：电场笔记
```markdown
#principle #physics/electromagnetism #entity/physical

## 定义
电场是由电荷产生的...
```

---

## 当前标签统计

基于仓库扫描（2025-11-18）：

| 标签 | 使用次数 | 知识领域 |
|-----|---------|--------|
| `#computer-science` | 7 | 网络、操作系统、机器学习 |
| `#math/algebra/abstract` | 5 | 抽象代数 |
| `#concept` | 3 | 通用概念 |
| `#entity/virtual` | 2 | 虚拟计算概念 |
| `#entity/abstract` | 2 | 抽象概念 |
| `#technique` | 1 | 技术实现 |
| `#history` | 1 | 历史背景 |
| `#algo` | 1 | 算法方法 |

---

## 推荐的标签层级结构

按优先级建议使用：

```
1️⃣ 一级分类（必须）
   - #concept, #principle, #algo, #technique, #history, #proof, #index
   
2️⃣ 二级领域（推荐）
   - #math/algebra/abstract
   - #computer-science/networks
   - #computer-science/os
   - #computer-science/dl
   - #physics/electromagnetism
   - #ml/training
   - #ml/arch
   - #general/thought
   
3️⃣ 实体分类（推荐）
   - #entity/abstract, #entity/concrete, #entity/virtual, #entity/physical
   
4️⃣ 属性标签（可选）
   - #important, #wip, #review, #solved
```

---

## 使用指南

### 标签添加方式

#### 方式 1：在文件顶部一行添加（推荐）
```markdown
#concept #math/algebra/abstract #entity/abstract

## 定义
```

#### 方式 2：在内容中自然使用
在笔记内容中提及相关概念时自然添加标签：
```markdown
## 相关领域
这与 #computer-science/networks 和 #entity/virtual 的概念相关。
```

#### 方式 3：使用 Frontmatter（适合大型笔记）
```markdown
---
tags:
  - concept
  - math/algebra/abstract
  - entity/abstract
---

## 定义
```

### 在 Obsidian 中查看标签

1. **打开右侧面板** → **Tags**
2. **搜索标签**：使用 `tag:#concept` 或 `tag:#math/algebra/abstract`
3. **按标签过滤**：点击 Tags 面板中的标签自动显示所有相关笔记
4. **标签树视图**：斜线会自动创建树状结构显示

---

## 下一步行动

- [ ] 逐个审查现有笔记，确保标签正确应用
- [ ] 为未标注笔记添加标签（见下方待标注列表）
- [ ] 定期查看 `tag:#wip` 完成未完成的笔记
- [ ] 根据新增笔记类型扩展标签系统