---
type: claim
status: draft
id: claim-reprune-experiments-detailed
title: REPrune 详细实验结果与分析
confidence: high
source: Park2024-REPrune
tags:
  - type/claim
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/experiments
---

# Intuition

REPrune 在多个标准数据集和模型上进行了全面评估，包括 **ImageNet**、**CIFAR-10**、**COCO** 等。实验结果表明 REPrune 在保持高精度的同时实现了显著的 FLOPs 减少。

---

# Rigour

## 实验设置

### 数据集

| 数据集 | 训练集 | 验证集 | 类别数 | 任务 |
|--------|--------|--------|--------|------|
| ImageNet-1K | 1.27M | 50K | 1,000 | 图像分类 |
| CIFAR-10 | 50K | 10K | 10 | 图像分类 |
| COCO-2017 | 118K | 5K | 80 | 目标检测 |
| COCO-2014 | 83K | 41K | 80 | 实例分割 |

### 模型架构

- **ResNet**: ResNet-18, ResNet-34, ResNet-50, ResNet-56
- **MobileNetV2**: 轻量级模型
- **SSD300**: 目标检测器（ResNet-50 作为 backbone）
- **Mask R-CNN**: 实例分割（ResNet-50-FPN 作为 backbone）

### 训练环境

| 配置 | 详情 |
|------|------|
| GPU | NVIDIA RTX A6000 × 8 |
| 框架 | PyTorch |
| 策略 | NVIDIA DeepLearningExamples |
| 精度 | Mixed Precision |

---

## ImageNet 结果 (Table 1)

### ResNet-18

| 方法 | PT? | FLOPs | Top-1 | Epochs |
|------|-----|-------|-------|--------|
| Baseline | - | 1.81G | 69.4% | - |
| SFP | ✓ | 1.05G | 67.1% | 200 |
| FPGM | ✓ | 1.05G | 68.4% | 200 |
| DMCP | ✗ | 1.04G | 69.0% | 150 |
| **REPrune** | ✗ | **1.03G** | **69.2%** | 250 |

**分析**:
- FLOPs 减少: 43% (1.81G → 1.03G)
- 精度: 仅下降 0.2% (69.4% → 69.2%)
- 优于 DMCP (+0.2%) 和 SFP (+2.1%)
- 无需预训练 (PT? = ✗)

### ResNet-34

| FLOPs | Top-1 基线 | Top-1 REPrune | 变化 |
|-------|-----------|---------------|------|
| 2.7G | 73.3% | 74.3% | **+1.0%** |
| 2.0G | 73.3% | 73.9% | **+0.6%** |

**关键发现**: 在 2.7G FLOPs 下，REPrune **超过基线 1.0%**！

### ResNet-50

| FLOPs | Top-1 基线 | Top-1 REPrune | 变化 |
|-------|-----------|---------------|------|
| 2.3G | 76.2% | 77.2% | **+1.0%** |
| 1.8G | 76.2% | 77.1% | **+0.9%** |
| 1.0G | 76.2% | 75.7% | -0.5% |

---

## CIFAR-10 结果 (Table 2)

### ResNet-56

| 方法 | FLOPs 减少 | 基线 Top-1 | 剪枝后 Top-1 | 变化 |
|------|-----------|-----------|-------------|------|
| CUP | 52.83% | 93.67% | 93.36% | -0.31% |
| LSC | 55.45% | 93.39% | 93.16% | -0.23% |
| ACP | 54.42% | 93.18% | 93.39% | +0.21% |
| **REPrune** | **60.38%** | **93.39%** | **93.40%** | **+0.01%** |

**优势**:
- 更高的 FLOPs 减少率 (60.38% vs 54.42%)
- 精度持平甚至略升
- 训练时间更短 (160 epochs vs 360-1160 epochs)

---

## 与核剪枝方法对比 (Table 3)

### ResNet-56 on CIFAR-10

| 方法 | FLOPs 减少 | 基线 Top-1 | 剪枝后 Top-1 | 变化 |
|------|-----------|-----------|-------------|------|
| TMI-GKP | 43.23% | 93.78% | 94.00% | +0.22% |
| **REPrune** | **47.57%** | **93.39%** | **94.00%** | **+0.61%** |

### ResNet-50 on ImageNet

| 方法 | FLOPs 减少 | 基线 Top-1 | 剪枝后 Top-1 | 变化 |
|------|-----------|-----------|-------------|------|
| TMI-GKP | 33.74% | 76.15% | 75.53% | -0.62% |
| **REPrune** | **56.09%** | **76.20%** | **77.04%** | **+0.84%** |

**结论**: REPrune 在更高的 FLOPs 减少率下实现了更好的精度保持。

---

## 目标检测 (COCO-2017) - Table 4

### SSD300 with ResNet-50 backbone

| FLOPs 减少 | mAP (基线) | mAP (REPrune) | 变化 |
|-----------|-----------|--------------|------|
| 50% | 25.7 | 25.5 | -0.2 |

- Backbone FLOPs 减少 50%
- mAP 仅下降 0.2
- 优于 DMCP (+0.9)

---

## 实例分割 (COCO-2014) - Appendix A.4

### Mask R-CNN with ResNet-50-FPN

| 指标 | 基线 | REPrune (30% FLOPs↓) | 变化 |
|------|------|---------------------|------|
| AP (检测) | - | - | -0.8 |
| AP (分割) | - | - | -0.4 |

**分析**: 即使对于多任务学习（分类 + 检测 + 分割），REPrune 也能有效工作。

---

## 训练时间效率 (Table 6)

### ResNet-50 on ImageNet

| FLOPs 减少 | 训练时间 | 节省时间 |
|-----------|---------|---------|
| 0% (基线) | 43.75h | - |
| 43% | 40.62h | 3.13h (7.2%) |
| 56% | 38.75h | 5.00h (11.4%) |
| 75% | 35.69h | 8.06h (18.4%) |

**结论**: 更高的剪枝率 → 更少的计算 → 更短的训练时间。

---

## 推理吞吐量 (Fig. 4)

### RTX A6000

| FLOPs 减少 | Batch=1 | Batch=16 | Batch=32 |
|-----------|---------|----------|----------|
| 0% | 基准 | 基准 | 基准 |
| 43% | +15% | +25% | +30% |
| 56% | +25% | +40% | +48% |
| 75% | +40% | +65% | +78% |

### Jetson TX2 (边缘设备)

- 实际加速比略低于理论值（由于 I/O 延迟、上下文切换等）
- 但仍保持显著加速

---

# Evidence

| 来源 | 内容 |
|------|------|
| Table 1 p6 | ImageNet 主要结果 |
| Table 2 p6 | 与聚类方法对比 |
| Table 3 p6 | 与核剪枝方法对比 |
| Table 4 p7 | COCO 目标检测 |
| Table 6 p7 | 训练时间效率 |
| Fig. 4 p7 | 推理吞吐量 |
| Appendix A.4 | 实例分割实验 |

---

# Links

- **支持**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **相关**: [[claims/claim-reprune-imagenet-results|ImageNet 实验结果]] (置信度: high)
- **相关**: [[engineering/engineering-reprune-training-settings|训练设置]] (置信度: high)
- **对比**: [[methods/method-dmcp|DMCP]] (置信度: medium)
- **对比**: [[methods/method-tmi-gkp|TMI-GKP]] (置信度: medium)
