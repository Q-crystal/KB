# GitHub 配置指南

## 1. 创建 GitHub 仓库

### 步骤 1: 在 GitHub 上创建新仓库

1. 访问 https://github.com/new
2. 填写仓库信息：
   - **Repository name**: `kb` (或您喜欢的名称)
   - **Description**: 个人知识库 (Obsidian + Git)
   - **Visibility**: 
     - 选择 **Private** 如果您希望知识库保密
     - 选择 **Public** 如果您希望分享知识库
   - **Initialize this repository with**: 不要勾选任何选项
3. 点击 **Create repository**

### 步骤 2: 连接本地仓库到 GitHub

在终端中执行以下命令（将 `YOUR_USERNAME` 替换为您的GitHub用户名）：

```bash
# 添加远程仓库
git remote add origin https://github.com/YOUR_USERNAME/kb.git

# 推送代码到GitHub
git branch -M main
git push -u origin main
```

或者使用 SSH（如果您配置了SSH密钥）：

```bash
git remote add origin git@github.com:YOUR_USERNAME/kb.git
git branch -M main
git push -u origin main
```

### 步骤 3: 验证推送

1. 访问 `https://github.com/YOUR_USERNAME/kb`
2. 确认所有文件都已上传

## 2. 日常使用流程

### 本地修改后推送到云端

```bash
# 查看修改
git status

# 添加所有修改
git add .

# 提交修改
git commit -m "描述您的修改"

# 推送到GitHub
git push origin main
```

### 从云端拉取最新版本

```bash
# 拉取最新代码
git pull origin main
```

## 3. 云端使用 GPT Codex

### 方案 1: GitHub Codespaces (推荐)

GitHub Codespaces 提供云端开发环境，可以直接在浏览器中使用。

#### 设置步骤：

1. **在GitHub仓库中启用Codespaces**
   - 访问仓库页面
   - 点击 **Code** 按钮
   - 选择 **Codespaces** 标签
   - 点击 **Create codespace on main**

2. **在Codespaces中安装Obsidian**
   - Codespaces基于VS Code，可以使用Markdown插件
   - 或者使用Obsidian的Web版本（如果有）

3. **配置Codex**
   - 在Codespaces中安装Codex扩展
   - 配置API密钥
   - 开始使用

#### 优点：
- 完全云端，无需本地环境
- 与GitHub深度集成
- 自动同步代码

### 方案 2: GitHub Copilot

GitHub Copilot 是GitHub提供的AI编程助手。

#### 设置步骤：

1. **申请GitHub Copilot**
   - 访问 https://github.com/features/copilot
   - 申请访问权限（可能需要等待）

2. **在本地或Codespaces中使用**
   - 安装GitHub Copilot扩展
   - 登录GitHub账号
   - 开始使用

#### 优点：
- 与GitHub无缝集成
- 支持多种IDE
- 强大的代码补全功能

### 方案 3: 使用其他云端AI工具

#### OpenAI Codex API

1. **获取API密钥**
   - 访问 https://platform.openai.com/
   - 创建API密钥

2. **在云端环境中使用**
   - 使用Python或其他语言调用API
   - 或者使用支持Codex的编辑器

#### 其他云端IDE

- **Gitpod**: https://gitpod.io
- **Replit**: https://replit.com
- **CodeSandbox**: https://codesandbox.io

## 4. 多设备同步

### 在新设备上克隆仓库

```bash
# 克隆仓库
git clone https://github.com/YOUR_USERNAME/kb.git

# 进入目录
cd kb

# 在Obsidian中打开
# 文件 → 打开文件夹 → 选择 kb 文件夹
```

### 同步流程

```
设备A修改 → git push → GitHub云端 → git pull → 设备B更新
```

### 避免冲突的最佳实践

1. **经常提交和推送**
   - 完成一个小任务后立即提交
   - 每天至少推送一次

2. **先拉取再推送**
   ```bash
   git pull origin main
   # 解决冲突（如果有）
   git push origin main
   ```

3. **使用有意义的提交信息**
   ```bash
   git commit -m "[类型] 具体描述
   
   详细说明修改原因和影响"
   ```

## 5. Obsidian Git 插件（可选）

### 安装 Obsidian Git 插件

1. 在Obsidian中打开 **设置** → **社区插件**
2. 关闭 **安全模式**
3. 点击 **浏览** 搜索 "Git"
4. 安装 **Obsidian Git** 插件
5. 启用插件

### 配置自动同步

1. 打开 **Obsidian Git** 设置
2. 配置：
   - **自动提交间隔**: 30分钟
   - **自动推送**: 开启
   - **提交信息模板**: `vault backup: {{date}}`

### 手动使用

- **Ctrl+P** → 输入 "Git" 查看所有Git命令
- **Backup**: 立即提交并推送
- **Commit**: 提交所有修改
- **Push**: 推送到远程
- **Pull**: 从远程拉取

## 6. 安全建议

### 保护敏感信息

1. **不要将敏感信息提交到Git**
   - API密钥
   - 密码
   - 个人身份信息

2. **使用 .gitignore**
   - 已配置排除 `.obsidian/workspace.json` 等文件
   - 如需排除更多文件，添加到 `.gitignore`

3. **使用环境变量**
   - 在云端环境中设置环境变量
   - 在代码中引用环境变量

### 私有仓库

- 建议将知识库设为 **Private**
- 只有您和授权用户可以访问

## 7. 故障排除

### 推送失败

```bash
# 先拉取最新代码
git pull origin main

# 解决冲突后重新推送
git push origin main
```

### 身份验证问题

```bash
# 配置Git用户名和邮箱
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 使用GitHub CLI登录
gh auth login
```

### 大文件问题

如果知识库包含大文件（如图片、PDF）：

1. **使用 Git LFS**
   ```bash
   git lfs install
   git lfs track "*.pdf"
   git lfs track "*.png"
   ```

2. **或者使用外部存储**
   - 将大文件存储在网盘
   - 在知识库中链接到外部文件

## 8. 总结

通过GitHub连接，您可以：

✅ **云端备份**: 知识库安全存储在云端
✅ **多设备同步**: 在任何设备上访问和编辑
✅ **版本历史**: 查看和恢复之前的版本
✅ **云端AI**: 使用GitHub Codespaces + Codex
✅ **协作共享**: 与他人共享和协作（可选）

下一步：创建GitHub仓库并推送代码！
