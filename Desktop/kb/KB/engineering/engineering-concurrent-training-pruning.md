---
type: engineering
status: draft
id: engineering-concurrent-training-pruning
title: 并发训练-剪枝(Concurrent Training-Pruning)
confidence: high
source: Park2024-REPrune
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition
将REPrune嵌入到训练过程中，使用BN的γ参数全局分位数阈值得到逐层稀疏度sl，每隔ΔT步触发一次选择+regrowth。这样可以在单阶段训练中完成剪枝，无需传统的train-prune-finetune三阶段。

# Rigour
- **机制**:
  - 全局稀疏度目标: s̄
  - BN γ分位数: Q(s̄; Γ) (Eq.8)
  - 逐层稀疏度: sl = 1 - Q(s̄; Γ) / γ_l (Eq.9)
  - 触发间隔: 每ΔT步
- **Regrowth机制**: 当sl=1时（整层删除），引入channel regrowth避免无法聚类
- **优势**: 单阶段训练，减少总训练时间

# Evidence
- **算法**: Alg.2 p5
- **公式**: Eq.(8)(9) p5
- **对比**: 优于传统三阶段方法

# Links
- **使用**: [[concepts/concept-reprune|REPrune]]
- **前提**: [[methods/method-bn-sparsity|BN稀疏度调度]]
