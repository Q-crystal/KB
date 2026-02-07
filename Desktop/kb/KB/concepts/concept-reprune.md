---
type: concept
status: draft
id: concept-reprune
title: REPrune - 用通道剪枝模拟核剪枝
confidence: high
source: Park2024-REPrune
tags:
  - type/concept
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/model-compression
  - domain/channel-pruning
---

# Intuition
REPrune通过通道剪枝(channel pruning)来模拟核剪枝(kernel pruning)的效果。传统核剪枝虽然粒度更细，但会产生稀疏连接，需要专门的硬件/软件支持。REPrune的创新在于：在每个输入通道内对核(kernel)进行聚类，然后挑选能覆盖这些聚类代表的滤波器(filters)，从而在保持密集连接(dense-narrow)的同时，实现对关键核模式的保留，直接获得加速效果。

类比：不是把每个滤波器里的3×3小块挖空（稀疏），而是从一堆"相似小块"里挑"代表性最强的那几个滤波器整体留下"。

# Rigour
- **定义**: REPrune是一种结构化剪枝方法，通过核层面的聚类和代表性选择，用通道剪枝实现类似核剪枝的精度保持，同时保持密集计算图以便直接部署加速。
- **假设**: 
  - 关键核模式可以通过选择代表性滤波器来保留
  - 同一输入通道内的核具有可比性
  - Ward聚类能有效识别核的相似性结构
- **适用范围**: 
  - CNN模型压缩（ResNet、VGG等）
  - 需要结构化剪枝的场景（边缘设备部署）
  - 单阶段训练-剪枝联合优化
- **代价或复杂度**: 
  - 额外计算：核聚类 + 最大覆盖问题求解
  - 内存开销：存储聚类结构和距离矩阵
  - 选择随机性：同分候选时随机采样
- **失败模式**: 
  - 若关键核模式无法通过"选滤波器"来覆盖（簇分布不利），模拟效果受限
  - 当目标稀疏度sl=1时（整层删除），聚类无法进行
  - 残差连接维度不匹配（需特殊处理）

# Evidence
- **动机**: Fig.1展示核剪枝的关键模式与REPrune通过选滤波器间接保留关键核的对齐关系
- **目标阐述**: p2明确提出"channel pruning emulating kernel pruning"的目标
- **实验验证**: ImageNet/ResNet在较低FLOPs下精度掉点很小甚至提升（Table 1-3, p6）
- **对比优势**: 优于传统train-prune-finetune三阶段，可在单阶段训练中完成

# Links
- **使用**: [[methods/method-kernel-clustering-ward|核聚类(Ward)]] (置信度: high)
- **使用**: [[methods/method-layer-wise-cutoff|层-wise截断]] (置信度: high)
- **使用**: [[methods/method-maximum-cluster-coverage|最大簇覆盖问题]] (置信度: high)
- **使用**: [[engineering/engineering-concurrent-training-pruning|并发训练-剪枝]] (置信度: high)
- **支持**: [[claims/claim-reprune-imagenet-results|REPrune ImageNet实验结果]] (置信度: high)
- **相似**: [[concepts/concept-channel-pruning|通道剪枝]] (置信度: medium)
- **衍生**: [[concepts/concept-kernel-pruning|核剪枝]] (置信度: high)
