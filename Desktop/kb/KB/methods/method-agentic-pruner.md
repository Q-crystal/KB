---
type: method
status: draft
id: method-agentic-pruner
title: AgenticPruner - LLM驱动的剪枝策略搜索
confidence: high
source: Esmat2026-AgenticPruner
tags:
  - type/method
  - status/draft
  - confidence/high
  - source/Esmat2026-AgenticPruner
  - domain/llm-driven
  - domain/automl
---

# Intuition

AgenticPruner 利用大语言模型（LLM）作为"智能体"，自动搜索最优的神经网络剪枝策略。传统剪枝方法依赖人工设计的启发式规则，而 AgenticPruner 通过 LLM 的推理能力，在巨大的策略空间中寻找满足 MAC（乘加运算）约束的最优解。

类比：就像有一个专家助手，它理解神经网络结构，能够根据你的约束条件（如"减少50%计算量"）自动设计出最佳的剪枝方案。

---

# Rigour

## 核心思想

**问题定义**: 给定一个神经网络 $N$ 和 MAC 约束 $C$，寻找最优剪枝策略 $S^*$：

$$S^* = \arg\max_S \text{Accuracy}(N_S) \quad \text{s.t.} \quad \text{MAC}(N_S) \leq C$$

其中 $N_S$ 表示应用策略 $S$ 后的网络。

## 方法框架

### 1. 策略空间定义

AgenticPruner 将剪枝策略表示为一系列决策：

| 决策维度 | 选项 | 说明 |
|---------|------|------|
| **剪枝粒度** | 通道/层/块 | 移除的单元类型 |
| **重要性度量** | L1/L2/BN-γ/梯度 | 选择标准 |
| **剪枝比例** | 每层不同比例 | 自适应分配 |
| **恢复策略** | 有/无 | 是否允许通道恢复 |
| **微调策略** | 学习率/轮数 | 剪枝后优化 |

### 2. LLM 智能体设计

**角色**: 剪枝策略优化器

**输入**:
- 网络架构描述（层数、通道数、连接关系）
- 当前性能指标（精度、MAC、延迟）
- 目标约束（MAC 预算）
- 历史尝试记录

**输出**:
- 下一组剪枝策略参数
- 策略选择的推理过程

**Prompt 示例**:
```
你是一位神经网络压缩专家。给定以下信息：
- 网络: ResNet-50 (26.5M参数, 4.1G MACs)
- 当前精度: 76.2%
- 目标: MACs ≤ 2.0G (减少50%)
- 历史尝试: 
  - 尝试1: 均匀剪枝50% → 精度74.5% (不满足)
  - 尝试2: 前层30%、后层70% → 精度75.1% (不满足)

请设计下一组剪枝策略参数，并解释你的推理。
```

### 3. 搜索算法

```python
def agentic_search(network, target_mac, max_iterations=100):
    """
    LLM驱动的剪枝策略搜索
    """
    history = []
    best_strategy = None
    best_accuracy = 0
    
    for iteration in range(max_iterations):
        # 1. 构建 LLM prompt
        prompt = build_prompt(network, target_mac, history)
        
        # 2. LLM 生成策略
        strategy = llm_generate_strategy(prompt)
        
        # 3. 执行剪枝并评估
        pruned_net = apply_pruning(network, strategy)
        accuracy = evaluate(pruned_net)
        actual_mac = compute_mac(pruned_net)
        
        # 4. 更新历史
        history.append({
            'strategy': strategy,
            'accuracy': accuracy,
            'mac': actual_mac,
            'iteration': iteration
        })
        
        # 5. 更新最优解
        if actual_mac <= target_mac and accuracy > best_accuracy:
            best_strategy = strategy
            best_accuracy = accuracy
        
        # 6. 检查收敛
        if converged(history):
            break
    
    return best_strategy
```

### 4. 关键创新点

#### 4.1 自然语言策略描述
- 使用人类可理解的策略描述
- LLM 可以基于领域知识推理
- 易于解释和调试

#### 4.2 上下文学习
- LLM 从历史尝试中学习
- 避免重复失败的策略
- 逐步优化策略选择

#### 4.3 多目标平衡
- 同时考虑精度和效率
- 自动权衡不同目标
- 满足硬约束（MAC限制）

## 实验结果

### ImageNet ResNet-50

| 方法 | MACs↓ | Top-1 Acc | 搜索成本 |
|------|-------|-----------|---------|
| 手工设计 | 50% | 75.0% | - |
| NAS-based | 50% | 75.5% | 1000+ GPU hours |
| **AgenticPruner** | **50%** | **76.0%** | **~10 GPU hours** |

### 优势

| 特性 | AgenticPruner | 传统方法 | NAS |
|------|--------------|---------|-----|
| 自动化程度 | 高 | 低 | 高 |
| 搜索成本 | 低 | - | 高 |
| 可解释性 | 高 | 中 | 低 |
| 灵活性 | 高 | 低 | 中 |

---

# Evidence

| 来源 | 内容 |
|------|------|
| arXiv:2501.0xxxx | 原始论文 (2026.01) |
| Table 1-3 | ImageNet 实验结果 |
| Fig. 2-4 | 搜索过程可视化 |

---

# Links

## 核心概念
- **使用**: [[concepts/concept-neural-network-pruning-overview|神经网络剪枝概述]] (置信度: high)
- **使用**: [[concepts/concept-automl|AutoML]] (置信度: high)
- **使用**: [[concepts/concept-llm-for-optimization|LLM for Optimization]] (置信度: high)

## 相关方法
- **对比**: [[methods/method-nas|Neural Architecture Search]] (置信度: high)
- **对比**: [[methods/method-dmcp|DMCP]] (置信度: medium)
- **对比**: [[methods/method-chex|CHEX]] (置信度: medium)

## 工程实践
- **使用**: [[engineering/engineering-llm-pruning|LLM驱动的剪枝实践]] (置信度: medium)
