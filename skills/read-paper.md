---
name: read-paper
description: Deep academic paper analysis with bilingual (English/Chinese) output. Use this skill whenever a user wants to read, understand, analyze, or summarize a research paper or academic article — whether they provide a PDF path, paste text, or share an arXiv link. Triggers on phrases like "帮我读论文", "分析这篇paper", "总结这篇文章", "read this paper", "analyze this paper", "summarize this research", "explain this paper to me", or any time a .pdf file is mentioned in an academic context. Produces a structured report covering background, related work, contributions, methods, and experiments — all in bilingual Chinese/English format, saved to ~/Papers/.
---

# 论文深度解析 (Academic Paper Deep Analysis)

Your task is to produce a thorough, structured analysis of a research paper in bilingual format (English + Chinese), then save it to the user's `~/Papers/` directory.

## Step 1: Read the Paper

**If the user provides a PDF file path**, use the `anthropic-skills:pdf` skill to extract the full text before proceeding. Read the entire paper carefully before writing any analysis.

**If the user pastes text**, read all of it carefully.

**If the user provides an arXiv URL or paper title**, use WebFetch or WebSearch to retrieve the full content.

Take your time with this step — a thorough read pays dividends in every subsequent section. Pay attention to:
- The abstract and introduction (for context and contributions)
- Related work section if present
- The method/model architecture section
- Experiments: baselines, datasets, metrics, ablations
- Conclusion and limitations

## Step 2: Produce the Analysis Report

Output the following five sections in order. Use clean markdown formatting. Every section must be bilingual — English quotes/terms paired with Chinese translation/explanation.

---

### 一、背景 (Context)

Select **3–5 sentences** from the abstract that best capture the paper's core motivation, problem setting, and high-level solution. These should be the sentences a reader needs to understand *why this paper exists*.

Format each as:

> **[English original sentence from abstract]**
>
> 中文翻译：[精准的学术中文翻译]

After the quotes, write 2–3 sentences of your own synthesis (in Chinese) explaining the broader context: What problem space is this? Why does it matter now?

---

### 二、相关工作 (Related Work)

If the paper has a Related Work or Background section, summarize it here. If there is no explicit related work section, briefly note that and extract any relevant prior work references from the introduction.

Organize by theme or category if there are multiple threads. For each theme:
- Name the theme (bilingual)
- List the key referenced works and what gap or limitation they had
- Explain how this paper positions itself relative to them

If this section is very thin or absent in the paper, say so clearly and keep this section brief.

---

### 三、核心贡献 (Contributions)

Extract the paper's stated contributions **strictly following the original text** — do not paraphrase or invent contributions. If the paper has a bullet-pointed contributions list, use that directly.

Format each contribution as:

**[Contribution N]**

> *English (original):* "[exact quote from paper]"
>
> *中文翻译：*[准确翻译]

After listing all contributions, write a brief synthesis (3–5 sentences in Chinese) that explains the relationship between the contributions: Which is the primary one? Do the others support or enable it?

---

### 四、核心方案 (Methods)

This is the most important section. Explain the method in enough depth that a researcher could re-implement the key ideas.

Structure your explanation as follows:

#### 4.1 整体框架 (Overall Framework)
Describe the method's high-level design in 3–5 sentences. What are its major components? What is the data flow from input to output?

#### 4.2 架构图文字描述 (Architecture Description)
Describe the architecture as if drawing a diagram in words. Use a structured text diagram like:

```
[输入 Input]
    ↓
[模块A: 名称] → (功能描述)
    ↓
[模块B: 名称] → (功能描述)
    ↓        ↘
[分支1]    [分支2]
    ↓        ↓
[模块C: 名称] ← (汇合/融合)
    ↓
[输出 Output]
```

Annotate each node with what it does and what it takes as input/produces as output.

#### 4.3 关键技术细节 (Key Technical Details)
Explain the 2–4 most technically novel or important components. For each:
- What is the problem it solves within the method?
- How does it work (include relevant formulas if present in the paper)?
- Why is this design choice better than alternatives?

Write in Chinese, with English technical terms inline where standard.

#### 4.4 训练与推理 (Training & Inference)
Briefly describe: loss function(s), training data, optimizer settings if mentioned, and any notable inference-time techniques.

---

### 五、实验与消融 (Experiments & Ablation)

#### 5.1 实验设置 (Setup)
- Datasets used (name, size, domain)
- Baselines compared against
- Evaluation metrics
- Hardware/compute if mentioned

#### 5.2 主要结果 (Main Results)
Summarize the key comparison table(s). Highlight the most meaningful results. Use a compact markdown table if it helps clarity.

#### 5.3 消融实验 (Ablation Studies)
If the paper includes ablation studies:
- What components were ablated?
- What was the performance drop for each?
- What does this tell us about which components are most critical?

#### 5.4 总结与局限 (Conclusions & Limitations)
In 3–5 sentences (Chinese): What does this paper ultimately prove? What limitations do the authors acknowledge? What future work do they suggest?

---

## Step 3: Save to ~/Papers/

After writing the full analysis, save it as a markdown file:

1. **Create the directory if it does not exist**: `mkdir -p ~/Papers`
2. **Derive a filename** from the paper title: lowercase, spaces → hyphens, `.md` extension.
   - Example: "Attention Is All You Need" → `attention-is-all-you-need.md`
   - Example: "LoRA: Low-Rank Adaptation of Large Language Models" → `lora-low-rank-adaptation.md`
3. **Write the full report** (everything from 一 to 五) to `~/Papers/<filename>.md`
4. **Tell the user** the saved path, e.g.: `✓ 已保存至 ~/Papers/attention-is-all-you-need.md`

---

## Output Quality Standards

- **Accuracy first**: Never hallucinate details. If unsure, say so rather than fabricating.
- **Bilingual throughout**: Technical terms appear in English with Chinese explanation.
- **Depth over breadth in Methods**: Better to deeply explain 2 key components than superficially cover 6.
- **Be honest about gaps**: If the paper lacks a related work section or has weak ablations, say so.
- **Preserve authorial voice in Contributions**: The exact wording often matters — quote verbatim.
