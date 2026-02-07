---
type: engineering
status: draft
id: engineering-resnet-pruning-constraints
title: ResNet剪枝约束
confidence: high
source: Park2024-REPrune
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition
ResNet的残差连接要求维度匹配。REPrune通过特定的剪枝策略避免skip-add时的通道不匹配：BasicBlock只剪第一个3×3；Bottleneck只剪前两层（1×1与3×3），不剪最后1×1。

# Rigour
- **BasicBlock策略**:
  - 只剪第一个3×3卷积
  - 保持第二个3×3和skip连接不变
- **Bottleneck策略**:
  - 剪第一个1×1和3×3
  - 不剪最后1×1（保持输出维度匹配shortcut）
- **原因**: 避免skip-add时的张量维度不匹配

# Evidence
- **图示**: Fig.5-6 p10
- **附录**: A.6 p10详细说明

# Links
- **使用**: [[concepts/concept-reprune|REPrune]]
- **前提**: [[concepts/concept-resnet|ResNet架构]]
