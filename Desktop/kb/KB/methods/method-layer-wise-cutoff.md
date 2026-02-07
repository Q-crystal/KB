---
type: method
status: draft
id: method-layer-wise-cutoff
title: 层-wise截断 $h_l$
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition

不同通道的核分布差异大，使用统一阈值会导致失真。REPrune通过**目标稀疏度反推**"合理簇数"：

1. 按目标保留通道数计算应有的合并步数
2. 取各通道在该步数的**最大距离**作为 $h_l$
3. 保证没有通道被过度合并

---

# Rigour

## 定义

给定层稀疏度 $s_l$，计算聚类截断阈值 $h_l$，使得最终保留 $\lceil(1-s_l)n_{l+1}\rceil$ 个通道。

## 数学表达

**目标保留**：

$$\text{保留数量} = \lceil(1-s_l)n_{l+1}\rceil \text{ 个 filters}$$

**合并步数**：

$$\tilde{n}_{l+1} = \lceil s_l \cdot n_{l+1}\rceil$$

**截断阈值**：

$$h_l = \max_j d(\tilde{n}_{l+1}; K_j^l)$$

**最终簇**：使用 $h_l$ 得到 $AC^*(K_j^l)$

## 假设

- 最大距离策略能平衡各通道的聚类程度
- 稀疏度 $s_l$ 能反映实际的通道重要性分布

## 适用范围

- REPrune的层-wise剪枝决策
- 需要自适应阈值的多通道场景

## 代价

- 需要计算所有通道的聚类距离
- 额外 $O(n_l)$ 次距离查询

---

# Evidence

| 来源 | 内容 |
|------|------|
| Eq.(4)(5) p4 | 公式定义 |
| 动机说明 | 避免统一阈值导致的通道间不平衡 |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[methods/method-kernel-clustering-ward|核聚类]] (置信度: high)
