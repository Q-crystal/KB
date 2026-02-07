---
type: concept
status: draft
id: concept-channel-pruning-survey
title: 通道剪枝方法综述
confidence: high
source: Park2024-REPrune
tags:
  - type/concept
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/survey
  - domain/channel-pruning
---

# Intuition

通道剪枝（Channel Pruning）是神经网络压缩的主流方法之一，通过在训练过程中识别并移除不重要的通道（滤波器），实现模型加速。不同方法在**重要性评估**、**剪枝策略**、**训练范式**等方面各有特点。

---

# Rigour

## 方法分类

### 1. 基于正则化的方法

**核心思想**: 通过LASSO或Group LASSO惩罚通道重要性

| 方法 | 特点 | 局限 |
|------|------|------|
| Liu et al. 2017 | Network Slimming，使用BN的γ参数 | 难以精确零化滤波器 |
| He et al. 2017 | 基于LASSO的通道选择 | 需要预训练模型 |
| You et al. 2019 | Gate Decorator，全局滤波器剪枝 | 训练不稳定 |

**共同局限**: 正则化难以强制通道精确为零，需要额外的阈值处理。

### 2. 基于代理问题的方法

**核心思想**: 使用ISTA等优化算法诱导稀疏性

| 方法 | 特点 |
|------|------|
| Ye et al. 2018 | ISTA-based，可精确零化 |
| Lin et al. 2019 | 生成对抗学习优化 |

**优势**: 可以精确控制稀疏性
**劣势**: 需要额外的优化步骤

### 3. 基于重要性的方法

**核心思想**: 通过排名、范数、独立性等度量评估通道重要性

| 方法 | 评估指标 | 特点 |
|------|---------|------|
| Lin et al. 2020 (HRank) | 特征图高秩 | 基于激活值 |
| He et al. 2018 (SFP) | 几何中位数 | Soft pruning |
| He et al. 2019 (FPGM) | 几何中位数改进 | 更鲁棒 |
| Sui et al. 2021 (CHIP) | 通道独立性 | 互信息度量 |

**共同特点**: 需要额外的微调（fine-tuning）过程

### 4. 基于采样的方法

**核心思想**: 使用可微参数近似通道移除过程

| 方法 | 机制 | 特点 |
|------|------|------|
| Kang & Han 2020 | 可微掩码 | 操作感知 |
| Gao et al. 2020 (DMC) | 离散模型压缩 | 资源约束 |
| Guo et al. 2020 (DMCP) | 可微马尔可夫链 | 自动剪枝 |

**优势**: 端到端训练，自动化程度高
**劣势**: 依赖可微参数，增加训练复杂度

### 5. 基于聚类的方法

**核心思想**: 利用滤波器分布差异识别相似通道

| 方法 | 聚类对象 | 特点 |
|------|---------|------|
| Duggal et al. 2019 (CUP) | 滤波器 | 层间差异 |
| Lee et al. 2020 (LSC) | 潜在空间表示 | 重初始化 |
| Chang et al. 2022 (ACP) | 特征图 | 自动通道数设计 |

**共同特点**: 利用滤波器相似性进行分组

### 6. 核剪枝方法

**核心思想**: 在核（kernel）层面进行剪枝，粒度更细

| 方法 | 策略 | 局限 |
|------|------|------|
| Ma et al. 2020 (PConv) | 模式-based剪枝 | 产生稀疏模型 |
| Niu et al. 2020 (PATDNN) | 模式-based + 负载均衡 | 需要专用硬件支持 |
| Zhong et al. 2022 (TMI-GKP) | 彩票假设 + 分组卷积 | 稀疏性挑战 |

**共同局限**: 产生稀疏连接，需要代码生成或负载均衡技术

---

## REPrune 的定位

| 特性 | REPrune | 传统通道剪枝 | 核剪枝 |
|------|---------|-------------|--------|
| 剪枝粒度 | 通道（filter） | 通道 | 核（kernel） |
| 内部机制 | 核聚类 + 最大覆盖 | 重要性评估 | 模式选择 |
| 输出结构 | Dense-narrow | Dense-narrow | Sparse |
| 硬件友好性 | ✅ 直接部署 | ✅ 直接部署 | ⚠️ 需要优化 |
| 训练范式 | 并发训练-剪枝 | 多阶段 | 多阶段 |

**REPrune 的独特之处**:
1. **核层面的分析** + **通道层面的剪枝** = 兼顾精度和硬件友好性
2. **无需可微参数** - 简化训练过程
3. **单阶段训练** - 无需 train-prune-finetune

---

# Evidence

| 来源 | 内容 |
|------|------|
| Related Work p2-3 | 详细的文献综述 |
| Table 1-3 p6 | 与各类方法的实验对比 |

---

# Links

- **对比**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **相关**: [[methods/method-network-slimming|Network Slimming]] (置信度: medium)
- **相关**: [[methods/method-dmcp|DMCP]] (置信度: medium)
- **相关**: [[methods/method-tmi-gkp|TMI-GKP]] (置信度: medium)
- **相关**: [[concepts/concept-hrank|HRank]] (置信度: medium)
