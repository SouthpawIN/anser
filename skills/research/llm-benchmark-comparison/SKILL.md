---
name: llm-benchmark-comparison
title: LLM Benchmark Comparison Research
description: Research published AI/LLM benchmark scores from scattered sources and synthesize them into validated, cited comparison tables.
triggers:
  - User asks to compare models on specific benchmarks like MMLU, SWE-bench, HumanEval, TerminalBench, LiveCodeBench
  - User asks for benchmark scores for one or more AI/LLM models
  - User wants a shareable table of model performance on standard evals
  - User asks how model X compares to model Y on benchmarks
steps:
  - Identify exact models and benchmarks. Note variants like SWE-bench Verified vs Pro, or MMLU vs MMLU-Pro.
  - Search each model and benchmark combination individually with targeted queries.
  - Prioritize primary sources in this order - official model blog, HuggingFace model card README, official API docs.
  - If primary sources are incomplete, cross-check reputable aggregators like vals.ai, openlm.ai, llm-stats.com, benchlm.ai.
  - Note the model mode or reasoning effort for each score. Compare like-for-like when possible.
  - Validate consistency across sources. Prefer official blogs or HF cards when numbers conflict.
  - Compile into a clean markdown table with a Quick Takeaways summary and a Sources section.
pitfalls:
  - Benchmark versioning. SWE-bench has Verified, Pro, and Multilingual variants. TerminalBench 2.0 differs from 1.0. Always verify the variant.
  - Mode confusion. Many frontier models report very different scores in thinking vs non-thinking modes. Note the mode and do not mix them without warning.
  - Stale aggregator data. Third-party sites may lag behind model releases. Always verify against the primary source.
  - Different evaluation harnesses. Some scores are self-reported by the provider, others from independent evals. Note the evaluator when it matters.
  - Missing data. Not all models are evaluated on all benchmarks. Leave cells blank or N/A rather than guessing.
examples:
  - Show me MMLU, TerminalBench, and SWE benchmarks for Qwen 3.6 27B vs DeepSeek v4 Pro vs GLM 5.1 vs Kimi K2.6
  - What is the SWE-bench Verified score for Claude Opus 4.6?
  - Compare GPT-5.4 and Gemini 3.1 Pro on coding benchmarks
---

# LLM Benchmark Comparison Research

When the user wants to compare AI/LLM models on standard benchmarks, follow this workflow to gather data from scattered sources and produce a clean, cited comparison table.

## 1. Decompose the Request
Extract the model names and benchmark names. Note any benchmark variants (e.g., SWE-bench Verified / Pro / Multilingual, MMLU-Pro vs MMLU-Redux).

## 2. Search Strategy
Run targeted web searches for each model and benchmark pair. Example queries:
- "ModelName" "BenchmarkName" score
- "ModelName" "BenchmarkName" site:huggingface.co
- "ModelName" site:official-blog.com

## 3. Source Hierarchy (most to least reliable)
1. Official model blog or tech report (e.g., qwen.ai/blog, z.ai/blog, DeepSeek HF discussion tables).
2. HuggingFace model card README (often contains exhaustive evaluation tables).
3. Official docs or API pages (e.g., Moonshot AI platform docs).
4. Reputable aggregators (Vals AI, OpenLM, LLM Stats, BenchLM). Use these to fill gaps, but verify against primary sources when numbers conflict.

## 4. Normalize and Validate
- Record the model mode for each score (e.g., Thinking, Max, xHigh, Non-think).
- Ensure you are comparing the same benchmark variant across models.
- If sources conflict, prefer the official source. Flag unresolved discrepancies in the output.

## 5. Output Format
Present results as:
- A clean markdown table with models as rows and benchmarks as columns.
- A Quick Takeaways section with 2-4 bullets highlighting leaders and notable surprises.
- A Sources section citing each data point (blog URL, HF card, aggregator page).

## Common Benchmark Variants to Watch
| Benchmark Family | Common Variants |
|---|---|
| MMLU | MMLU, MMLU-Pro, MMLU-Redux |
| SWE-bench | Verified, Pro, Multilingual |
| TerminalBench | 1.0, 2.0 (often with different harnesses like Terminus-2 or Claude Code) |
| AIME | AIME 2024, AIME 2025, AIME 2026 |
| HMMT | Nov 2025, Feb 2026, etc. |

## Pitfall: Reasoning Mode Mismatch
Many models report scores in thinking or max effort mode that are 10-30 points higher than their default mode. Always label the mode in the table header or footnotes, and do not mix modes within the same comparison unless explicitly requested.