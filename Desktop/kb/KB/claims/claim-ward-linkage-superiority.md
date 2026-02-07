---
type: claim
status: draft
id: claim-ward-linkage-superiority
title: Ward linkage优于其他linkage方法
confidence: medium
source: Park2024-REPrune
tags:
  - type/claim
  - status/draft
  - confidence/medium
  - source/Park2024-REPrune
---

# Intuition
在相同全局稀疏度s̄下，Ward linkage比single/complete/average linkage更能保持精度。作者推测：其他linkage方法更偏向前层大FLOPs、信息敏感通道的删除。

# Rigour
- **实验对比**: Table 5 p7
- **假设解释**: 
  - Ward避免簇内部过分发散
  - 其他linkage可能过度合并重要通道
  - 单调性让阈值切分更一致
- **置信度**: medium（需要更多验证）

# Evidence
- **表格**: Table 5 p7
- **解释**: p7文字说明

# Links
- **支持**: [[methods/method-kernel-clustering-ward|核聚类(Ward)]]
- **相关**: [[concepts/concept-reprune|REPrune]]
