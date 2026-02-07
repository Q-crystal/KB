---
type: method
status: draft
id: method-maximum-cluster-coverage
title: 最大簇覆盖问题 (MCP)
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition

将**滤波器选择**表述为最大覆盖问题：

- 每个 filter 覆盖若干 cluster
- **贪心选择**使未覆盖 cluster 数最大化的 filter
- 直到保留目标数量的 filters

这确保选择的 filters 能**代表所有重要的核模式**。

---

# Rigour

## 定义

给定簇集合和所有 filters 的覆盖关系，选择 $\lceil(1-s_l)n_{l+1}\rceil$ 个 filters 使得覆盖的簇最大化。

## 算法 (Alg.1)

```
输入: 簇集合 C, filters集合 F, 目标数量 k
输出: 选中的 filters 集合 S

1. S ← ∅
2. while |S| < k:
3.     f* ← argmax_{f∈F\S} |Cover(f) ∩ (C \ Cover(S))|
4.     S ← S ∪ {f*}
5. return S
```

## 复杂度分析

| 指标 | 值 | 说明 |
|------|---|------|
| 时间复杂度 | $O(k \cdot n)$ | $k$=目标filter数, $n$=总filter数 |
| 空间复杂度 | $O(|C|)$ | 覆盖状态存储 |
| 近似比 | $1 - 1/e \approx 63\%$ | 贪心算法理论保证 |

## 关键性质

- **子模性**: 覆盖函数具有子模性
- **单调性**: 添加filter不会减少覆盖
- **贪心最优**: 在多项式时间内达到最优近似

---

# Evidence

| 来源 | 内容 |
|------|------|
| Alg.1 p4-5 | 贪心算法伪代码 |
| Eq.(6)(7) p4 | 数学 formulation |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **相似**: [[concepts/concept-set-cover|集合覆盖问题]] (置信度: high)
- **相关**: [[methods/method-greedy-algorithm|贪心算法]] (置信度: medium)
