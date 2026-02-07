---
type: engineering
status: draft
id: engineering-reprune-implementation
title: REPrune 实现细节与工程实践
confidence: high
source: Park2024-REPrune
tags:
  - type/engineering
  - status/draft
  - confidence/high
  - source/Park2024-REPrune
  - domain/implementation
---

# Intuition

REPrune 的工程实现涉及多个关键技术点，包括**聚类算法实现**、**稀疏度调度**、**残差连接处理**等。理解这些实现细节对于复现和部署至关重要。

---

# Rigour

## 1. 核心组件实现

### 1.1 凝聚聚类实现

**输入**: 核集合 $\mathcal{K}^l_j \in \mathbb{R}^{n_{l+1} \times k_h \times k_w}$

**步骤**:
```python
def agglomerative_clustering(kernels, target_clusters):
    """
    kernels: shape (n_{l+1}, k_h, k_w)
    target_clusters: 目标簇数
    """
    # 1. 初始化：每个核为一个簇
    clusters = [{i} for i in range(n_{l+1})]
    
    # 2. 预计算距离矩阵
    dist_matrix = compute_ward_distance_matrix(kernels)
    
    # 3. 迭代合并
    while len(clusters) > target_clusters:
        # 找到距离最近的两个簇
        i, j = find_min_distance_pair(dist_matrix, clusters)
        
        # 合并簇
        new_cluster = clusters[i] | clusters[j]
        clusters = [c for k, c in enumerate(clusters) if k not in (i, j)]
        clusters.append(new_cluster)
        
        # 更新距离矩阵
        update_distance_matrix(dist_matrix, new_cluster, clusters)
    
    return clusters
```

**关键优化**:
- 使用 **scipy.cluster.hierarchy** 加速
- 预计算距离矩阵避免重复计算
- Ward distance 的单调性保证正确性

### 1.2 Ward Distance 计算

```python
def ward_distance(cluster_a, cluster_b, kernels):
    """
    计算两个簇的 Ward distance
    """
    # 计算质心
    centroid_a = np.mean(kernels[cluster_a], axis=0)
    centroid_b = np.mean(kernels[cluster_b], axis=0)
    
    # Ward distance 公式
    n_a = len(cluster_a)
    n_b = len(cluster_b)
    distance = (n_a * n_b / (n_a + n_b)) * np.linalg.norm(centroid_a - centroid_b)**2
    
    return distance
```

### 1.3 最大覆盖问题求解

```python
def greedy_mcp_filter_selection(clusters_per_channel, target_filters):
    """
    贪心算法求解最大覆盖问题
    """
    n_filters = len(clusters_per_channel[0])  # 总滤波器数
    selected = []
    covered = set()
    
    while len(selected) < target_filters:
        best_filter = -1
        best_coverage = 0
        
        # 遍历所有未选中的滤波器
        for f in range(n_filters):
            if f in selected:
                continue
            
            # 计算该滤波器的覆盖
            coverage = 0
            for channel_clusters in clusters_per_channel:
                # 滤波器 f 在该通道覆盖的簇
                covered_by_f = get_covered_clusters(f, channel_clusters)
                new_coverage = len(covered_by_f - covered)
                coverage += new_coverage
            
            if coverage > best_coverage:
                best_coverage = coverage
                best_filter = f
        
        # 处理平局：随机选择
        if best_filter == -1:
            candidates = get_tied_filters(n_filters, selected)
            best_filter = random.choice(candidates)
        
        selected.append(best_filter)
        
        # 更新已覆盖集合
        for channel_clusters in clusters_per_channel:
            covered |= get_covered_clusters(best_filter, channel_clusters)
    
    return selected
```

## 2. 稀疏度调度实现

### 2.1 BN Gamma 统计

```python
def compute_global_threshold(model, global_sparsity):
    """
    基于 BN gamma 计算全局阈值
    """
    all_gammas = []
    
    for module in model.modules():
        if isinstance(module, nn.BatchNorm2d):
            # 收集所有 gamma 的绝对值
            gammas = torch.abs(module.weight).detach().cpu().numpy()
            all_gammas.extend(gammas.flatten().tolist())
    
    # 计算分位数
    threshold = np.percentile(all_gammas, global_sparsity * 100)
    return threshold

def compute_layer_sparsity(bn_module, threshold):
    """
    计算单层稀疏度
    """
    gammas = torch.abs(bn_module.weight)
    n_channels = len(gammas)
    
    # 统计低于阈值的通道数
    pruned = (gammas <= threshold).sum().item()
    sparsity = pruned / n_channels
    
    return sparsity
```

### 2.2 硬剪枝实现

```python
def hard_prune_layer(conv_layer, bn_layer, selected_indices):
    """
    硬剪枝：直接移除未选中的通道
    """
    # 1. 剪枝 BN 层
    bn_layer.weight.data = bn_layer.weight.data[selected_indices]
    bn_layer.bias.data = bn_layer.bias.data[selected_indices]
    bn_layer.running_mean = bn_layer.running_mean[selected_indices]
    bn_layer.running_var = bn_layer.running_var[selected_indices]
    bn_layer.num_features = len(selected_indices)
    
    # 2. 剪枝卷积层（输出通道）
    conv_layer.weight.data = conv_layer.weight.data[selected_indices]
    if conv_layer.bias is not None:
        conv_layer.bias.data = conv_layer.bias.data[selected_indices]
    conv_layer.out_channels = len(selected_indices)
    
    # 3. 调整下一层的输入通道
    next_layer = get_next_layer(conv_layer)
    if next_layer is not None:
        next_layer.weight.data = next_layer.weight.data[:, selected_indices]
        next_layer.in_channels = len(selected_indices)
```

