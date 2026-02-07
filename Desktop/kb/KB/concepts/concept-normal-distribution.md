---
type: concept
status: stable
id: concept-normal-distribution
title: 正态分布
confidence: high
source: general-knowledge
tags:
  - type/concept
  - status/stable
  - confidence/high
  - source/general-knowledge
---

# Intuition
正态分布是一种对称的钟形概率分布，由均值和标准差完全描述。例如，人类身高通常服从正态分布。

# Rigour
- **定义**: 正态分布的概率密度函数为 f(x) = (1/(σ√(2π))) * e^(-(x-μ)²/(2σ²))
- **假设**: 数据来自正态分布总体
- **适用范围**: 统计推断、假设检验、质量管理
- **代价或复杂度**: 计算简单，参数估计容易
- **失败模式**: 数据不服从正态分布时使用会导致错误结果

# Evidence
正态分布是统计学中最重要的分布之一，中心极限定理为其广泛应用提供了理论基础。

# Links
- **特化**: [[concepts/concept-probability-distribution|概率分布]] (置信度: high)
