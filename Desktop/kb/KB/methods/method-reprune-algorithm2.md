---
type: method
status: draft
id: method-reprune-algorithm2
title: REPrune Algorithm 2 - Complete Training Pipeline
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

Algorithm 2 是 REPrune 的**完整训练流程**，将剪枝过程集成到正常的 CNN 训练中。它使用 **BN (Batch Normalization) 的 $\gamma$ 参数**来评估通道重要性，并通过**全局分位数阈值**自动计算每层的稀疏度。

核心思想：**边训练边剪枝**，避免传统的 train-prune-finetune 三阶段。

---

# Rigour

## 算法输入

| 符号 | 说明 |
|------|------|
| $\mathcal{M}$ | 带卷积滤波器 $\{\mathcal{F}^l : \forall l \in \mathcal{L}\}$ 的 CNN 模型 |
| $\mathcal{AC}$ | 凝聚聚类算法 |
| $T_{prune}$ | 剪枝的总 epoch 数 |
| $\Delta T$ | 剪枝间隔 epoch |
| $\bar{s}$ | 全局通道稀疏度 |
| $\mathcal{D}$ | 数据集 |

## 算法输出

| 符号 | 说明 |
|------|------|
| 剪枝后的模型 | 带滤波器 $\{\tilde{\mathcal{F}}^l : \forall l \in \mathcal{L}\}$ |

## 完整算法流程

```
Algorithm 2: Overview of REPrune

输入:
  - CNN 模型 ℳ，卷积滤波器 {ℱ^l : ∀l ∈ ℒ}
  - 凝聚聚类算法 𝒜𝒞
  - 剪枝总 epoch 数 T_prune
  - 剪枝间隔 ΔT
  - 全局稀疏度 s̄
  - 数据集 𝒟

输出: 剪枝后的模型

1:  初始化 ℳ
2:  t ← 1  ▷ t 表示 epoch
3:  
4:  while ℳ 未收敛 do
5:      从 𝒟 采样 mini-batch
6:      在 ℳ 上执行梯度下降
7:      
8:      if t mod ΔT = 0 且 t ≤ T_prune then
9:          计算通道稀疏度 {s^l : ∀l ∈ ℒ}  ▷ Eq.9
10:         通过 REPrune 执行通道选择  ▷ Alg.1
11:             ▷ 当 s^l = 1 时，跳过该层
12:         执行通道 regrowth 过程
13:     end if
14:     
15:     t ← t + 1
16: end while
```

## 关键组件详解

### 1. BN γ 参数与稀疏度计算

**全局阈值计算** (Eq.8):

$$Q(\bar{s}; \Gamma) = \inf\{\gamma^l_i \in \Gamma : F(\gamma^l_i) \geq \bar{s}, (i,l) \in (\mathcal{I}, \mathcal{L})\}$$

其中：
- $\Gamma = \{\gamma^l_i : (i,l) \in (\mathcal{I}, \mathcal{L})\}$ 是所有层 BN 缩放因子的集合
- $F(\cdot)$ 是累积分布函数 (CDF)
- $\bar{s}$ 是全局稀疏度目标

**逐层稀疏度计算** (Eq.9):

$$s^l = \frac{1}{|\mathcal{I}|} \sum_{i \in \mathcal{I}} \mathds{1}(\gamma^l_i \leq \gamma^*), \quad l \in \mathcal{L}$$

其中指示函数：

$$
\mathds{1}(\gamma^l_i \leq \gamma^*) = 
\begin{cases}
0, & \text{if } \gamma^l_i \leq \gamma^* \text{ (保留通道)} \\
1, & \text{otherwise (剪枝通道)}
\end{cases}
$$

### 2. 触发机制

**触发条件**:
- 当前 epoch $t$ 满足 $t \mod \Delta T = 0$
- $t \leq T_{prune}$ (在剪枝阶段内)

**典型参数**:
- $T_{prune} = 180$ (ImageNet)
- $\Delta T = 2$ (每 2 个 epoch 剪枝一次)
- 总训练 epoch: 250

### 3. Channel Regrowth 机制

**问题**: 当 $s^l = 1$ 时，表示该层所有通道都要被删除，导致无法进行聚类。

**解决方案**:
- 借鉴 CHEX (Hou et al. 2022) 的通道 regrowth 策略
- 允许恢复之前训练迭代中被剪枝的某些通道
- 确保 REPrune 在训练过程中无缝运行

**触发条件**:
```
if s^l == 1 then
    跳过该层的 REPrune 子程序
    执行 channel regrowth
end if
```

### 4. Hard Pruning 一致性

**问题**: 第 $l$ 层的输入通道数 $n_l$ 与第 $l-1$ 层的输出通道数不匹配。

**解决方案**:
- 使用 Hard Pruning 方法 (He et al. 2018)
- 调整 $n_l$ 以匹配 $\lceil(1-s^{l-1})n_l\rceil$
- 确保层间维度一致性

---

# Evidence

| 来源 | 内容 |
|------|------|
| Alg.2 p5 | 完整算法流程 |
| Eq.8 | 全局阈值计算 |
| Eq.9 | 逐层稀疏度计算 |
| Table 6 | 训练时间效率对比 |

---

# 超参数设置

## ImageNet (ResNet)

| 参数 | 值 | 说明 |
|------|---|------|
| $T_{prune}$ | 180 | 剪枝总 epoch |
| $\Delta T$ | 2 | 剪枝间隔 |
| 总 epoch | 250 | 完整训练 |
| Batch size | 128×8 | 8 GPUs，每 GPU 128 |

## CIFAR-10

| 参数 | 值 | 说明 |
|------|---|------|
| $T_{prune}$ | 96 | 剪枝总 epoch |
| 总 epoch | 160 | 完整训练 |
| Weight decay | 5e-4 | 权重衰减 |
| Batch size | 256 | 批量大小 |

---

# Links

- **调用**: [[methods/method-reprune-algorithm1|REPrune Algorithm 1]] (置信度: high)
- **使用**: [[engineering/engineering-concurrent-training-pruning|并发训练-剪枝]] (置信度: high)
- **相关**: [[methods/method-bn-sparsity|BN 稀疏度调度]] (置信度: high)
- **相关**: [[concepts/concept-channel-regrowth|Channel Regrowth]] (置信度: medium)
