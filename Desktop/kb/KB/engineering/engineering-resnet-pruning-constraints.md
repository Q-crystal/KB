---
type: engineering
status: draft
id: engineering-resnet-pruning-constraints
title: ResNet 剪枝约束
confidence: high
source: Park2024-REPrune
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition

ResNet 的**残差连接**要求维度匹配，否则无法进行 skip-add 操作。REPrune 通过特定的剪枝策略避免这个问题。

---

# Rigour

## BasicBlock 策略

结构：`Conv3x3 → BN → ReLU → Conv3x3 → BN → Add`

| 层 | 剪枝策略 | 原因 |
|---|---------|------|
| 第一个 Conv3x3 | ✅ **可以剪枝** | 主要计算层 |
| 第二个 Conv3x3 | ❌ **保持不动** | 保持输出维度匹配 shortcut |
| Shortcut | ❌ **保持不动** | 残差连接需要维度一致 |

## Bottleneck 策略

结构：`Conv1x1 → BN → ReLU → Conv3x3 → BN → ReLU → Conv1x1 → BN → Add`

| 层 | 剪枝策略 | 原因 |
|---|---------|------|
| 第一个 Conv1x1 | ✅ **可以剪枝** | 降维层，可压缩 |
| Conv3x3 | ✅ **可以剪枝** | 主要计算层 |
| 最后一个 Conv1x1 | ❌ **不剪枝** | **保持输出维度匹配 shortcut** |
| Shortcut | ❌ **保持不动** | 残差连接需要维度一致 |

## 核心原则

> **关键约束**：最后一个卷积层的输出通道数必须与 shortcut 的通道数相同，否则无法进行 element-wise add。

---

# Evidence

| 来源 | 内容 |
|------|------|
| Fig.5-6 p10 | 图示说明 BasicBlock 和 Bottleneck 的剪枝策略 |
| 附录 A.6 p10 | 详细的技术说明和实现细节 |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[concepts/concept-resnet|ResNet 架构]] (置信度: high)
- **相关**: [[concepts/concept-residual-connection|残差连接]] (置信度: high)
