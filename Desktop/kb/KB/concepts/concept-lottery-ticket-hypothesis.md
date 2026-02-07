---
type: concept
status: draft
id: concept-lottery-ticket-hypothesis
title: 彩票假设 (Lottery Ticket Hypothesis, LTH)
categories:
  - model-compression
  - pruning-theory
confidence: high
source: Frankle2019-LTH
tags:
  - type/concept
  - status/draft
  - confidence/high
  - domain/pruning-theory
  - domain/model-compression
---

# Intuition

彩票假设认为：在一个随机初始化的密集神经网络中，存在一个稀疏的子网络（称为"中奖彩票"），当单独训练时，可以达到甚至超过原始网络的性能。就像买彩票，虽然大多数彩票没奖，但存在中奖的彩票。

核心发现：**随机初始化时，某些子网络天生就具有更好的训练潜力**。

---

# Rigour

## 正式定义

> **彩票假设** (Frankle & Carbin, 2019): 
> 一个随机初始化的密集神经网络包含一个子网络，该子网络：
> 1. 初始化时即存在
> 2. 当单独训练时，在最多相同的迭代次数内达到与原始网络相当的测试精度
> 3. 这个子网络称为"中奖彩票" (Winning Ticket)

## 中奖彩票识别流程 (Iterative Magnitude Pruning)

```
1. 随机初始化网络 f(x; θ₀)，θ₀ ~ D_θ
2. 训练网络 j 轮，得到参数 θⱼ
3. 剪枝 p% 的权重，创建掩码 m (保留高幅度权重)
4. 重置剩余参数到初始值：θ = m ⊙ θ₀
5. 重复步骤2-4，直到达到目标稀疏度
```

## 关键发现

### 1. 重置的重要性
- ✅ **重置到初始化**: 中奖彩票可以训练成功
- ❌ **保留训练后值**: 稀疏网络难以训练
- ❌ **随机重新初始化**: 性能大幅下降

### 2. 稀疏度与性能关系

| 稀疏度 | 典型性能 | 说明 |
|--------|---------|------|
| 10-30% | 匹配或超越密集网络 | 最佳"中奖彩票"区域 |
| 50-70% | 接近密集网络 | 仍可找到有效子网络 |
| 90%+ | 性能下降 | 过于稀疏，难以训练 |

### 3. 学习率敏感性
- **高学习率**: LTH 现象更明显
- **低学习率**: 难以发现中奖彩票
- **Warmup**: 有助于稳定早期训练

## 理论解释

### 1. 初始化优势
中奖彩票的初始化值使其：
- 更容易优化
- 更好的梯度流动
- 更适合任务的数据分布

### 2. 与Dropout的联系
- Dropout: 训练时随机丢弃，测试时全部使用
- LTH: 永久丢弃，只训练子网络
- **相似**: 都发现稀疏子网络足够好

### 3. 与神经架构搜索(NAS)的联系
- NAS: 搜索最优架构
- LTH: 从密集网络中"发现"好架构
- **优势**: LTH 计算成本远低于 NAS

## 变体与扩展

### 1. 学习率彩票假设 (Learning Rate LTH)
> 存在特定学习率 schedule 下的中奖彩票

### 2. 数据彩票假设 (Data LTH)
> 特定数据子集足以训练中奖彩票

### 3. 超参数彩票假设
> 特定超参数配置下的中奖彩票

### 4. 线性模式连接 (LMC)
> 不同中奖彩票之间存在线性路径

## 失败模式

- ⚠️ **深度网络困难**: 极深网络（如ResNet-152）难以找到中奖彩票
- ⚠️ **大学习率不稳定**: 大学习率下训练不稳定
- ⚠️ **数据依赖性**: 某些数据集上现象不明显
- ⚠️ **计算成本**: 迭代剪枝需要多次训练

---

# Evidence

## 原始实验 (Frankle & Carbin, ICLR 2019)

### MNIST (LeNet)
- 稀疏度: 3.6% (压缩27倍)
- 性能: 匹配密集网络

### CIFAR-10 (ResNet-18)
- 稀疏度: 10-20%
- 性能: 匹配或超越密集网络

### ImageNet (ResNet-50)
- 稀疏度: 30-50%
- 性能: 接近密集网络

## 后续验证

| 年份 | 工作 | 主要发现 |
|------|------|---------|
| 2019 | Zhou et al. | Deconstructing LTH |
| 2020 | You et al. | LTH in RL |
| 2021 | Chen et al. | LTH in BERT |
| 2022 | Zhang et al. | LTH at scale |
| 2023 | Ma et al. | Theoretical analysis |
| 2025 | Arora & Teuscher | Winning by preserving dynamics |

---

# Links

## 核心概念
- **泛化**: [[concepts/concept-neural-network-pruning-overview|神经网络剪枝概述]] (置信度: high)
- **相关**: [[concepts/concept-early-bird-ticket|Early Bird Ticket]] (置信度: high)
- **相关**: [[concepts/concept-supermask|Supermask]] (置信度: medium)

## 方法
- **使用**: [[methods/method-imp|Iterative Magnitude Pruning]] (置信度: high)
- **相关**: [[methods/method-snip|SNIP]] (置信度: medium)
- **相关**: [[methods/method-grasp|GraSP]] (置信度: medium)

## 主张
- **支持**: [[claims/claim-lth-existence|LTH存在性证明]] (置信度: high)
- **矛盾**: [[claims/claim-lth-limitations|LTH局限性]] (置信度: medium)
