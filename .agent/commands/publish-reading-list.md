# publish-reading-list

把一条值得收藏的原文链接发布到 `syt-nju.github.io` 的 `/reading-list/` 栏目：在数据文件里追加一条（标题 + 链接 + 日期）、校验 YAML、提交并推送。**没有** HTML 文件要落地，整个动作就是改一个 YAML。

## 输入

用户会给：
- `url`（必填）：原文链接；来源域名和 favicon 都由它自动派生，无需额外字段

可选：
- `title`：卡片标题。**缺省时直接抓取并使用原文自己的标题**（保留原语言、原大小写，不要转述/改写/加工）
- `date`：`YYYY-MM-DD`；缺省用今天。只用于排序，不在卡片上展示

缺失 `url` 时先问用户，不要猜。`title` 缺省时去原文抓取：优先用 `WebFetch` 取页面 `<title>` / `og:title`，抓不到再 `WebSearch` 该 URL 确认官方标题；抓到的标题原样使用，只做必要清理（去掉站点后缀如 `· Cursor`、` - Medium` 这类）。仅当用户明确给了 `title` 时才用用户给的。

## 仓库约定

- 站点根：`syt-nju.github.io/`
- 数据文件：`syt-nju.github.io/_data/reading_list.yml`（`entries` 列表，新的在前）
- 索引页：`_pages/reading-list.md`（读 `_data/reading_list.yml` 渲染，并按 `date` 倒序排序，无需手改）
- 卡片样式 CSS 追加在 `assets/css/main.scss` 末尾（发布时无需改动）
- 每条 entry 只有三个字段：`title` / `url` / `date`。**没有** summary，**没有** 个人评论——刻意保持极简

## 步骤

### 1. 校验输入
- 确认拿到 `url`；没有就向用户询问，不要猜
- `title` 未给：去原文抓取标题（`WebFetch` 取 `<title>`/`og:title`，失败则 `WebSearch`），原样使用，仅去掉站点后缀（`· Cursor`、` - Medium` 等）。实在抓不到再问用户
- `date` 未给则用今天（`YYYY-MM-DD`）

### 2. 在 `_data/reading_list.yml` 顶部插入一条

用 `StrReplace` 工具对 `syt-nju.github.io/_data/reading_list.yml` 做精确替换，保留文件头注释。新条目放在 `entries` 列表**第一位**（页面本身也按 `date` 倒序，但保持「新的在前」的书写约定）。两种情况：

**情况 A：当前是空列表**（文件里有 `entries: []`）

把：
```
entries: []
```
替换为：
```
entries:
  - title: "<title>"
    url: "<url>"
    date: <date>
```

**情况 B：已有条目**（文件里是 `entries:\n  - title: ...`）

定位到现有第一条 `  - title: ...` 所在行，在它上方插入新条目块（同样缩进两空格、字段对齐 4 空格）。用 `StrReplace` 把：
```
entries:
  - title: "<existing_first_title>"
```
替换为：
```
entries:
  - title: "<new_title>"
    url: "<url>"
    date: <date>
  - title: "<existing_first_title>"
```

**字符串转义注意**：`title` 里如果含英文双引号，替换时把字段值改成单引号包裹；如果同时含单双引号，转成 YAML 折行/字面块字符串（`>-` 或 `|`）。`url` 同理用双引号包裹。**不要**让 YAML 无法解析。

### 3. 本地快速校验（不启服务也能做）
```bash
python3 -c "import yaml; yaml.safe_load(open('syt-nju.github.io/_data/reading_list.yml'))"
```
失败则回到第 2 步修正。

### 4. 提交并推送

**原则**：一次 publish 只发一条，commit 里只能含本条相关改动（`_data/reading_list.yml`）。

先在 `syt-nju.github.io/` 内跑 `git status`，根据情况分支：

**情况 A：工作区干净，只有本次对 `reading_list.yml` 的修改** — 直接走下面的 add/commit/push。

**情况 B：检测到其他无关改动**（别的目录的修改、其他人/其他任务留下的 staged 或 unstaged 变更） — **不要**把它们一起 add 进来，也不要替用户判断要不要带上。用 stash 隔离干净再发：
```bash
git stash push -u -m "pre-publish-reading-list"   # -u 一并暂存 untracked
```
更稳妥的做法是**先完成 stash 再做第 2 步**，把发布动作和无关改动彻底分开。stash 后再次 `git status` 确认工作区干净，然后把准备好的 YAML 改动重新落地。

接着：
```bash
git add _data/reading_list.yml
git status
```
确认只有 `_data/reading_list.yml` 被改动，然后：
```bash
git commit -m "reading-list: add <short-slug-or-title>"
git push
```
若 `git push` 被拒绝（远端有新提交），按 `.cursor/commands/git-push.md` 的方式 `git pull --rebase && git push`。

push 成功后，恢复之前暂存的无关改动：
```bash
git stash pop
```
若 pop 出现冲突，停下来告知用户冲突文件，**不要**自行 resolve 别人的工作。

### 5. 交付
给用户：
- 本地访问路径：`/reading-list/`
- commit hash
- 远端上线后的完整 URL：`https://syt-nju.github.io/reading-list/`

## 验证标准

- `_data/reading_list.yml` 是合法 YAML，且新条目在 `entries` 列表第一位
- 新条目只含 `title` / `url` / `date` 三个字段
- `git log -1` 显示新增 commit，`git status` 干净
- 远端 push 成功

## 反面教训

1. **手改 `_pages/reading-list.md`**：没必要，索引页会自动从 `_data/reading_list.yml` 渲染，并按 `date` 倒序排序。只有改页面文案/样式时才动它。
2. **加 summary 或个人评论字段**：方案刻意极简，卡片只展示标题 + 自动推导的来源域名/favicon。不要新增字段，信息全部压进 title。
3. **不校验 YAML 就 push**：解析失败会让 `/reading-list/` 整页构建失败。第 3 步必跑。
4. **擅自转述/改写标题**：reading-list 的卡片只是跳转入口，标题就用原文自己的标题（保留原语言、原大小写），不要转述、不要加主语锚点、不要"翻译润色"。只在去掉站点后缀（`· Cursor` 等）时做最小清理。只有用户明确给了 `title` 才覆盖原文标题。
5. **手动补 source/favicon 字段**：来源域名和 favicon 由 `url` 经 Liquid 自动派生，写死反而会出错。只给 `url` 就够了。
6. **把无关改动一起 commit**：publish 是单条粒度的操作，commit message 也是 `reading-list: add <short-slug-or-title>`。检测到其他改动必须 `git stash push -u` 隔离，发布完再 `git stash pop`，不要图省事一起带上。
