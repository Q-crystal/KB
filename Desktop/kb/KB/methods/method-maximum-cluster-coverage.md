---
type: method
status: draft
id: method-maximum-cluster-coverage
title: 最大簇覆盖问题(MCP)
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition
将滤波器选择表述为最大覆盖问题：每个filter覆盖若干cluster，贪心选择使未覆盖cluster数最大化的filter，直到保留目标数量的filters。这确保选择的filters能代表所有重要的核模式。

# Rigour
- **定义**: 给定簇集合和所有filters的覆盖关系，选择⌈(1-sl)n_{l+1}⌉个filters使得覆盖的簇最大化。
- **算法**: 贪心算法(Alg.1)
  - 每次选择覆盖最多未覆盖簇的filter
  - 直到达到目标filter数量
- **复杂度**: O(k·n) where k=目标filter数, n=总filter数
- **近似比**: 贪心算法达到(1-1/e)近似比

# Evidence
- **算法**: Alg.1 p4-5
- **公式**: Eq.(6)(7) p4

# Links
- **使用**: [[concepts/concept-reprune|REPrune]]
- **相似**: [[concepts/concept-set-cover|集合覆盖问题]]
