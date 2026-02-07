---
type: method
status: draft
id: method-reprune-algorithm1
title: REPrune Algorithm 1 - Channel Selection
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/algorithm
---

# Intuition

Algorithm 1 是 REPrune 的核心算法，负责在给定层稀疏度 $s^l$ 的情况下，选择最优的滤波器集合。算法通过**聚类**和**最大覆盖问题**两个步骤，确保选择的滤波器能够代表所有重要的核模式。

---

# Rigour

## 算法输入

| 符号 | 说明 |
|------|------|
| $\mathcal{F}^l = \{\mathcal{K}^l_j : j \in \mathcal{J}\}$ | 第 $l$ 层的滤波器集合，包含所有输入通道的核集合 |
| $\forall l \in \mathcal{L}$ | 对所有卷积层执行 |
| $\mathcal{AC}$ | 凝聚聚类算法 |
| $s^l, \forall l \in \mathcal{L}$ | 每层的通道稀疏度目标 |

## 算法输出

| 符号 | 说明 |
|------|------|
| $\tilde{\mathcal{F}}^l, \forall l \in \mathcal{L}$ | 剪枝后的滤波器集合 |

## 算法流程

```
Algorithm 1: Channel Selection in REPrune

输入: 
  - Filter set ℱ^l = {𝒦^l_j : j ∈ 𝒥}, ∀l ∈ ℒ
  - Non-pruned filter set ℱ̃^l, ∀l ∈ ℒ
  - Agglomerative clustering 𝒜𝒞
  - Target channel sparsity s^l, ∀l ∈ ℒ

输出: 剪枝后的滤波器集合 ℱ̃^l

1:  for layer l ∈ ℒ do
2:      for channel j from 1 to n_l do
3:          执行 𝒜𝒞_c(𝒦^l_j) 直到合并步数 c 达到 ñ_{l+1}
4:          获取 d(ñ_{l+1}; 𝒦^l_j) 并计算截断阈值 h_l
5:          设置每通道簇 𝒜𝒞*(𝒦^l_j)  ▷ Eq. 5
6:      end for
7:      定义最优簇覆盖 U^l
8:      初始化 ℱ̃^l 为空队列
9:      while |ℱ̃^l| < ⌈(1-s^l)n_{l+1}⌉ do
10:         在 ℱ^l 中选择候选滤波器  ▷ Eq. 7
11:         从候选中采样 ℱ^l_r 并加入 ℱ̃^l
12:         更新剩余核的覆盖分数
13:     end while
14: end for
```

## 关键步骤详解

### Step 1-6: 聚类阶段

**目标**: 为每个输入通道生成核簇

**过程**:
1. 对每个输入通道 $j$，对其核集合 $\mathcal{K}^l_j$ 执行凝聚聚类
2. 聚类直到合并步数达到 $\tilde{n}_{l+1} = \lceil s^l n_{l+1} \rceil$
3. 收集 Ward 距离 $d(\tilde{n}_{l+1}; \mathcal{K}^l_j)$
4. 计算层-wise 截断阈值 $h_l = \max_j d(\tilde{n}_{l+1}; \mathcal{K}^l_j)$
5. 使用 Eq. 5 确定每通道的最终簇 $\mathcal{AC}^*(\mathcal{K}^l_j)$

### Step 7: 定义覆盖集合

$$U^l = \{\mathcal{AC}^*(\mathcal{K}^l_1); \cdots; \mathcal{AC}^*(\mathcal{K}^l_{n_l})\}$$

表示需要被覆盖的所有簇的集合。

### Step 9-13: 滤波器选择阶段

**目标**: 选择能最大化覆盖 $U^l$ 的滤波器

**贪心策略** (Eq. 7):
```
i = argmax_{i ∈ ℐ} Σ_{j ∈ 𝒥} S(κ^l_{i,j}), ∀κ^l_{i,j} ∈ ℱ^l_i
```

**覆盖分数更新**:
- 每个核初始覆盖分数 $S(κ^l_{r,j}) = 1$
- 当某个簇已被覆盖时，该簇中所有核的分数变为 0
- 重复选择直到达到目标滤波器数量 $\lceil(1-s^l)n_{l+1}\rceil$

## 复杂度分析

| 阶段 | 时间复杂度 | 说明 |
|------|-----------|------|
| 聚类阶段 | $O(n_l \cdot n_{l+1}^2 \log n_{l+1})$ | 每层 $n_l$ 个通道，每通道聚类 |
| 选择阶段 | $O(k \cdot n_{l+1} \cdot n_l)$ | $k$ 为目标滤波器数 |
| 总复杂度 | $O(|

\mathcal{L}

| \cdot (n_{max}^3))$ | 所有层 |

---

# Evidence

| 来源 | 内容 |
|------|------|
| Alg.1 p4-5 | 原始算法伪代码 |
| Eq.5 | 簇选择公式 |
| Eq.7 | 贪心选择公式 |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[methods/method-kernel-clustering-ward|核聚类 (Ward)]] (置信度: high)
- **前提**: [[methods/method-layer-wise-cutoff|层-wise 截断]] (置信度: high)
- **使用**: [[methods/method-maximum-cluster-coverage|最大簇覆盖问题]] (置信度: high)
- **被调用**: [[methods/method-reprune-algorithm2|REPrune Algorithm 2]] (置信度: high)
