# PPT Script Markdown Format

## File structure

```
# <演讲/报告标题>

> **活动**：<答辩/汇报/评审>  **时间**：<N> 分钟  **共 <N> 页**

---

## 第 1 页：<标题>

### 幻灯片文案

...

### 配图

...

### Speaker Notes

...

---

## 第 2 页：<标题>

...
```

---

## Per-slide format spec

### `### 幻灯片文案`

The **visible text content** of the slide. Use the exact formatting you want on the slide:

- **Bold** for section headers, key terms, and callout labels
- Plain text for body copy
- `> ` blockquote for important callout boxes or highlighted conclusions
- Tables for comparisons (markdown table syntax)
- Code-fenced ASCII diagrams for layouts or flow arrows

Rules:
- Keep slide copy **dense but not verbose** — 3–6 bullet points max per slide
- Write as the audience will read it, not as speaker notes
- Figures are described in `### 配图`, not here

### `### 配图`

One line per figure on this slide. Format:

```
- <filename.ext>：<one-line caption explaining what the figure shows>
```

If no figures: write `无`.

If figures aren't ready yet: write `[待补充] <描述所需图的内容>`.

### `### Speaker Notes`

The spoken narration for this slide. Rules:

- Write in **first-person spoken Chinese** (natural, not written/formal)
- One paragraph per logical point; blank line between paragraphs
- Start with a transition that connects to the previous slide where appropriate
- Cover: context → content → significance; don't just read the slide
- For formula slides: explain the intuition, not the math
- Target ~60–90 seconds of speech per slide (100–150 Chinese characters per minute)

---

## Complete example — one content slide

```markdown
## 第 3 页：相关工作与研究定位

### 幻灯片文案

**现有方法：两条路径**

路径一：Rollout 后过滤（DAPO）
- ⚠ 生成完 rollout 后过滤零信号组
- ⚠ 无效 rollout 成本已发生，需重采样补 batch

路径二：入口侧采样（本文）
- ✅ 采样前调整题目出现频率
- ✅ 无效 rollout 从源头消除

**入口侧现有方法的局限**

| 方法 | 粒度 | 价值指标 | 采样控制 | 主要局限 |
|------|------|----------|----------|----------|
| DUMP | 分布级 | 优势绝对值均值 | UCB | 单领域调度空间受限 |
| SPO  | 题目级 | 伯努利标准差 | 概率下界 | 框架内验证，维度耦合 |
| Actor-Curator | 题目级 | 策略提升估计 | bandit约束 | 额外 curator 模型，复杂度高 |

> ⚠ 多维度同时变化，难以判断**哪个机制**真正带来收益

**本文工作**：统一实现下系统消融**价值指标 × 采样控制**的交互效应

### 配图

无（两栏对比 + 表格布局）

### Speaker Notes

围绕如何让每次采样都产生有效学习信号，现有工作大致沿两条路径展开。

一条是 rollout 后过滤，DAPO 是代表：生成完一组响应后，根据组内奖励是否有区分度决定要不要保留，
缺点是无效的 rollout 成本已经发生，还需要重复采样补齐合格 batch。另一条是把干预前移到数据入口，
在采样之前就根据题目价值调整出现频率，工程开销更低，本文属于这一路。

入口侧已有 DUMP、SPO、Actor-Curator 三个代表方法，但它们各自在自己的框架内验证，价值定义、
历史估计、采样控制这些设计维度同时变化，很难单独判断哪个因素真正起作用。

所以本文的核心定位是：在统一的实现和实验设置下，将入口侧动态采样的设计空间显式拆解为四个维度，
重点系统考察价值指标和采样控制这两个维度及其交互效应。

---
```

---

## Cover slide example

```markdown
## 第 1 页：封面

### 幻灯片文案

**<论文/报告完整标题>**

<English subtitle if academic>

- 学生 / 汇报人：<姓名>
- 指导教师：<姓名 职称>（如适用）
- 单位：<院系>
- 日期：<年月>

### 配图

无（封面纯文字）

### Speaker Notes

大家好，我叫<姓名>，来自<单位>。今天我要汇报的题目是《<标题>》。
（如有导师：指导老师是<导师姓名>。）
```

---

## Summary/conclusion slide example

```markdown
## 第 N 页：总结与展望

### 幻灯片文案

**三条核心发现**

① <发现一，一句话>
② <发现二，一句话>
③ <发现三，一句话>

**局限性**
- <局限一>
- <局限二>

**未来工作**
- ▸ <方向一>
- ▸ <方向二>

### 配图

无

### Speaker Notes

最后做个总结。<整体贡献一句话>，得到了三条核心发现。

第一，<发现一详细解释>。
第二，<发现二详细解释>。
第三，也是最重要的，<发现三详细解释>。

此外，<局限性说明>。

谢谢！
```

---

## Common pitfalls

| Pitfall | Fix |
|---|---|
| Speaker notes just repeat slide text | Notes add context, transitions, and intuition the slide doesn't show |
| Too many bullets on one slide | Split into two slides or condense to 3–5 key points |
| Figure listed in 幻灯片文案 | Move figure reference to 配图 section |
| Formal written Chinese in speaker notes | Use natural spoken Chinese; test by reading aloud |
| Missing transition sentences | Start each notes section with a connecting sentence to the previous slide |
