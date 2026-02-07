# 云端 Codex 使用指南

## 概述

本指南介绍如何在云端环境中使用 GPT Codex 与您的 KB 知识库协作。

## 方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| GitHub Codespaces | 完全云端，与GitHub集成 | 需要网络连接 | 主要工作环境 |
| GitHub Copilot | 强大的代码补全 | 需要订阅 | 编程辅助 |
| OpenAI API | 直接调用，灵活 | 需要API密钥 | 自动化任务 |
| 本地+云端混合 | 离线可用，云端备份 | 配置复杂 | 多设备工作 |

## 方案 1: GitHub Codespaces + Codex

### 步骤 1: 在 GitHub 创建 Codespace

1. 访问您的 KB GitHub 仓库
2. 点击绿色 **Code** 按钮
3. 选择 **Codespaces** 标签
4. 点击 **Create codespace on main**

### 步骤 2: 配置 Codespace

Codespace 启动后，您会看到一个基于 VS Code 的云端开发环境。

#### 安装必要的扩展：

1. **Markdown 相关**
   - Markdown All in One
   - Markdown Preview Enhanced
   - YAML

2. **Git 相关**
   - GitLens
   - GitHub Pull Requests

3. **AI 助手**（选择其一）
   - GitHub Copilot
   - ChatGPT 扩展
   - Codeium

### 步骤 3: 配置 Codex

#### 选项 A: 使用 GitHub Copilot

1. 在 Codespace 中安装 GitHub Copilot 扩展
2. 点击扩展图标，登录 GitHub 账号
3. 开始使用 Copilot 的代码补全功能

#### 选项 B: 使用 OpenAI API

1. 获取 OpenAI API 密钥
   - 访问 https://platform.openai.com/api-keys
   - 创建新的 API 密钥

2. 在 Codespace 中配置环境变量
   ```bash
   export OPENAI_API_KEY="your-api-key-here"
   ```

3. 安装 OpenAI Python 库
   ```bash
   pip install openai
   ```

4. 创建简单的 Codex 调用脚本
   ```python
   # codex_helper.py
   import openai
   import os

   openai.api_key = os.getenv("OPENAI_API_KEY")

   def ask_codex(prompt, context=""):
       response = openai.ChatCompletion.create(
           model="gpt-4",
           messages=[
               {"role": "system", "content": "You are a helpful assistant for knowledge base management."},
               {"role": "user", "content": f"Context: {context}\n\nQuestion: {prompt}"}
           ]
       )
       return response.choices[0].message.content

   if __name__ == "__main__":
       # 示例：读取知识库文件
       with open("concepts/concept-probability.md", "r") as f:
           content = f.read()
       
       result = ask_codex("总结这个概率概念的核心要点", content)
       print(result)
   ```

### 步骤 4: 使用 Codex 处理知识库

#### 示例 1: 生成卡片摘要

```python
import os
import glob

def summarize_card(file_path):
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    # 使用 Codex 生成摘要
    prompt = f"""请为以下知识库卡片生成一个简短的摘要（2-3句话）：

{content}

摘要："""
    
    response = ask_codex(prompt)
    return response

# 处理所有概念卡片
for card in glob.glob("concepts/*.md"):
    summary = summarize_card(card)
    print(f"{os.path.basename(card)}: {summary}\n")
```

#### 示例 2: 查找相关卡片

```python
def find_related_cards(query, card_dir="concepts"):
    """使用 Codex 查找与查询相关的卡片"""
    
    cards = []
    for card_file in glob.glob(f"{card_dir}/*.md"):
        with open(card_file, 'r', encoding='utf-8') as f:
            content = f.read()
            cards.append({
                'file': os.path.basename(card_file),
                'content': content[:1000]  # 前1000字符
            })
    
    # 构建提示
    cards_text = "\n\n".join([f"卡片 {c['file']}:\n{c['content']}" for c in cards])
    
    prompt = f"""以下是一组知识库卡片，请找出与查询"{query}"最相关的3个卡片，并说明原因。

{cards_text}

相关卡片："""
    
    response = ask_codex(prompt)
    return response

# 使用示例
result = find_related_cards("随机变量的分布")
print(result)
```

#### 示例 3: 检查卡片一致性

```python
def check_card_consistency(file_path):
    """使用 Codex 检查卡片的一致性"""
    
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    prompt = f"""请检查以下知识库卡片的一致性和完整性：

{content}

请检查：
1. Front matter 是否完整（type, status, id, title, tags）
2. 是否包含 Intuition、Rigour、Evidence、Links 部分
3. Links 中的双向链接格式是否正确
4. 内容是否清晰、准确

检查结果："""
    
    response = ask_codex(prompt)
    return response

# 检查所有卡片
for card in glob.glob("concepts/*.md"):
    result = check_card_consistency(card)
    print(f"\n=== {os.path.basename(card)} ===")
    print(result)
```

## 方案 2: 使用 OpenAI Codex API 直接调用

### 安装依赖

```bash
pip install openai python-dotenv
```

### 创建配置文件

创建 `.env` 文件：

```env
OPENAI_API_KEY=your-api-key-here
```

### 创建 Codex 工具类

```python
# codex_tools.py
import os
import openai
from dotenv import load_dotenv

load_dotenv()

class KBCodexHelper:
    def __init__(self):
        openai.api_key = os.getenv("OPENAI_API_KEY")
        self.model = "gpt-4"
    
    def generate_card(self, topic, card_type="concept"):
        """生成新的知识卡片"""
        
        prompt = f"""请为以下主题创建一个知识库卡片：

主题：{topic}
类型：{card_type}

请按照以下格式生成：

```yaml
---
type: {card_type}
status: draft
id: {card_type}-{topic.lower().replace(' ', '-')}
title: {topic}
confidence: medium
source: ai-generated
tags:
  - type/{card_type}
  - status/draft
  - confidence/medium
