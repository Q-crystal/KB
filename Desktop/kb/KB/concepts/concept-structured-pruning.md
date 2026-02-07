---
type: concept
status: draft
id: concept-structured-pruning
title: 结构化剪枝 (Structured Pruning)
categories:
  - model-compression
  - structured-pruning
confidence: high
source: general-knowledge
tags:
  - type/concept
  - status/draft
  - confidence/high
  - domain/structured-pruning
  - domain/model-compression
---

# Intuition

结构化剪枝通过移除完整的结构单元（如整个滤波器、通道或层），产生规则的稀疏网络结构。这种规则性使得剪枝后的模型可以直接在标准硬件上高效运行，无需特殊的稀疏计算支持。

类比：就像从一支军队中整建制地裁撤某些连队，而不是从每个连队中随机裁撤士兵，这样剩余部队仍能保持完整的指挥结构和作战能力。

---

# Rigour

## 定义

结构化剪枝是指以**结构单元**为单位进行剪枝，移除的参数在原始网络中形成规则的几何模式（如整行、整列或整个通道），从而保持网络的密集连接特性。

## 剪枝单元层级

### 1. 滤波器剪枝 (Filter Pruning)
- **对象**: 整个卷积滤波器（输出通道）
- **效果**: 减少输出通道数
- **影响**: 当前层输出通道↓，下一层输入通道↓
- **代表**: Network Slimming, ThiNet

### 2. 通道剪枝 (Channel Pruning)
- **对象**: 输入通道
- **效果**: 减少输入通道数
- **影响**: 当前层输入通道↓，前一层输出通道↓
- **代表**: Channel Pruning (He et al. 2017)

### 3. 层剪枝 (Layer Pruning)
- **对象**: 整个网络层
- **效果**: 完全移除某些层
- **影响**: 网络深度↓
- **挑战**: 残差连接处理困难
- **代表**: LayerPrune

### 4. 块剪枝 (Block Pruning)
- **对象**: 残差块或整个模块
- **效果**: 移除多个连续层
- **适用**: ResNet等残差网络
- **代表**: DBP (Dense-to-Sparse Block Pruning)

## 重要性评估方法

| 方法 | 描述 | 公式 | 代表 |
|------|------|------|------|
| **L1范数** | 滤波器权重绝对值之和 | $\sum |w|$ | Li et al. 2017 |
| **L2范数** | 滤波器权重平方和开方 | $\sqrt{\sum w^2}$ | He et al. 2018 |
| **几何中位数** | 滤波器到几何中心的距离 | $\|\mathbf{f} - \mathbf{f}_{geo}\|$ | He et al. 2019 (FPGM) |
| **BN缩放因子** | BatchNorm的γ参数 | $|\gamma|$ | Liu et al. 2017 |
| **特征图秩** | 特征图的平均秩 | $\text{rank}(\mathbf{H})$ | Lin et al. 2020 (HRank) |

## 优势与劣势

### 优势 ✅
- **硬件友好**: 直接加速，无需特殊库
- **内存连续**: 权重存储连续，缓存友好
- **部署简单**: 标准框架直接支持
- **推理加速**: 实际FLOPs减少 = 理论减少

### 劣势 ❌
- **压缩率有限**: 通常只能减少2-10倍
- **精度损失**: 相比非结构化剪枝损失更大
- **灵活性低**: 受网络结构约束
- **层间依赖**: 剪枝一层影响相邻层

## 关键挑战

### 1. 层间依赖处理
```
Layer 1: [64, 3, 3, 3] → 剪枝后 [32, 3, 3, 3]
Layer 2: [128, 64, 3, 3] → 必须变为 [128, 32, 3, 3]
```

### 2. 残差连接处理
- **问题**: 残差块输入输出通道数必须匹配
- **解决方案**:
  - 只剪枝块内部分层（如REPrune）
  - 使用投影层调整维度
  - 同步剪枝多个相关层

### 3. 剪枝比例分配
- **问题**: 不同层对剪枝敏感度不同
- **策略**:
  - 均匀剪枝: 所有层相同比例
  - 自适应剪枝: 根据敏感度调整
  - 搜索-based: 自动寻找最优配置

---

# Evidence

## 代表性方法

| 年份 | 方法 | 核心思想 | 特点 |
|------|------|---------|------|
| 2017 | Network Slimming | BN γ参数 | 训练时联合优化 |
| 2017 | ThiNet | 贪心通道选择 | 基于下一层统计 |
| 2018 | SFP | Soft Filter Pruning | 逐步剪枝 |
| 2019 | FPGM | 几何中位数 | 更鲁棒的度量 |
| 2020 | HRank | 特征图秩 | 基于激活值 |
| 2022 | CHEX | 通道探索 | 自动恢复机制 |
| 2024 | REPrune | 核聚类 | 模拟核剪枝效果 |

## 实验结果对比 (ImageNet ResNet-50)

| 方法 | FLOPs↓ | Top-1 Acc | 类型 |
|------|--------|-----------|------|
| Baseline | 0% | 76.1% | - |
| Network Slimming | 50% | ~75% | 通道剪枝 |
| ThiNet | 50% | ~74% | 通道剪枝 |
| SFP | 50% | ~74% | 滤波器剪枝 |
| FPGM | 50% | ~75% | 滤波器剪枝 |
| REPrune | 56% | 77.0% | 通道剪枝 |

---

# Links

## 核心概念
- **泛化**: [[concepts/concept-neural-network-pruning-overview|神经网络剪枝概述]] (置信度: high)
- **对比**: [[concepts/concept-unstructured-pruning|非结构化剪枝]] (置信度: high)
- **对比**: [[concepts/concept-semi-structured-pruning|半结构化剪枝]] (置信度: high)

## 具体方法
- **使用**: [[methods/method-network-slimming|Network Slimming]] (置信度: high)
- **使用**: [[methods/method-fpgm|FPGM]] (置信度: high)
- **使用**: [[methods/method-hrank|HRank]] (置信度: high)
- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)

## 工程实践
- **使用**: [[engineering/engineering-torch-pruning|Torch-Pruning]] (置信度: high)
- **使用**: [[engineering/engineering-channel-pruning-impl|通道剪枝实现]] (置信度: medium)
