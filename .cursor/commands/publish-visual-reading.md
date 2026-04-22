# publish-visual-reading

发布一篇 visualized paper/blog reading 到 `syt-nju.github.io` 的 `/visual-reading/` 栏目：把独立 HTML 文件放到指定目录、在元数据里追加一条卡片、提交并推送。

## 输入

用户会给：
- `html_path`（必填）：本地 HTML 文件路径（单文件、自包含，符合 `blog-to-html` 产出）
- `title`（必填）：卡片标题——列表页上**只有标题**，所以标题本身必须讲清这是在讲什么
  - **风格**：像朋友之间介绍文章时会说的那一句话，名词+动作，带一点具体性；中英混排没问题（专有名词/术语保留英文）
  - **形式**：短、不加标点结尾；不写成论文题目的严肃口吻；不要用"浅谈/深入/全面"这类套话
  - **示例（好）**：`"Anthropic 怎么构建 multi-agent research system"`、`"DeepSeek-V3 的 MLA 到底省了什么"`
  - **示例（差）**：`"我们如何构建多智能体研究系统"`（照搬原文、没有主语锚点）、`"多智能体研究系统设计浅析"`（空话）

可选：
- `slug`：URL 目录名；缺省时由 `title` 派生（小写、空格→`-`、去掉非 `[a-z0-9-]` 字符、中文人工译为对应英文）
- `date`：`YYYY-MM-DD`；缺省用今天
- `source_type`：`paper` 或 `blog`；缺省 `blog`
- `source_url`：原文链接；缺省省略

缺失的必填字段先问用户，不要猜。标题如果明显不达标（空话/照搬原题），也主动提一个更好的版本给用户选。

## 仓库约定

- 站点根：`syt-nju.github.io/`
- HTML 落地：`syt-nju.github.io/files/visual-reading/<slug>/index.html`
- 元数据：`syt-nju.github.io/_data/visual_reading.yml`（`entries` 列表，新的在前）
- 索引页：`_pages/visual-reading.md`（读 `_data/visual_reading.yml` 渲染，无需手改）

## 步骤

### 1. 校验输入
- 确认 `html_path` 存在且是 `.html`
- 若缺 `title`/`summary`，向用户询问
- 计算 `slug`：若未给，按 `title` 派生；若目标目录 `files/visual-reading/<slug>/` 已存在，停下来向用户确认是覆盖还是换个 slug

### 2. 放置 HTML 文件
```bash
mkdir -p syt-nju.github.io/files/visual-reading/<slug>
cp <html_path> syt-nju.github.io/files/visual-reading/<slug>/index.html
```

### 3. 在 `_data/visual_reading.yml` 顶部插入一条

用 `StrReplace` 工具对 `syt-nju.github.io/_data/visual_reading.yml` 做精确替换，保留文件头注释。两种情况：

**情况 A：当前是空列表**（文件里有 `entries: []`）

把：
```
entries: []
```
替换为：
```
entries:
  - slug: <slug>
    title: "<title>"
    date: <date>
    source_type: <source_type>
    source_url: "<source_url>"
```
（省略的可选字段就不写该行）

**情况 B：已有条目**（文件里是 `entries:\n  - slug: ...`）

定位到现有第一条 `  - slug: ...` 所在行，在它上方插入新条目块（同样缩进两空格、字段对齐 4 空格）。用 `StrReplace` 把：
```
entries:
  - slug: <existing_first_slug>
```
替换为：
```
entries:
  - slug: <new_slug>
    title: "<title>"
    date: <date>
    source_type: <source_type>
    source_url: "<source_url>"
  - slug: <existing_first_slug>
```

**字符串转义注意**：`title` 里如果含英文双引号，替换时把字段值改成单引号包裹；如果同时含单双引号，转成 YAML 折行字符串（`>-` 或 `|`）。**不要**让 YAML 无法解析。

### 4. 本地快速校验（不启服务也能做）
```bash
python3 -c "import yaml,sys; yaml.safe_load(open('syt-nju.github.io/_data/visual_reading.yml'))"
```
失败则回到第 3 步修正。

### 5. 提交并推送
在 `syt-nju.github.io/` 内执行：
```bash
git add files/visual-reading/<slug>/ _data/visual_reading.yml
git status
```
确认只有预期文件被改动，然后：
```bash
git commit -m "visual-reading: add <slug>"
git push
```
若 `git push` 被拒绝（远端有新提交），按 `.cursor/commands/git-push.md` 的方式 `git pull --rebase && git push`。

### 6. 交付
给用户：
- 本地访问路径：`/visual-reading/` 和 `/files/visual-reading/<slug>/`
- commit hash
- 远端上线后的完整 URL：`https://syt-nju.github.io/visual-reading/`、`https://syt-nju.github.io/files/visual-reading/<slug>/`

## 验证标准

- `syt-nju.github.io/files/visual-reading/<slug>/index.html` 存在且非空
- `_data/visual_reading.yml` 是合法 YAML，且新条目在 `entries` 列表第一位
- `git log -1` 显示新增 commit，`git status` 干净
- 远端 push 成功

## 反面教训

1. **手改 `_pages/visual-reading.md`**：没必要，索引页会自动从 `_data` 渲染。只有改页面文案/样式时才动它。
2. **把 HTML 放到 `syt-nju.github.io/_pages/` 或 `_posts/`**：会被 Jekyll 当页面套 layout，破坏独立整页交互。必须放 `files/` 下。
3. **slug 用中文或空格**：URL 会被 URL-encode，影响可读性；固定用 kebab-case 英文。
4. **不校验 YAML 就 push**：解析失败会让 `/visual-reading/` 整页构建失败。第 4 步必跑。
5. **标题照搬原文标题**：列表页只靠标题撑信息量，原文标题通常太抽象或太谦逊（"我们如何构建 X"），转述成带主语锚点和具体对象的版本（"Anthropic 怎么构建 multi-agent research system"）。
6. **加回 summary 字段**：之前试过用 summary 做副标题，视觉上抢戏、写起来也容易空话。方案已废弃；信息全部压进 title。
