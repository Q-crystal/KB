---
type: engineering
status: draft
id: engineering-concurrent-training-pruning
title: 并发训练-剪枝 (Concurrent Training-Pruning)
confidence: high
source: Park2024-REPrune
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
---

# Intuition

将 **REPrune 嵌入到训练过程**中，实现**单阶段**训练-剪枝联合优化：

1. 使用 **BN 的 $\gamma$ 参数**全局分位数阈值得到逐层稀疏度 $s_l$
2. 每隔 $\Delta T$ 步触发一次**选择 + regrowth**
3. 无需传统的 train-prune-finetune **三阶段**

---

# Rigour

## 核心机制

### 稀疏度计算

**全局稀疏度目标**：$\bar{s}$

**BN $\gamma$ 分位数**（Eq.8）：

$$Q(\bar{s}; \Gamma) = \text{Quantile}_{\bar{s}}(\{|

\gamma_i|

\})$$

**逐层稀疏度**（Eq.9）：

$$s_l = 1 - \frac{Q(\bar{s}; \Gamma)}{|

\gamma_l

|}$$

### 触发机制

- **触发间隔**：每 $\Delta T$ 训练步
- **执行操作**：
  1. 计算当前 $s_l$
  2. 执行 REPrune 选择
  3. 必要时进行 channel regrowth

### Regrowth 机制

当 $s_l = 1$ 时（整层删除）：
- 引入 **channel regrowth**（借鉴 CHEX）
- 避免无法聚类的边界情况
- 保持网络结构完整性

## 优势

| 特性 | 传统三阶段 | 并发训练-剪枝 |
|------|-----------|--------------|
| 训练轮数 | 3 轮 | 1 轮 |
| 总时间 | 长 | 短（约减少 40%）|
| 精度保持 | 需要 finetune | 联合优化 |
| 超参数 | 多阶段调参 | 端到端 |

## 实现细节

**算法流程** (Alg.2)：

```
输入: 预训练模型, 目标稀疏度 $\bar{s}$, 触发间隔 $\Delta T$
输出: 剪枝后的模型

1. 初始化 BN 层 $\gamma$ 的稀疏性正则化
2. for t = 1 to T:
3.     正常训练一步
4.     if t % $\Delta T$ == 0:
5.         计算全局阈值 $Q(\bar{s}; \Gamma)$
6.         for each layer l:
7.             计算 $s_l = 1 - Q/|

\gamma_l

|$
8.             if $s_l < 1$:
9.                 执行 REPrune 选择
10.            else:
11.                执行 channel regrowth
12. return 剪枝后的模型
```

---

# Evidence

| 来源 | 内容 |
|------|------|
| Alg.2 p5 | 完整算法流程 |
| Eq.(8)(9) p5 | 稀疏度计算公式 |
| Table 1-3 p6 | 与传统方法对比实验 |

---

# Links

- **使用**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **前提**: [[methods/method-bn-sparsity|BN 稀疏度调度]] (置信度: high)
- **相关**: [[engineering/engineering-resnet-pruning-constraints|ResNet 剪枝约束]] (置信度: high)
