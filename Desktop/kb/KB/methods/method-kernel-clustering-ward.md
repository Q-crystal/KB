---
type: method
status: draft
id: method-kernel-clustering-ward
title: 核聚类(Ward Linkage)
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/clustering
  - domain/ward-linkage
---

# Intuition

在REPrune中，对每个输入通道内的所有核(kernel)进行**凝聚层次聚类**。固定输入通道 $j$ 时，该通道对应的所有输出通道的核 $\kappa_{i,j}^l$ 都是同尺寸（如 $3\times3$ 或 $1\times1$），可以公平比较相似度。

使用 **Ward linkage** 计算聚类距离，它用"合并后SSE增量"作为距离度量，具有**单调递增特性**，便于用阈值控制聚类停止。

---

# Rigour

## 定义

对每个输入通道 $j$，将其对应的核集合 $K_j^l \in \mathbb{R}^{n_{l+1}\times k_h\times k_w}$ 进行凝聚层次聚类，使用 **Ward linkage** 计算距离。

## 数学表达

**Ward距离**：

$$I(A_c, B_c) = \frac{|A_c||B_c|}{|A_c|+|B_c|} \|m_{A_c} - m_{B_c}\|^2$$

**Linkage目标**：

$$d(c; K_j^l) = \min I(A_c, B_c)$$

随 $c$ 单调不减

**控制函数**：当 $d(c)$ 超过阈值时停止聚类（Eq.3）

## 假设

- 同一输入通道内的核具有可比性
- Ward距离能有效衡量核的相似性
- 单调性使"阈值切分"更一致

## 适用范围

- CNN卷积层的核分析
- 需要识别核模式相似性的场景
- 与REPrune剪枝策略配合使用

## 复杂度分析

| 复杂度类型 | 表达式 | 说明 |
|-----------|--------|------|
| 时间复杂度 | $O(n^2 \log n)$ | $n$ = 每通道核数 |
| 空间复杂度 | $O(n^2)$ | 距离矩阵存储 |
| 执行次数 | $n_l$ 次 | 每个输入通道一次 |

## 失败模式

- ⚠️ 核分布过于分散时聚类效果差
- ⚠️ 当 $n_{l+1}=1$ 时无法聚类（只有一个核）
- ⚠️ 阈值选择不当导致过度或不足合并

---

# Evidence

| 证据类型 | 来源 | 内容 |
|---------|------|------|
| 符号定义 | p3 | 详细定义核集合 $K_j^l$ 和聚类过程 |
| Ward推导 | 附录p8 | Ward距离的数学推导 |
| 对比实验 | Table 5 (p7) | Ward linkage 优于 single/complete/average linkage |
| 作者解释 | p7 | Ward避免簇内部过分发散，单调性让阈值切分更一致 |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[methods/method-layer-wise-cutoff|层-wise截断]] (置信度: high)
- **支持**: [[claims/claim-ward-linkage-superiority|Ward linkage优势]] (置信度: medium)
- **相似**: [[concepts/concept-agglomerative-clustering|凝聚层次聚类]] (置信度: high)
