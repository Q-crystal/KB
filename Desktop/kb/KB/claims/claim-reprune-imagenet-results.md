---
type: claim
status: draft
id: claim-reprune-imagenet-results
title: REPrune ImageNet实验结果
confidence: high
source: Park2024-REPrune
tags:
  - type/claim
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition
REPrune在ImageNet/ResNet上的实验表明：在较低FLOPs下，REPrune精度掉点很小甚至提升，并且可在单阶段训练中完成，无需传统的train-prune-finetune三阶段。

# Rigour
- **实验设置**:
  - 数据集: ImageNet
  - 模型: ResNet-18, ResNet-50
  - 指标: Top-1/Top-5准确率, FLOPs
- **关键结果**:
  - 低FLOPs下精度掉点很小甚至提升
  - 单阶段训练完成（对比传统三阶段）
  - 优于DMCP等对比方法
- **COCO/SSD300结果**:
  - backbone FLOPs -50%时mAP约等同baseline

# Evidence
- **表格**: Table 1-3 p6 (ImageNet), Table 4 p7 (COCO)
- **对比**: 与DMCP等传统方法对比

# Links
- **支持**: [[concepts/concept-reprune|REPrune]]
- **相关**: [[claims/claim-ward-linkage-superiority|Ward linkage优势]]