## 3. ResNet 特殊处理

### 3.1 BasicBlock 剪枝

```python
class PrunedBasicBlock(nn.Module):
    """
    剪枝后的 BasicBlock
    只剪第一个 3x3，保持第二个 3x3 和 shortcut
    """
    def __init__(self, in_channels, out_channels, stride=1):
        super().__init__()
        
        # 第一个 3x3：可剪枝
        self.conv1 = nn.Conv2d(in_channels, out_channels, 3, stride, 1)
        self.bn1 = nn.BatchNorm2d(out_channels)
        
        # 第二个 3x3：保持原样（不剪枝）
        self.conv2 = nn.Conv2d(out_channels, out_channels, 3, 1, 1)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        # Shortcut：保持原样
        self.shortcut = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 1, stride),
                nn.BatchNorm2d(out_channels)
            )
    
    def forward(self, x):
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += self.shortcut(x)  # 维度匹配
        out = F.relu(out)
        return out
```

### 3.2 Bottleneck 剪枝

```python
class PrunedBottleneck(nn.Module):
    """
    剪枝后的 Bottleneck
    剪前两个卷积（1x1 和 3x3），不剪最后一个 1x1
    """
    def __init__(self, in_channels, mid_channels, out_channels, stride=1):
        super().__init__()
        
        # 第一个 1x1：可剪枝
        self.conv1 = nn.Conv2d(in_channels, mid_channels, 1)
        self.bn1 = nn.BatchNorm2d(mid_channels)
        
        # 3x3：可剪枝
        self.conv2 = nn.Conv2d(mid_channels, mid_channels, 3, stride, 1)
        self.bn2 = nn.BatchNorm2d(mid_channels)
        
        # 最后一个 1x1：不剪枝（保持输出维度）
        self.conv3 = nn.Conv2d(mid_channels, out_channels, 1)
        self.bn3 = nn.BatchNorm2d(out_channels)
        
        # Shortcut
        self.shortcut = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 1, stride),
                nn.BatchNorm2d(out_channels)
            )
```

## 4. Channel Regrowth 实现

```python
def channel_regrowth(model, layer_idx, regrowth_ratio=0.1):
    """
    通道恢复：当某层被完全剪枝时恢复部分通道
    借鉴 CHEX 策略
    """
    layer = model.layers[layer_idx]
    
    # 检查是否被完全剪枝
    if layer.out_channels == 0:
        # 恢复部分通道
        original_channels = layer.original_out_channels
        n_regrowth = int(original_channels * regrowth_ratio)
        
        # 随机选择恢复的通道
        candidates = list(range(original_channels))
        regrowth_indices = random.sample(candidates, n_regrowth)
        
        # 重新初始化这些通道
        for idx in regrowth_indices:
            layer.weight.data[idx] = torch.randn_like(layer.weight.data[idx]) * 0.01
            if layer.bias is not None:
                layer.bias.data[idx] = 0
        
        layer.out_channels = n_regrowth
        
        return True
    
    return False
```

## 5. 训练循环集成

```python
def train_with_reprune(model, train_loader, val_loader, config):
    """
    集成 REPrune 的完整训练循环
    """
    optimizer = torch.optim.SGD(model.parameters(), lr=config.lr, momentum=0.9)
    scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, config.epochs)
    
    for epoch in range(config.epochs):
        # 正常训练
        model.train()
        for batch_idx, (data, target) in enumerate(train_loader):
            optimizer.zero_grad()
            output = model(data)
            loss = F.cross_entropy(output, target)
            loss.backward()
            optimizer.step()
        
        # REPrune 剪枝触发
        if (epoch % config.prune_interval == 0 and 
            epoch <= config.prune_epochs):
            
            # 1. 计算全局阈值
            threshold = compute_global_threshold(model, config.global_sparsity)
            
            # 2. 逐层执行 REPrune
            for layer_idx, (conv, bn) in enumerate(get_conv_bn_pairs(model)):
                sparsity = compute_layer_sparsity(bn, threshold)
                
                if sparsity >= 1.0:
                    # 完全剪枝，执行 regrowth
                    channel_regrowth(model, layer_idx)
                else:
                    # 执行 REPrune 选择
                    selected = reprune_layer_selection(conv, bn, sparsity)
                    hard_prune_layer(conv, bn, selected)
            
            # 3. 调整优化器（移除已剪枝参数）
            optimizer = adjust_optimizer(optimizer, model)
        
        scheduler.step()
        
        # 验证
        if epoch % config.val_interval == 0:
            validate(model, val_loader)
    
    return model
```

---

# Evidence

| 来源 | 内容 |
|------|------|
| Appendix A.5 | ResNet 剪枝细节 |
| Appendix A.2 | 训练设置 |
| Methodology p3-5 | 算法流程 |

---

# Links

- **实现**: [[concepts/concept-reprune|REPrune]] (置信度: high)
- **使用**: [[methods/method-reprune-algorithm1|Algorithm 1]] (置信度: high)
- **使用**: [[methods/method-reprune-algorithm2|Algorithm 2]] (置信度: high)
- **相关**: [[engineering/engineering-resnet-pruning-constraints|ResNet 剪枝约束]] (置信度: high)
- **相关**: [[engineering/engineering-concurrent-training-pruning|并发训练-剪枝]] (置信度: high)