---

# Intuition
[一句话核心概念 + 小例子]

# Rigour
- **定义**: [定义]
- **假设**: [假设]
- **适用范围**: [适用范围]
- **代价或复杂度**: [复杂度]
- **失败模式**: [失败模式]

# Evidence
[证据来源]

# Links
- **相关**: [相关卡片链接]
```

请生成内容："""
        
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a knowledge base expert. Create clear, accurate, and well-structured knowledge cards."},
                {"role": "user", "content": prompt}
            ]
        )
        
        return response.choices[0].message.content
    
    def improve_card(self, card_content):
        """改进现有卡片"""
        
        prompt = f"""请改进以下知识库卡片，使其更清晰、更准确、更完整：

{card_content}

改进建议：
1. 完善 Intuition 部分，使其更易理解
2. 补充 Rigour 部分的细节
3. 添加更多 Evidence
4. 完善 Links 部分的关系

请输出改进后的完整卡片："""
        
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a knowledge base editor. Improve clarity, accuracy, and completeness."},
                {"role": "user", "content": prompt}
            ]
        )
        
        return response.choices[0].message.content
    
    def find_relationships(self, card_content, other_cards):
        """发现卡片之间的关系"""
        
        other_cards_text = "\n\n".join([f"卡片 {name}:\n{content[:500]}" for name, content in other_cards.items()])
        
        prompt = f"""请分析以下卡片与其他卡片之间的关系：

目标卡片：
{card_content}

其他卡片：
{other_cards_text}

请识别以下类型的关系：
- 前提 (prerequisite_of)
- 衍生 (derived_from)
- 泛化 (generalizes)
- 特化 (specializes)
- 使用 (uses)
- 相似 (similar_to)

请输出关系列表："""
        
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a knowledge graph expert. Identify meaningful relationships between concepts."},
                {"role": "user", "content": prompt}
            ]
        )
        
        return response.choices[0].message.content

# 使用示例
if __name__ == "__main__":
    helper = KBCodexHelper()
    
    # 生成新卡片
    new_card = helper.generate_card("贝叶斯定理", "concept")
    print("生成的卡片：")
    print(new_card)
```

## 方案 3: 使用 Obsidian 插件 + 云端 API

### 安装 Obsidian 插件

1. **Obsidian Git**: 自动同步到 GitHub
2. **Templater**: 使用模板创建卡片
3. **Dataview**: 查询和分析卡片

### 创建自动化脚本

在 Codespace 中创建自动化脚本：

```bash
#!/bin/bash
# auto_sync.sh

# 自动同步脚本
cd /workspaces/kb

# 拉取最新更改
git pull origin main

# 运行 Codex 检查
python codex_tools.py

# 提交更改
git add .
git commit -m "Auto sync: $(date)"
git push origin main
```

### 设置定时任务

在 Codespace 中设置定时同步：

```bash
# 安装 cron
sudo apt-get update && sudo apt-get install -y cron

# 添加定时任务
echo "*/30 * * * * /workspaces/kb/auto_sync.sh >> /workspaces/kb/sync.log 2>&1" | crontab -

# 启动 cron
sudo service cron start
```

## 最佳实践

### 1. 定期同步

```bash
# 创建同步脚本 sync.sh
#!/bin/bash
echo "Syncing KB..."
git pull origin main
git add .
git commit -m "Sync: $(date '+%Y-%m-%d %H:%M:%S')"
git push origin main
echo "Sync completed!"
```

### 2. 使用有意义的提交信息

```bash
git commit -m "[CREATE] 添加贝叶斯定理概念卡片

- 添加直觉解释和数学定义
- 包含实际应用示例
- 链接到概率基础概念

相关: #concept-probability"
```

### 3. 版本控制策略

- **main 分支**: 稳定版本
- **draft 分支**: 草稿和实验性内容
- **feature 分支**: 特定主题的开发

### 4. 备份策略

```bash
# 创建备份脚本
#!/bin/bash
DATE=$(date +%Y%m%d)
git bundle create kb-backup-$DATE.bundle --all
echo "Backup created: kb-backup-$DATE.bundle"
```

## 故障排除

### 问题 1: API 密钥失效

**解决方案**:
```bash
# 检查环境变量
echo $OPENAI_API_KEY

# 重新设置
export OPENAI_API_KEY="new-api-key"
```

### 问题 2: Git 推送失败

**解决方案**:
```bash
# 检查远程仓库
git remote -v

# 重新设置远程仓库
git remote set-url origin https://github.com/YOUR_USERNAME/kb.git

# 或者使用 SSH
git remote set-url origin git@github.com:YOUR_USERNAME/kb.git
```

### 问题 3: Codespace 启动缓慢

**解决方案**:
- 使用预构建的 Codespace
- 配置 `.devcontainer/devcontainer.json` 预装依赖

## 总结

通过云端 Codex，您可以：

✅ **随时随地访问**: 通过 GitHub Codespaces 在任何设备上工作
✅ **AI 辅助**: 使用 Codex 生成、改进和分析知识卡片
✅ **自动同步**: 自动备份和同步到 GitHub
✅ **协作**: 与他人共享和协作编辑知识库
✅ **版本历史**: 完整的修改历史和回滚能力

下一步：配置您的云端 Codex 环境！
