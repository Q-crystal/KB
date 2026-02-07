---
type: concept
status: draft
id: concept-neural-network-pruning-overview
title: 神经网络剪枝概述
categories: 
  - model-compression
  - neural-network-pruning
confidence: high
source: general-knowledge
tags:
  - type/concept
  - status/draft
  - confidence/high
  - domain/model-compression
  - domain/neural-network-pruning
  - fundamentals
---

# Intuition

神经网络剪枝（Neural Network Pruning）是一种模型压缩技术，通过识别并移除神经网络中"不重要"的参数或结构，在保持模型性能的同时显著减少模型大小和计算量。类比：就像修剪树木的枝叶，去除冗余部分让树木更健康地生长。

核心思想：**并非所有参数都对模型输出同等重要**，移除低重要性参数可以大幅降低计算成本而几乎不影响精度。

---

# Rigour

## 定义

神经网络剪枝是指在训练好的神经网络中，根据某种重要性度量标准，移除部分权重、神经元、通道或层，从而得到一个更小、更快的子网络。

## 剪枝粒度分类

| 粒度级别 | 剪枝对象 | 硬件友好性 | 压缩率 | 精度损失 |
|---------|---------|-----------|--------|---------|
| **细粒度** | 单个权重 | ❌ 需要特殊硬件 | 高 | 低 |
| **模式级** | 权重模式 | ⚠️ 需要支持 | 中高 | 低 |
| **核级** | 卷积核 | ⚠️ 需要支持 | 中 | 中 |
| **通道级** | 整个通道 | ✅ 直接加速 | 中 | 中 |
| **层級** | 整个层 | ✅ 直接加速 | 低 | 高 |

## 剪枝时机分类

### 1. 训练后剪枝 (Post-training Pruning)
- **流程**: 训练完整模型 → 剪枝 → 微调
- **优点**: 简单直接，基于成熟模型
- **缺点**: 需要额外微调，可能损失精度

### 2. 训练中剪枝 (During-training Pruning)
- **流程**: 边训练边剪枝（如REPrune）
- **优点**: 单阶段完成，联合优化
- **缺点**: 训练过程复杂，需要仔细调参

### 3. 训练前剪枝 (Pre-training Pruning)
- **流程**: 初始化时确定结构 → 训练
- **代表**: 彩票假设（Lottery Ticket Hypothesis）
- **优点**: 从头训练稀疏网络
- **缺点**: 需要找到 winning ticket

## 重要性度量方法

| 方法 | 描述 | 代表工作 |
|------|------|---------|
| **基于幅度** | 权重绝对值大小 | Optimal Brain Damage |
| **基于梯度** | 梯度幅值或敏感性 | SNIP, GraSP |
| **基于激活** | 神经元激活统计 | HRank, Network Slimming |
| **基于Hessian** | 二阶导数信息 | Optimal Brain Surgeon |
| **基于学习** | 可学习的掩码 | DMC, DMCP |

## 剪枝策略

### 结构化剪枝 (Structured Pruning)
- 移除整个结构单元（通道、层）
- 输出规则稀疏结构
- **优点**: 硬件友好，直接加速
- **缺点**: 压缩率相对较低

### 非结构化剪枝 (Unstructured Pruning)
- 移除任意位置的权重
- 输出不规则稀疏矩阵
- **优点**: 压缩率高，精度损失小
- **缺点**: 需要专用硬件/库支持

### 半结构化剪枝 (Semi-structured)
- 2:4, 4:8 等固定模式
- 平衡压缩率和硬件效率
- **代表**: NVIDIA Ampere 稀疏张量核心

## 失败模式

- ⚠️ **过度剪枝**: 移除过多重要参数导致精度崩溃
- ⚠️ **层间不平衡**: 某些层过度剪枝影响信息流动
- ⚠️ **微调灾难**: 剪枝后微调导致性能下降
- ⚠️ **硬件不匹配**: 稀疏模式不被目标硬件支持

---

# Evidence

## 历史发展

| 时间 | 里程碑 | 代表工作 |
|------|--------|---------|
| 1989 | 早期剪枝 | LeCun's Optimal Brain Damage |
| 1993 | 二阶方法 | Hassibi's Optimal Brain Surgeon |
| 2015 | 现代复兴 | Han et al. Deep Compression |
| 2017 | 结构化剪枝 | Network Slimming, ThiNet |
| 2019 | 彩票假设 | Frankle & Carbin LTH |
| 2020 | 动态剪枝 | Runtime Neural Pruning |
| 2024 | 自动化剪枝 | AgenticPruner, LLM驱动 |

## 应用领域

- **计算机视觉**: CNN压缩（ResNet, MobileNet）
- **自然语言处理**: Transformer压缩（BERT, GPT）
- **语音处理**: ASR模型压缩
- **推荐系统**: 大规模Embedding压缩
- **边缘设备**: IoT, 移动端部署

---

# Links

## 核心概念
- **特化**: [[concepts/concept-structured-pruning|结构化剪枝]] (置信度: high)
- **特化**: [[concepts/concept-unstructured-pruning|非结构化剪枝]] (置信度: high)
- **特化**: [[concepts/concept-lottery-ticket-hypothesis|彩票假设]] (置信度: high)
- **特化**: [[concepts/concept-dynamic-pruning|动态剪枝]] (置信度: medium)

## 方法分类
- **使用**: [[methods/method-magnitude-based-pruning|基于幅度的剪枝]] (置信度: high)
- **使用**: [[methods/method-gradient-based-pruning|基于梯度的剪枝]] (置信度: high)
- **使用**: [[methods/method-learning-based-pruning|基于学习的剪枝]] (置信度: high)

## 相关领域
- **相关**: [[concepts/concept-model-compression|模型压缩]] (置信度: high)
- **相关**: [[concepts/concept-quantization|量化]] (置信度: high)
- **相关**: [[concepts/concept-knowledge-distillation|知识蒸馏]] (置信度: high)

## 实践工具
- **使用**: [[engineering/engineering-torch-pruning|Torch-Pruning框架]] (置信度: high)
