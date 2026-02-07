# KB 知识库

这是基于 Obsidian 的知识管理系统。

## 目录结构

- **00-Index/**: 索引和管理指南
  - [[KB_MANAGEMENT_GUIDE|KB 管理指南]]

- **concepts/**: 概念卡片
  - [[concept-probability|概率]]
  - [[concept-random-variable|随机变量]]
  - [[concept-probability-distribution|概率分布]]
  - [[concept-normal-distribution|正态分布]]
  - [[concept-reprune|REPrune - 用通道剪枝模拟核剪枝]] ⭐ 新增

- **methods/**: 方法卡片
  - [[method-probability-calculation|概率计算方法]]
  - [[method-kernel-clustering-ward|核聚类(Ward)]] ⭐ 新增
  - [[method-layer-wise-cutoff|层-wise截断]] ⭐ 新增
  - [[method-maximum-cluster-coverage|最大簇覆盖问题]] ⭐ 新增
  - [[method-reprune-algorithm1|REPrune Algorithm 1]] ⭐⭐ 详细算法
  - [[method-reprune-algorithm2|REPrune Algorithm 2]] ⭐⭐ 完整流程

- **concepts/** (REPrune 详细)
  - [[concept-ward-linkage-derivation|Ward's Linkage 完整推导]] ⭐⭐ 数学推导
  - [[concept-channel-pruning-survey|通道剪枝方法综述]] ⭐⭐ 领域综述

- **engineering/**: 工程卡片 ⭐ 新增
  - [[engineering-concurrent-training-pruning|并发训练-剪枝]]
  - [[engineering-resnet-pruning-constraints|ResNet剪枝约束]]
  - [[engineering-reprune-implementation|REPrune 实现细节]] ⭐⭐ 代码实现

- **claims/**: 主张卡片 ⭐ 新增
  - [[claim-reprune-imagenet-results|REPrune ImageNet实验结果]]
  - [[claim-ward-linkage-superiority|Ward linkage优势]]
  - [[claim-reprune-experiments-detailed|REPrune 详细实验结果]] ⭐⭐ 全面分析
  - [[claim-reprune-ablation-studies|REPrune 消融实验]] ⭐⭐ 组件验证

- **deprecated/**: 废弃卡片
  - [[old-concept-probability|旧概率概念]]

## 标签系统

### 类型标签
- `#type/concept` - 概念卡片
- `#type/method` - 方法卡片

### 状态标签
- `#status/stable` - 稳定状态
- `#status/draft` - 草稿状态
- `#status/deprecated` - 废弃状态

### 置信度标签
- `#confidence/high` - 高置信度
- `#confidence/medium` - 中置信度
- `#confidence/low` - 低置信度

## 关系边类型

- **前提**: 概念 A 是理解概念 B 的前提
- **衍生**: 概念 A 是从概念 B 衍生而来
- **泛化**: 概念 A 是概念 B 的超集
- **特化**: 概念 A 是概念 B 的子集
- **使用**: 方法 A 使用了概念 B
- **替代**: 概念 A 替代了概念 B（废弃卡片）

## 快速链接

- 查看所有稳定卡片: `#status/stable`
- 查看所有概念卡片: `#type/concept`
- 查看所有废弃卡片: `#status/deprecated`

## 云端协作

- [[00-Index/GITHUB_SETUP|GitHub 配置指南]] - 配置GitHub仓库和云端同步
- [[00-Index/CLOUD_CODEX_GUIDE|云端 Codex 使用指南]] - 在云端使用GPT Codex
- [[00-Index/AGENT_PROMPTS|智能体提示词]] - Trae智能体与Codex的协作规范

## GitHub 状态

[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/Q-crystal/KB)

**仓库地址**: `https://github.com/Q-crystal/KB.git`

**同步状态**: ✅ 已推送到GitHub，与云端同步

**日常使用**:
```bash
# 推送到云端
git push origin main

# 从云端拉取
git pull origin main
```
