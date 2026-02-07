---
type: method
status: draft
id: method-layer-wise-cutoff
title: 层-wise截断 hl
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition
不同通道的核分布差异大，使用统一阈值会导致失真。REPrune通过目标稀疏度反推"合理簇数"：先按目标保留通道数计算应有的合并步数，然后取各通道在该步数的最大距离作为hl，保证没有通道被过度合并。

# Rigour
- **定义**: 给定层稀疏度sl，计算聚类截断阈值hl，使得最终保留⌈(1-sl)n_{l+1}⌉个通道。
- **数学表达**:
  - 目标保留: ⌈(1-sl)n_{l+1}⌉个filters
  - 合并步数: ñ_{l+1} = ⌈sl·n_{l+1}⌉
  - 截断阈值: hl = max_j d(ñ_{l+1}; K_j^l)
  - 最终簇: AC*(K_j^l) using hl
- **假设**:
  - 最大距离策略能平衡各通道的聚类程度
  - 稀疏度sl能反映实际的通道重要性分布
- **适用范围**: REPrune的层-wise剪枝决策
- **代价**: 需要计算所有通道的聚类距离

# Evidence
- **公式**: Eq.(4)(5) p4
- **动机**: 避免统一阈值导致的通道间不平衡

# Links
- **使用**: [[concepts/concept-reprune|REPrune]]
- **前提**: [[methods/method-kernel-clustering-ward|核聚类]]
