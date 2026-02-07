# 论文阅读与提取指南

## 联网读取论文的方法

### 方法1: ar5iv.org HTML版本（推荐）

ar5iv.org 提供 arXiv 论文的 HTML 版本，便于阅读和提取。

**URL格式**:
```
https://ar5iv.org/html/[arXiv_ID]
```

**示例**:
```
https://ar5iv.org/html/2402.17862
```

**优点**:
- ✅ 结构化 HTML，易于解析
- ✅ 保留数学公式（LaTeX格式）
- ✅ 包含图表和表格
- ✅ 支持章节导航

### 方法2: arXiv Abstract页面

获取论文基本信息和摘要。

**URL格式**:
```
https://arxiv.org/abs/[arXiv_ID]
```

**示例**:
```
https://arxiv.org/abs/2402.17862
```

### 方法3: 直接下载PDF

**URL格式**:
```
https://arxiv.org/pdf/[arXiv_ID].pdf
```

**注意**: PDF解析相对复杂，建议优先使用 HTML 版本。

## 论文提取工作流程

### Step 1: 获取论文HTML

使用 WebFetch 工具获取论文内容：

```
WebFetch: https://ar5iv.org/html/2402.17862
```

### Step 2: 结构化提取

从 HTML 中提取以下部分：

| 部分 | 内容 |
|------|------|
| Title | 论文标题 |
| Authors | 作者列表 |
| Abstract | 摘要 |
| Introduction | 引言和动机 |
| Related Work | 相关工作 |
| Methodology | 方法详细描述 |
| Experiments | 实验设置和结果 |
| Conclusion | 结论 |
| Appendix | 附录（数学推导等） |

### Step 3: 识别知识原子

从论文中提取候选原子：

**概念 (Concept)**:
- 核心创新点
- 关键定义
- 技术术语

**方法 (Method)**:
- 算法步骤
- 数学公式
- 优化目标

**工程 (Engineering)**:
- 实现细节
- 超参数设置
- 代码结构

**主张 (Claim)**:
- 实验结果
- 性能对比
- 消融实验

### Step 4: 创建知识卡片

按照 KB 规范创建卡片：

```markdown
---
type: concept/method/engineering/claim
status: draft
id: unique-id
title: 标题
confidence: high/medium/low
source: AuthorYear-Title
tags:
  - type/xxx
  - status/draft
  - confidence/xxx
  - source/AuthorYear-Title
---

# Intuition
一句话核心 + 具体例子

# Rigour
- 定义
- 假设
- 适用范围
- 复杂度
- 失败模式

# Evidence
来源定位 + 解释

# Links
- 关系: [[path/card|标题]] (置信度: xxx)
```

## 数学公式处理

### 行内公式

使用 `$...$` 格式：
```markdown
Ward distance: $I(A_c, B_c) = \frac{|A_c||B_c|}{|A_c|+|B_c|} \|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$
```

### 独立公式

使用 `$$...$$` 格式：
```markdown
$$
h_l = \max_{j \in \mathcal{J}} d(\tilde{n}_{l+1}; \mathcal{K}^l_j)
$$
```

### 公式编号引用

保留原文的公式编号：
```markdown
**Ward distance** (Eq. 2):
$$I(A_c, B_c) = \frac{|A_c||B_c|}{|A_c|+|B_c|} \|\mathbf{m}_{A_c} - \mathbf{m}_{B_c}\|^2$$
```

## 证据引用规范

### 表格引用

```markdown
| 来源 | 内容 |
|------|------|
| Table 1 p6 | ImageNet 主要结果 |
| Table 5 p7 | Linkage 方法对比 |
```

### 图表引用

```markdown
| 来源 | 内容 |
|------|------|
| Fig. 1 p1 | REPrune 动机示意图 |
| Fig. 3 p7 | Coverage rate 变化 |
```

### 公式引用

```markdown
| 来源 | 内容 |
|------|------|
| Eq. 2 p3 | Ward distance 定义 |
| Eq. 8-9 p5 | 稀疏度计算公式 |
```

## 批量提取策略

### 优先级排序

1. **高优先级**: 核心概念、主要算法
2. **中优先级**: 实验结果、消融分析
3. **低优先级**: Related Work、实现细节

### 迭代提取

```
第一轮: 核心概念 + 主要算法
  ↓
第二轮: 实验结果 + 消融分析
  ↓
第三轮: 实现细节 + Related Work
  ↓
第四轮: 完善链接 + 补充细节
```

## 质量控制检查清单

- [ ] Front Matter 完整（type, status, id, title, tags）
- [ ] 数学公式正确渲染
- [ ] 证据来源明确（页码、表格号）
- [ ] Links 关系准确
- [ ] 格式统一（标题、表格、代码）
- [ ] 提交信息清晰

## 工具使用

### WebFetch

获取论文 HTML 内容：
```
WebFetch: https://ar5iv.org/html/[arXiv_ID]
```

### WebSearch

搜索相关论文：
```
WebSearch: "REPrune channel pruning"
```

## 示例：REPrune 提取总结

**论文**: REPrune: Channel Pruning via Kernel Representative Selection
**arXiv**: 2402.17862
**方法**: ar5iv.org HTML

**提取结果**:
- 7 概念卡片
- 6 方法卡片
- 3 工程卡片
- 4 主张卡片
- 共 20 张卡片

**关键提取**:
1. 核心概念: REPrune 机制
2. 算法: Alg.1, Alg.2 详细分析
3. 数学: Ward's Linkage 完整推导
4. 实验: Table 1-12 全面分析
5. 消融: 5项消融实验
6. 实现: Python 代码指南

---

**最后更新**: 2026-02-07
**版本**: v1.0
