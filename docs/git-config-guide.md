# Git 配置指南

本指南介绍如何为 xiaozhi-server 项目配置 Git。

## 目录

- [配置文件说明](#配置文件说明)
- [首次配置 Git](#首次配置-git)
- [推荐的 Git 设置](#推荐的-git-设置)
- [常见问题](#常见问题)

---

## 配置文件说明

项目包含以下 Git 配置文件：

| 文件 | 说明 |
|------|------|
| `.gitignore` | 指定需要忽略的文件和目录 |
| `.gitattributes` | 指定文件属性和换行符处理 |
| `models/.gitkeep` | 保留空的 models 目录 |
| `music/.gitkeep` | 保留空的 music 目录 |
| `data/.gitkeep` | 保留空的 data 目录 |
| `tmp/.gitkeep` | 保留空的 tmp 目录 |

---

## 首次配置 Git

### 1. 设置用户信息

```bash
# 设置用户名
git config --global user.name "你的名字"

# 设置邮箱
git config --global user.email "你的邮箱@example.com"
```

### 2. 设置默认分支名称

```bash
git config --global init.defaultBranch main
```

### 3. 设置换行符处理

**Windows 用户：**
```bash
# 检出时转换为 CRLF，提交时转换为 LF
git config --global core.autocrlf true
```

**Linux/macOS 用户：**
```bash
# 检出时不转换，提交时转换为 LF
git config --global core.autocrlf input
```

### 4. 设置默认编辑器（可选）

```bash
# 使用 VS Code
git config --global core.editor "code --wait"

# 或使用 Vim
git config --global core.editor "vim"
```

---

## 推荐的 Git 设置

### 1. 启用颜色输出

```bash
git config --global color.ui true
```

### 2. 设置拉取策略

```bash
# 优先使用 rebase 而不是 merge
git config --global pull.rebase true
```

### 3. 设置别名（可选）

```bash
# 常用命令缩写
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all"
```

---

## .gitignore 详解

### 已忽略的文件类型

| 类别 | 说明 |
|------|------|
| **Python 缓存** | `__pycache__/`, `*.pyc` |
| **虚拟环境** | `.venv/`, `venv/`, `env/` |
| **IDE 配置** | `.idea/`, `.vscode/` |
| **用户配置** | `data/.config.yaml`, `.env` |
| **临时文件** | `tmp/`, `*.log` |
| **模型文件** | `models/`, `*.pt`, `*.onnx` |
| **音频文件** | `music/`, `*.mp3`, `*.wav` |

### 重要：不要提交的文件

以下文件**绝对不要**提交到 Git 仓库：

```yaml
# 包含 API 密钥的配置文件
data/.config.yaml
.env
.secrets.yaml

# 大文件模型
models/*.pt
models/*.onnx

# 个人音频文件
music/*.mp3
*.wav
```

### 如何配置而不提交

1. 复制模板配置：
```bash
# 系统会自动使用 data/.config.yaml 覆盖 config.yaml
```

2. 编辑 `data/.config.yaml`（已被 .gitignore 忽略）：
```yaml
selected_module:
  LLM: ChatGLMLLM

LLM:
  ChatGLMLLM:
    api_key: "你的密钥在这里"
```

---

## .gitattributes 详解

### 换行符处理

| 文件类型 | 换行符 |
|---------|-------|
| `.sh` 脚本 | LF（Unix 风格）|
| `.bat/.ps1` | CRLF（Windows 风格）|
| 其他文本 | 自动（根据系统）|

### 二进制文件

以下文件被标记为二进制，不进行换行符转换：
- 图片：`.png`, `.jpg`, `.gif`
- 音频：`.mp3`, `.wav`, `.opus`
- 模型：`.pt`, `.onnx`, `.bin`
- 压缩包：`.zip`, `.tar.gz`

---

## 大文件管理（Git LFS）

如果需要版本控制大文件（如模型文件），可以使用 Git LFS。

### 安装 Git LFS

**Windows:**
```bash
# 使用 Chocolatey
choco install git-lfs

# 或从 https://git-lfs.com 下载
```

**Linux:**
```bash
sudo apt install git-lfs
```

**macOS:**
```bash
brew install git-lfs
```

### 启用 Git LFS

```bash
# 安装 LFS
git lfs install

# 跟踪大文件
git lfs track "*.pt"
git lfs track "*.onnx"
git lfs track "*.pth"

# 提交 .gitattributes
git add .gitattributes
git commit -m "Add Git LFS tracking"
```

---

## 常见问题

### Q: 我不小心提交了密钥文件怎么办？

**A:** 从历史记录中删除：

```bash
# 从 Git 历史中删除文件，但保留本地文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch data/.config.yaml" \
  --prune-empty --tag-name-filter cat -- --all

# 强制推送到远程
git push origin --force --all
```

**重要**：提交后立即更换所有泄露的密钥！

### Q: 如何检查哪些文件被忽略了？

```bash
# 检查某个文件是否被忽略
git check-ignore -v data/.config.yaml

# 列出所有被忽略的文件
git status --ignored
```

### Q: 模型文件太大怎么办？

**A:** 有几种方案：

1. **使用 .gitignore 忽略**（推荐）- 模型文件不提交
2. **使用 Git LFS** - 适合需要版本控制的场景
3. **提供下载脚本** - 在 README 中说明如何获取模型

### Q: 换行符总是显示修改？

**A:** 检查并设置 core.autocrlf：

```bash
# 查看当前设置
git config core.autocrlf

# Windows 设置
git config --global core.autocrlf true

# Linux/macOS 设置
git config --global core.autocrlf input

# 重新归一化所有文件
git add --renormalize .
git commit -m "Normalize line endings"
```

---

## 延伸阅读

- [Git 官方文档](https://git-scm.com/doc)
- [.gitignore 规范](https://git-scm.com/docs/gitignore)
- [Git LFS 文档](https://git-lfs.com/)
