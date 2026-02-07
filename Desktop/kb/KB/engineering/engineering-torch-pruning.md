---
type: engineering
status: draft
id: engineering-torch-pruning
title: Torch-Pruning - 通用结构化剪枝框架
confidence: high
source: Fang2023-DepGraph
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Fang2023-DepGraph
  - domain/framework
  - domain/structured-pruning
---

# Intuition

Torch-Pruning (TP) 是一个通用的PyTorch结构化剪枝框架，基于DepGraph（依赖图）算法实现。与其他仅通过mask模拟剪枝的工具不同，TP能够**物理地移除参数和通道**，真正降低模型的推理成本。

核心优势：**自动处理层间依赖关系**，支持复杂的现代网络结构（如ResNet、Transformer、YOLO等）。

---

# Rigour

## 核心特性

| 特性 | 说明 | 优势 |
|------|------|------|
| **通用性** | 支持任意网络结构 | 无需针对特定网络修改 |
| **结构化** | 物理移除参数 | 真实加速，非模拟 |
| **自动化** | 自动处理依赖 | 简化剪枝流程 |
| **灵活性** | 支持多种剪枝策略 | 可定制重要性评估 |

## DepGraph 算法

### 核心思想

神经网络中的层之间存在复杂的依赖关系。DepGraph通过构建**依赖图**来建模这些关系，确保剪枝时自动处理所有相关层。

### 依赖类型

```
输入: Conv1 (3→64) ──┐
                     ├──> Conv2 (64→128)
输入: Conv1 (3→64) ──┘

剪枝 Conv1 的输出通道 → 必须同时剪枝 Conv2 的输入通道
```

### 依赖图构建

```python
# 伪代码：构建依赖图
def build_dependency_graph(model):
    graph = DependencyGraph()
    
    for module in model.modules():
        if isinstance(module, nn.Conv2d):
            # 查找输入依赖
            input_deps = find_input_dependencies(module)
            # 查找输出依赖
            output_deps = find_output_dependencies(module)
            
            graph.add_node(module, {
                'inputs': input_deps,
                'outputs': output_deps
            })
    
    return graph
```

## 安装与使用

### 安装

```bash
pip install torch-pruning
```

### 基础用法

```python
import torch
import torch_pruning as tp

# 1. 准备模型
model = torchvision.models.resnet50(pretrained=True)

# 2. 定义重要性评估
example_inputs = torch.randn(1, 3, 224, 224)
imp = tp.importance.MagnitudeImportance(p=2)  # L2范数

# 3. 初始化剪枝器
pruner = tp.pruner.MagnitudePruner(
    model,
    example_inputs,
    importance=imp,
    iterative_steps=1,
    pruning_ratio=0.5,  # 剪枝50%
)

# 4. 执行剪枝
pruner.step()

# 5. 验证
print(f"MACs: {tp.utils.count_ops_and_params(model, example_inputs)}")
```

## 高级功能

### 1. 分层剪枝比例

```python
# 为不同层设置不同剪枝比例
pruning_ratio = {
    model.conv1: 0.3,    # 第一层剪30%
    model.layer1: 0.5,   # layer1剪50%
    model.layer2: 0.6,   # layer2剪60%
    # ...
}

pruner = tp.pruner.MagnitudePruner(
    model,
    example_inputs,
    importance=imp,
    pruning_ratio=pruning_ratio,
)
```

### 2. 自定义重要性评估

```python
class CustomImportance(tp.importance.Importance):
    def __call__(self, group):
        # 自定义重要性计算
        importance = 0
        for dep, idxs in group:
            layer = dep.target.module
            if isinstance(layer, nn.Conv2d):
                # 例如：基于梯度的重要性
                importance += torch.abs(layer.weight.grad).sum()
        return importance

imp = CustomImportance()
```

### 3. 迭代剪枝

```python
pruner = tp.pruner.MagnitudePruner(
    model,
    example_inputs,
    importance=imp,
    iterative_steps=5,      # 分5步剪枝
    pruning_ratio=0.5,      # 总共剪50%
    # 每步剪枝: 1 - (1-0.5)^(1/5) ≈ 12.9%
)

for i in range(5):
    pruner.step()
    # 每步后微调
    finetune(model, epochs=1)
```

