---
type: claim
status: draft
id: claim-reprune-imagenet-results
title: REPrune ImageNet 实验结果
confidence: high
source: Park2024-REPrune
tags:
  - type/claim
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition

REPrune 在 **ImageNet/ResNet** 上的实验表明：

- 在较低 FLOPs 下，精度掉点很小**甚至提升**
- 可在**单阶段训练**中完成
- 无需传统的 train-prune-finetune **三阶段**

---

# Rigour

## 实验设置

| 配置 | 详情 |
|------|------|
| 数据集 | ImageNet-1K |
| 模型 | ResNet-18, ResNet-50 |
| 指标 | Top-1/Top-5 准确率, FLOPs |

## 关键结果

### ImageNet/ResNet

| 方法 | FLOPs 减少 | 精度变化 |
|------|-----------|---------|
| REPrune | 50% | 掉点很小或**提升** |
| 传统三阶段 | 50% | 需要 finetune 恢复 |

### COCO/SSD300

| 指标 | 结果 |
|------|------|
| Backbone FLOPs | -50% |
| mAP | **约等同 baseline** |
| 对比 | 优于 DMCP |

## 核心优势

| 特性 | REPrune | 传统方法 |
|------|---------|---------|
| 训练阶段 | **1 阶段** | 3 阶段 (train-prune-finetune) |
| 总训练时间 | **更短** | 更长 |
| 精度保持 | **更好** | 需要额外恢复 |
| 超参数调优 | **端到端** | 分阶段调参 |

---

# Evidence

| 来源 | 内容 |
|------|------|
| Table 1-3 p6 | ImageNet/ResNet 详细实验结果 |
| Table 4 p7 | COCO/SSD300 目标检测实验 |
| 对比实验 | 与 DMCP 等传统方法对比 |

---

# Links

- **支持**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **相关**: [[claims/claim-ward-linkage-superiority|Ward linkage 优势]] (置信度: medium)
- **对比**: [[concepts/concept-dmcp|DMCP 方法]] (置信度: medium)
