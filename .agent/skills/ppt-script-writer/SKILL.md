---
name: ppt-script-writer
description: >
  Generate a slide-by-slide Markdown PPT script (演讲文稿) from an existing document
  (PDF thesis, LaTeX source, research paper, project report, or meeting notes).
  The script maps one-to-one to PPT pages and captures: slide copy, figure references,
  and speaker notes.

  Use when the user asks to:
  - 写 PPT 文稿 / 起 PPT 稿 / 做 PPT 设计文稿
  - Create a presentation script for thesis defense, research progress report, or project review
  - Convert an academic paper, thesis, or report into a structured PPT script
  - Draft speaker notes or narration for a presentation

  Do NOT use for actually implementing the .pptx file — use the pptx skill for that.
---

# PPT Script Writer

## Workflow

### Step 1 — Clarify requirements

Ask these 4 questions **before reading any source document**. Use `AskQuestion` tool for a polished experience.

| Question | Key options |
|---|---|
| **Source document** | Provide file path or paste content |
| **Slide structure** | Propose based on source vs. user specifies |
| **Length** | ≤10 pages (short) · 10–20 (standard) · 20+ (detailed) |
| **Figures** | All ready · Some ready · None yet · Need to search |

### Step 2 — Read source document

- **PDF/LaTeX thesis**: Read PDF for content, scan `figures/` directory for available plots
- **Research paper**: Read abstract, introduction, methodology, results, conclusion
- **Project report / meeting notes**: Read all sections, note any tables or charts

Extract: core argument, key findings, supporting evidence, conclusion.

### Step 3 — Propose slide structure

Present a numbered slide plan (one line per slide). **Wait for user approval before writing the full script.**

Example for a 9-slide thesis defense:
```
1. 封面 — 题目 / 作者 / 导师 / 院系
2. 研究背景与动机 — 问题定义 + 核心洞察
3. 相关工作 — 现有方法定位
4. 方法框架 — 设计维度 / 架构图
5. 方法细节 — 核心组件说明
6. 实验设置 — 数据集 / 模型 / 评测指标
7. 实验一 — 主要消融结果
8. 实验二 — 进一步分析
9. 总结与展望 — 贡献 / 局限 / 未来工作
```

### Step 4 — Write the full Markdown script

See [references/script-format.md](references/script-format.md) for the exact format spec and a complete slide example.

**Key rules:**
- One `## 第 N 页：<标题>` section per slide, matching the approved structure
- Each slide has three subsections: `### 幻灯片文案`, `### 配图`, `### Speaker Notes`
- Speaker notes are in first-person, natural spoken Chinese; one complete thought per paragraph
- Figure references use filename + caption description (not file path)
- Formulas: write inline as plain text approximations (`A_i = (r_i - mean) / std`) unless user requests LaTeX

---

## Common Structure Patterns

### Academic thesis defense (毕业论文答辩)
Focus: Problem → Gap → Method → Experiments → Conclusion  
Typical: 9–12 slides, 15–20 min

### Research progress report (科研进展汇报)
Focus: What was done since last meeting, key findings, blockers, next steps  
Typical: 6–8 slides, 10 min

### Project technical review (项目技术评审)
Focus: Requirements → Architecture → Implementation → Test results → Risks  
Typical: 10–15 slides, 20–30 min

### Paper reading / literature review (论文阅读分享)
Focus: Problem → Prior work → Method → Key experiments → Takeaway  
Typical: 8–10 slides, 20 min