### 4. 支持的模型类型

| 模型类型 | 支持程度 | 示例 |
|---------|---------|------|
| CNN | ✅ 完全支持 | ResNet, VGG, MobileNet |
| Transformer | ✅ 完全支持 | ViT, DeiT, Swin |
| 检测模型 | ✅ 完全支持 | YOLOv5/v8, Faster R-CNN |
| 分割模型 | ✅ 完全支持 | DeepLab, UNet |
| 大语言模型 | ✅ 支持 | LLaMA, GPT-2 |
| 扩散模型 | ✅ 支持 | Stable Diffusion |

## 实际案例

### ResNet-50 剪枝

```python
import torch
import torchvision
import torch_pruning as tp

model = torchvision.models.resnet50(pretrained=True)
example_inputs = torch.randn(1, 3, 224, 224)

# 计算原始模型MACs
base_ops, base_params = tp.utils.count_ops_and_params(model, example_inputs)
print(f"Baseline: {base_ops/1e9:.2f}G MACs, {base_params/1e6:.2f}M params")

# 剪枝50%
imp = tp.importance.MagnitudeImportance(p=2)
pruner = tp.pruner.MagnitudePruner(
    model, example_inputs, importance=imp, pruning_ratio=0.5
)
pruner.step()

# 计算剪枝后MACs
pruned_ops, pruned_params = tp.utils.count_ops_and_params(model, example_inputs)
print(f"Pruned: {pruned_ops/1e9:.2f}G MACs, {pruned_params/1e6:.2f}M params")
print(f"Reduction: {base_ops/pruned_ops:.2f}x")
```

**输出**:
```
Baseline: 4.11G MACs, 25.56M params
Pruned: 2.05G MACs, 12.78M params
Reduction: 2.00x
```

## 与其他工具对比

| 特性 | Torch-Pruning | torch.nn.utils.prune | NVIDIA Apex |
|------|--------------|---------------------|-------------|
| 结构化剪枝 | ✅ 支持 | ⚠️ 仅非结构化 | ✅ 支持 |
| 物理移除参数 | ✅ 是 | ❌ 否（仅mask） | ✅ 是 |
| 自动处理依赖 | ✅ 是 | ❌ 否 | ⚠️ 部分 |
| 支持复杂网络 | ✅ 是 | ⚠️ 有限 | ⚠️ 有限 |
| 易用性 | ✅ 高 | ✅ 高 | ⚠️ 中 |

## 最佳实践

### 1. 选择合适的剪枝比例
- 从低到高逐步尝试
- 通常10-50%是安全范围
- 超过70%可能导致精度显著下降

### 2. 重要性评估选择
- **Magnitude (L1/L2)**: 简单有效，适合大多数情况
- **BN Scaling Factor**: 适合有BN层的网络
- **Gradient-based**: 训练时剪枝效果更好

### 3. 微调策略
- 剪枝后立即微调1-5个epoch
- 使用较低学习率（如原学习率的1/10）
- 监控精度恢复情况

### 4. 验证剪枝效果
```python
# 检查模型结构
print(model)

# 验证推理速度
with torch.no_grad():
    start = time.time()
    for _ in range(100):
        model(example_inputs)
    print(f"Inference time: {(time.time()-start)/100*1000:.2f}ms")
```

---

# Evidence

| 来源 | 内容 |
|------|------|
| GitHub: VainF/Torch-Pruning | 官方仓库 |
| Paper: DepGraph (CVPR 2023) | 理论基础 |
| Examples | 官方示例代码 |

---

# Links

## 核心概念
- **使用**: [[concepts/concept-structured-pruning|结构化剪枝]] (置信度: high)
- **使用**: [[concepts/concept-dependency-graph|依赖图]] (置信度: high)

## 方法
- **基于**: [[methods/method-depgraph|DepGraph算法]] (置信度: high)
- **相关**: [[methods/method-magnitude-pruning|幅度剪枝]] (置信度: high)

## 实践
- **文档**: [[engineering/engineering-torch-pruning-examples|使用示例]] (置信度: high)
- **对比**: [[engineering/engineering-pruning-frameworks|剪枝框架对比]] (置信度: medium)
