---
description: 初始化一个新仓库 —— git、/doc 文档目录、.gitignore、基于 uv 的环境，并完成初始 commit
---

# Init Repo

把当前目录初始化为一个干净、最小化、基于 `uv` 的项目仓库。保持通用，不强加任何脚手架（lint / 测试 / CI 等按需后续再加）。

执行前先确认当前工作目录就是要初始化的项目根目录，且尚未初始化（无 `.git`、无 `pyproject.toml`）。若已存在，停下来向用户确认是覆盖还是跳过对应步骤。

## 步骤

### 1. 初始化 git

```bash
git init
```

### 2. 创建文档目录

`/doc` 用于存放设计相关文档（架构、关键决策、方案讨论等）。

```bash
mkdir -p doc
```

为避免空目录不被 git 跟踪，放一个占位文件：

```bash
touch doc/.gitkeep
```

### 3. 创建 .gitignore

写入以下内容（软连接 `.cursor` / `.claude` 必须忽略，避免把共享配置提交进项目仓库）：

```gitignore
# Symlinks / shared agent config
.cursor
.claude

# Python
__pycache__/
*.py[cod]
*.egg-info/
.venv/
venv/
env/

# uv
.uv/

# Logs
*.log
logs/

# OS
.DS_Store
```

### 4. 基于 uv 构建环境

用 `uv` 生成最小 `pyproject.toml`（`--bare` 不会生成示例代码 / README / 额外 .gitignore，便于保持最小化）：

```bash
uv init --bare
```

创建虚拟环境并同步（此时无依赖，仅建立 `.venv` 与锁定状态）：

```bash
uv sync
```

后续加依赖统一用 `uv add <pkg>`，不要手动 `pip install`。

### 5. 初始 commit

确认 `git status` 中 `.cursor` / `.claude` 已被忽略（不应出现在待提交列表里），然后：

```bash
git add -A
git commit -m "chore: init repo (uv env, doc dir, gitignore)"
```

## 完成后

向用户简要汇报：已初始化的内容、`.gitignore` 是否成功忽略软连接、以及 uv 环境是否就绪。
