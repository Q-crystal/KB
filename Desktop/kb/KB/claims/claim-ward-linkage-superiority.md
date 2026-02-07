---
type: claim
status: draft
id: claim-ward-linkage-superiority
title: Ward Linkage 优于其他 Linkage 方法
confidence: medium
source: Park2024-REPrune
tags:
  - type/claim
  - status/draft
  - confidence/medium
  - source/Park2024-REPrune
---

# Intuition

在相同**全局稀疏度** $\bar{s}$ 下，**Ward linkage** 比 single/complete/average linkage **更能保持精度**。

作者推测原因：其他 linkage 方法更偏向**删除前层大 FLOPs、信息敏感的通道**。

---

# Rigour

## 实验对比

| Linkage 方法 | 精度保持 | 特点 |
|-------------|---------|------|
| **Ward** | ✅ **最好** | 避免簇内部过分发散 |
| Single | 较差 | 容易形成链状簇 |
| Complete | 较差 | 对异常值敏感 |
| Average | 较差 | 平衡但不够精确 |

## 假设解释

### Ward 的优势

1. **簇内紧凑性**
   - Ward 最小化合并后的 SSE（Sum of Squared Errors）
   - 保持簇内部样本的紧凑性
   - 避免"松散"的簇结构

2. **单调性**
   - Ward 距离具有**单调递增**特性
   - 便于用统一阈值控制聚类停止
   - 切分点更稳定

3. **信息保护**
   - 其他 linkage 可能过度合并**重要通道**
   - Ward 更保守，保护关键信息

### 作者推测

> 其他 linkage 方法更偏向于删除**前层大 FLOPs**、**信息敏感**的通道，导致精度下降更明显。

---

# Evidence

| 来源 | 内容 |
|------|------|
| Table 5 p7 | 四种 linkage 方法的对比实验 |
| p7 文字 | 作者的解释和推测 |

---

# 置信度说明

**置信度: medium**

原因：
- ✅ 有实验数据支持（Table 5）
- ⚠️ 仅在 REPrune 特定场景下验证
- ⚠️ 需要更多不同模型/数据集的验证
- ⚠️ 作者的解释是推测性的

---

# Links

- **支持**: [[methods/method-kernel-clustering-ward|核聚类 (Ward)]] (置信度: high)
- **相关**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **对比**: [[methods/method-linkage-methods|Linkage 方法对比]] (置信度: low)
