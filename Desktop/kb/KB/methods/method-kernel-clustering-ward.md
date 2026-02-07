---
type: method
status: draft
id: method-kernel-clustering-ward
title: 核聚类(Ward Linkage)
confidence: high
source: Park2024-REPrune
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/clustering
  - domain/ward-linkage
---

# Intuition
在REPrune中，对每个输入通道内的所有核(kernel)进行凝聚层次聚类。固定输入通道j时，该通道对应的所有输出通道的核κ_{i,j}^l都是同尺寸（如3×3或1×1），可以公平比较相似度。使用Ward linkage计算聚类距离，它用"合并后SSE增量"作为距离度量，具有单调递增特性，便于用阈值控制聚类停止。

# Rigour
- **定义**: 对每个输入通道j，将其对应的核集合K_j^l ∈ R^{n_{l+1}×k_h×k_w}进行凝聚层次聚类，使用Ward linkage计算距离。
- **数学表达**:
  - Ward距离: I(A_c, B_c) = \frac{|A_c||B_c|}{|A_c|+|B_c|} \|m_{A_c} - m_{B_c}\|^2
  - Linkage目标: d(c; K_j^l) = \min I(A_c, B_c) 随c单调不减
  - 控制函数: 当d(c)超过阈值时停止聚类（Eq.3）
- **假设**:
  - 同一输入通道内的核具有可比性
  - Ward距离能有效衡量核的相似性
  - 单调性使"阈值切分"更一致
- **适用范围**:
  - CNN卷积层的核分析
  - 需要识别核模式相似性的场景
  - 与REPrune剪枝策略配合使用
- **代价或复杂度**:
  - 时间复杂度: O(n^2 log n) for n kernels per channel
  - 空间复杂度: O(n^2) for distance matrix
  - 每层需要执行n_l次聚类（每个输入通道一次）
- **失败模式**:
  - 核分布过于分散时聚类效果差
  - 当n_{l+1}=1时无法聚类（只有一个核）
  - 阈值选择不当导致过度或不足合并

# Evidence
- **符号定义**: p3详细定义了核集合K_j^l和聚类过程
- **Ward推导**: 附录p8提供Ward距离的数学推导
- **对比实验**: Table 5 (p7)显示Ward linkage比single/complete/average linkage更能保持精度
- **作者解释**: Ward linkage避免簇内部过分发散，且单调性让"阈值切分"更一致

# Links
- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[methods/method-layer-wise-cutoff|层-wise截断]] (置信度: high)
- **支持**: [[claims/claim-ward-linkage-superiority|Ward linkage优势]] (置信度: medium)
- **相似**: [[concepts/concept-agglomerative-clustering|凝聚层次聚类]] (置信度: high)
