---
type: concept
status: draft
id: concept-ward-linkage-derivation
title: Ward's Linkage Distance 完整推导
confidence: high
source: Park2024-REPrune
tags:
  - type/concept
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/math
  - domain/clustering
---

# Intuition

Ward's linkage distance 衡量两个簇合并时的**异质性增加量**。它基于**误差平方和 (SSE)** 的概念，确保合并后的簇内部尽可能紧凑。

核心思想：**优先合并使 SSE 增加最小的簇对**。

---

# Rigour

## 定义

对于两个任意簇 $A_c$ 和 $B_c$，Ward's linkage distance 定义为：

$$I(A_c, B_c) = \Delta(A_c \cup B_c) - \Delta(A_c) - \Delta(B_c)$$

其中 $\Delta(\cdot)$ 表示簇的 **Sum of Squared Error (SSE)**。

## SSE 定义

对于簇 $C$，其 SSE 定义为簇内所有核到簇均值的平方欧氏距离之和：

$$\Delta(C) = \sum_{\kappa \in C} \|\kappa - \mathbf{m}_C\|^2$$

其中 $\mathbf{m}_C$ 是簇 $C$ 的均值（质心）。

## 完整推导过程

### Step 1: 合并簇的 SSE

当合并 $A_c$ 和 $B_c$ 时，合并后簇的 SSE 为：

$$\Delta(A_c \cup B_c) = \sum_{\kappa \in A_c \cup B_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2$$

### Step 2: 分解 SSE

将合并簇的核分为两部分：

$$\Delta(A_c \cup B_c) = \sum_{\kappa \in A_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2 + \sum_{\kappa \in B_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2$$

### Step 3: 分析第一部分

对于第一部分（$A_c$ 中的核）：

$$\sum_{\kappa \in A_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2$$

注意到合并后的均值可以表示为：

$$\mathbf{m}_{A_c \cup B_c} = \frac{|A_c|\mathbf{m}_{A_c} + |B_c|\mathbf{m}_{B_c}}{|A_c| + |B_c|} = \mathbf{m}_{A_c} - \frac{|B_c|}{|A_c| + |B_c|}(\mathbf{m}_{A_c} - \mathbf{m}_{B_c})$$

### Step 4: 代数变换

令 $\delta = \mathbf{m}_{A_c} - \mathbf{m}_{B_c}$，则：

$$\|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2 = \left\|\kappa - \mathbf{m}_{A_c} + \frac{|B_c|}{|A_c| + |B_c|}\delta\right\|^2$$

展开：

$$= \|\kappa - \mathbf{m}_{A_c}\|^2 + 2\frac{|B_c|}{|A_c| + |B_c|}(\kappa - \mathbf{m}_{A_c})^T\delta + \left(\frac{|B_c|}{|A_c| + |B_c|}\right)^2\|\delta\|^2$$

### Step 5: 利用性质简化

关键性质：**$\sum_{\kappa \in A_c} (\kappa - \mathbf{m}_{A_c}) = 0$**

因此，交叉项消失：

$$\sum_{\kappa \in A_c} (\kappa - \mathbf{m}_{A_c})^T\delta = 0$$

### Step 6: 第一部分结果

$$\sum_{\kappa \in A_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2 = \Delta(A_c) + \frac{|A_c||B_c|^2}{(|A_c| + |B_c|)^2}\|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$

### Step 7: 第二部分结果（对称）

同理：

$$\sum_{\kappa \in B_c} \|\kappa - \mathbf{m}_{A_c \cup B_c}\|^2 = \Delta(B_c) + \frac{|A_c|^2|B_c|}{(|A_c| + |B_c|)^2}\|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$

### Step 8: 合并结果

$$\Delta(A_c \cup B_c) = \Delta(A_c) + \Delta(B_c) + \frac{|A_c||B_c|}{|A_c| + |B_c|}\|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$

### Step 9: 最终公式

因此：

$$I(A_c, B_c) = \Delta(A_c \cup B_c) - \Delta(A_c) - \Delta(B_c) = \frac{|A_c||B_c|}{|A_c| + |B_c|}\|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$

## 关键性质

| 性质 | 说明 |
|------|------|
| **单调性** | $d(c; \mathcal{K}^l_j) \leq d(c+1; \mathcal{K}^l_j)$ |
| **非负性** | $I(A_c, B_c) \geq 0$ |
| **对称性** | $I(A_c, B_c) = I(B_c, A_c)$ |
| **零值条件** | 当且仅当 $\mathbf{m}_{A_c} = \mathbf{m}_{B_c}$ 时为零 |

## 直观解释

Ward's distance 可以重写为：

$$I(A_c, B_c) = \frac{1}{\frac{1}{|A_c|} + \frac{1}{|B_c|}} \cdot \|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$

这表示：
- **调和平均**的簇大小
- 乘以**质心距离的平方**

**含义**：
- 大簇合并代价高（避免大簇过早合并）
- 远距离簇合并代价高（保持簇内紧凑）

---

# Evidence

| 来源 | 内容 |
|------|------|
| Appendix A.1 p8 | 完整数学推导 |
| Eq.2 | Ward's distance 定义 |
| Eq.13 | 最终公式 |
| Ward Jr 1963 | 原始论文 |

---

# Links

- **使用**: [[methods/method-kernel-clustering-ward|核聚类 (Ward)]] (置信度: high)
- **前提**: [[concepts/concept-sse|Sum of Squared Error]] (置信度: high)
- **相关**: [[concepts/concept-hierarchical-clustering|层次聚类]] (置信度: high)
- **引用**: [[papers/ward1963-hierarchical-grouping|Ward 1963]] (置信度: high)
