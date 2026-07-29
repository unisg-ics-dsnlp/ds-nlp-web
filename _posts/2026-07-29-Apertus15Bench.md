---
title: LLM Benchmark Evaluation - Apertus 1.5-8B
author:
  - Götz-Henrik Wiegand
image: images/posts/Apertus15Bench-summary_average_performance.png
date: 2026-07-29
tags:
  - transformers
  - machine-learning
  - large language models
  - artificial intelligence
  - switzerland
---

## The Update

Last year we evaluated the newly released Apertus-8B-Instruct and shared our independent
benchmark impressions in [our first Apertus post](#). Since then, the Swiss AI Initiative (a
collaboration between ETH Zurich, EPFL, and the Swiss National Supercomputing Centre, CSCS) has
continued to develop the model family, and the 8B checkpoint has received a substantial update:
Apertus 1.5.

The underlying philosophy has not changed. Apertus 1.5 is still a fully open model. It is trained
on publicly available data, respects robots.txt, filters toxic and personally identifiable
content, and ships with reproduction artifacts. What changed is capability. During our evaluation
we observed three concrete additions in the 8B checkpoint (`swiss-ai/Apertus-v1.5-8B`):

- A "thinking" (deliberation) mode. The model can reason at length before producing an answer,
  controlled through the chat template.
- A longer context window of 262k tokens, up from 65k in the 1.0 instruct model.
- Multimodal components in the weights (vision and audio tokenizers) and a tool-calling chat template.

This post concentrates on two questions. First, how much does 1.5 improve over 1.0? Second, what
does thinking mode actually contribute? Rather than a broad cross-model leaderboard, we present a
generational study: Apertus 1.5 in normal and thinking mode, measured against the 1.0 instruct
baseline from last year under identical settings.

{% include section.html %}

{% include figure.html image="images/posts/Apertus15Bench-summary_average_performance.png" %}

{% include section.html %}

## Benchmarking the Model

Our goals were the same as before, and deliberately modest. We wanted to form our own impression
of the model and to check whether the reported improvements hold under a consistent, reproducible
harness. We again used [EleutherAI's Language Model Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness),
this time driven by vLLM through the official [Swiss-AI Apertus vLLM fork](https://github.com/swiss-ai/vllm). The image is required,
because a stock installation of vLLM cannot yet serve the 1.5 architecture.

All evaluations ran fully on one H100 GPU. Apertus 1.5 (normal and thinking) and the Apertus 1.0 baseline were evaluated
with identical settings, so the comparison between them is fair.

{% include section.html dark=true %}

## Open Models and Open Weight Models

We continue to use the terminology from the Technical Report. Open weight models publish their
weights but not their data or training artifacts (for example, the Qwen family). Open models aim
to make everything reproducible, including data-pipelines and training details (for example, OLMo, and
Apertus).

{% include section.html %}

**Quicklinks to Apertus Resources:**
- **Technical Report**: [Swiss-AI Apertus Technical Report](https://github.com/swiss-ai/apertus-tech-report)
- **Model Hub**: [Hugging Face: Swiss AI Models](https://huggingface.co/swiss-ai/)
- **Official Blog**: [Swiss AI Apertus Announcement](https://www.swiss-ai.org/apertus)

{% include section.html %}

**Important disclaimer:**
The benchmarks were produced with `swiss-ai/Apertus-v1.5-8B` and `swiss-ai/Apertus-8B-Instruct-2509`
from Hugging Face, using a subset of tasks from [EleutherAI's Language Model Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness)
served through the Swiss-AI vLLM image. The raw results and benchmark logs can be found in our
GitHub repository [unisg-ics-dsnlp/Apertus-1.5-Benchmark-Results](https://github.com/unisg-ics-dsnlp/Apertus-1.5-Benchmark-Results).

These benchmarks are not exhaustive and are intended only to provide an independent overview. For
comprehensive, official numbers please refer to the [Swiss-AI Apertus Technical Report](https://github.com/swiss-ai/apertus-tech-report).

{% include section.html %}

## Selected Benchmarks

We organized the evaluation into three parts, each addressing a different question.

**Part 1, core text capabilities.** Is 1.5 broadly better than 1.0? We used GPQA-Diamond (graduate
science), Global-MMLU (knowledge), IFEval (instruction following), MMLU-Pro (multi-domain
knowledge), LongBench (long context), and GSM8K (grade-school math).

**Part 2, multilingual breadth.** Does 1.5 improve across languages, and not only in English? We
used four families covering up to 14 languages: Global-MMLU (knowledge), Belebele (reading
comprehension), multilingual ARC-Challenge (reasoning), and multilingual HellaSwag (commonsense).

**Part 3, thinking and reasoning.** What does deliberation mode contribute? We assembled a
generative reasoning suite in which step-by-step reasoning is expected to help: MATH-500, AIME 2024
and 2025 (competition math), GPQA-main (science, chain-of-thought), and MGSM (multilingual
grade-school math). We ran 1.5 in both normal and thinking mode, with 1.0 as the baseline.

{% include section.html %}

## Part 1: Core Text Benchmarks

The following table compares Apertus 1.5 (normal), Apertus 1.5 thinking, and our re-measured 1.0
baseline on the full test sets. The final column reproduces the published Apertus 1.0 Technical
Report figure for the same 8B-Instruct model, where a comparable benchmark exists.

| Benchmark | Metric | Apertus **1.5** | 1.5 **thinking** | Apertus **1.0** (ours) | Apertus **1.0** (tech report) |
|---|---|--:|--:|--:|--:|
| GPQA-Diamond | acc, 0-shot | **29.8** | 26.3 ᵃ | 24.7 | 27.0 ᵇ |
| Global-MMLU (en) | acc | **71.0** | n/a ᶜ | 63.7 | 55.7 ᵈ |
| IFEval | prompt-strict acc | **85.8** | 72.1 | 68.9 | 71.7 ᵉ |
| MMLU-Pro | exact-match, 5-shot | **45.9** | 45.5 | 35.3 | n/a |
| LongBench | score | **44.2** | 43.4 | 35.3 | n/a |
| GSM8K | exact-match (flexible) | 79.6 | **85.3** | 60.2 | 62.9 |

**Footnotes**
- ᵃ Thinking mode cannot run multiple-choice tasks, so GPQA here is the CoT-generative variant (`gpqa_diamond_cot_zeroshot`, exact-match). It is therefore not directly comparable to the 0-shot multiple-choice number in the other columns.
- ᵇ The Technical Report uses GPQA Main (0-shot), a different and larger question subset than GPQA-Diamond.
- ᶜ Global-MMLU is multiple-choice, so it was not run in thinking mode.
- ᵈ The Report's "Global-MMLU" is a multilingual average, whereas ours is English only.
- ᵉ Our 1.0 prompt-loose IFEval is 71.7. Once the metric variant is matched, our number agrees with the Report.

The generational improvement is large. In normal mode, Apertus 1.5 improves on 1.0 on every core
benchmark: MMLU-Pro by 10.6 points, GSM8K by 19.4, IFEval by 16.9, LongBench by 8.9, Global-MMLU-en
by 7.3, and GPQA by 5.1. The gains are consistent across quite different task types.

One result is worth calling out as a validation check. Our re-measured 1.0 IFEval (68.9 strict,
71.7 loose) now agrees closely with the Technical Report value of 71.7. In last year's evaluation
our IFEval score was well below the Report. The setup used here, the official vLLM image together
with the model's own chat template, closes that gap. This is a useful reminder that evaluation
tooling has a measurable effect on results.

{% include figure.html image="images/posts/Apertus15Bench-core_results_grouped.png" %}

### Thinking mode in the core set

Thinking mode is not uniformly better. It helps math (GSM8K improves from 79.6 to 85.3). It is
roughly flat on broad knowledge and long context (MMLU-Pro, LongBench). And it reduces strict
instruction-following (IFEval falls from 85.8 to 72.1), because the model reasons at length instead
of following the requested output format exactly. This pattern recurs throughout the study.
Deliberation helps on hard multi-step problems, but it is not a free improvement on every task.

{% include section.html dark=true %}

## Part 2: Multilingual Breadth

Apertus is designed for broad language coverage. To test whether 1.5 delivers this across
languages, we ran four multiple-choice families. All values are accuracy. HellaSwag uses the
standard length-normalized `acc_norm`.

| Family | Metric | Languages | Apertus **1.5** | Apertus **1.0** | Δ |
|---|---|--:|--:|--:|--:|
| Global-MMLU | acc | 10 | **62.1** | 56.9 | +5.2 |
| Belebele (reading) | acc | 14 | **77.6** | 67.5 | +10.1 |
| ARC-Challenge (multilingual) | acc | 10 | **46.8** | 42.9 | +3.9 |
| HellaSwag (multilingual) | acc_norm | 9 | 53.6 | **61.9** | −8.3 |

{% include figure.html image="images/posts/Apertus15Bench-multilingual_grouped.png" %}

Apertus 1.5 improves on knowledge, reading comprehension, and reasoning in every language we
tested. The gains are broad rather than concentrated in a few high-resource languages. Belebele
shows this most clearly, with an average improvement of 10 points over 14 languages and every
language improving (for example, Arabic from 65 to 78, Italian from 69 to 80, and Hindi from 61 to
69).

Portuguese is a notable case. It was a weak point in 1.0 and is much stronger in 1.5. Global-MMLU
Portuguese improves from 39.5 to 63.2, a gain of 23.7 points, and Belebele Portuguese from 57.2 to
78.7, a gain of 21.4 points. These are the largest per-language changes in the study.

{% include figure.html image="images/posts/Apertus15Bench-multilingual_by_language.png" %}

HellaSwag is the exception. Here 1.0 scores higher than 1.5 in every language, with an average
difference of 8.3 points on `acc_norm`. This is a consistent regression. One interpretation is that
1.5 traded some surface-level sentence-completion performance for its gains in knowledge, reading,
and reasoning. We report it directly rather than averaging it away.

{% include section.html %}

## Part 3: Thinking and Reasoning

The final question is what thinking mode contributes on problems that require multi-step reasoning.
We built a generative suite in which deliberation is expected to help, and ran Apertus 1.5 in both
normal and thinking mode, with 1.0 as the baseline (1.0 has no thinking mode). Normal mode was
given a generation budget large enough that math solutions are not truncated.

| Benchmark | Metric | Apertus **1.0** | 1.5 **normal** | 1.5 **thinking** |
|---|---|--:|--:|--:|
| MATH-500 | math_verify (symbolic) | 20.2 | 54.0 | **71.0** |
| AIME 2024 | exact-match | 0.0 | 10.0 | **20.0** |
| AIME 2025 | exact-match | 0.0 | 13.3 | **20.0** |
| GPQA-main | exact-match (CoT) | 26.3 | **29.7** | 25.0 |
| MGSM (avg, 7 languages) | exact-match | 50.4 | 68.6 | **79.1** |

{% include figure.html image="images/posts/Apertus15Bench-reasoning_grouped.png" %}

Two observations follow.

First, most of the improvement is already present in normal mode, before thinking is enabled. In
normal mode, 1.5 substantially improves the model's math ability. MATH-500 rises from 20.2 to 54.0,
and MGSM from 50.4 to 68.6. On the AIME competition sets, 1.0 solves none of the problems in either
year, whereas 1.5 solves 10 to 13 percent. The 1.0 instruct model could not solve competition math
problems, and 1.5 can.

Second, thinking mode adds a further, sizable gain on top. MATH-500 reaches 71.0, AIME roughly
doubles to 20 percent, and MGSM reaches 79.1, improving in all seven languages (French by 17.6
points, Russian by 17.2). For hard multi-step math, thinking mode is clearly the right setting.

The exception is science knowledge. GPQA-main is flat to slightly lower under thinking, moving from
29.7 to 25.0. Thinking mode helps reasoning rather than factual recall, which is consistent with
MMLU-Pro being flat in Part 1.

### A metric caveat

On MATH-500, the strict `\boxed{}` answer-extraction metric falls to 2.8 percent in thinking mode.
This is not because the answers are wrong. The longer deliberation output does not place the final
answer where the rigid parser expects it. The symbolic-equivalence metric (`math_verify`) recovers
the true score of 71.0 percent. This is the same effect as the IFEval drop in Part 1. Thinking mode
can break brittle exact-format metrics while improving the underlying answer. When evaluating a
reasoning model, it is important to use a semantics-aware metric. Otherwise the model's capability
is badly under-reported.

{% include section.html %}

## Apertus 1.5 in Context

The results so far are generational, comparing 1.5 against 1.0. A natural follow-up question is how
Apertus 1.5 stands among other openly available models of a similar size. The figure below places it
next to a selection of open models, which publish their training data as well as their weights (for
example OLMo), and open weight models, which publish weights only (for example Qwen, Llama, Gemma,
and Nemotron), on four widely reported benchmarks.

{% include figure.html image="images/posts/Apertus15Bench-open_model_comparison.png" %}

Two cautions apply. First, only the Apertus numbers were produced by our harness. The other values
are taken from public model cards, technical reports, and the Open LLM Leaderboard, and they were
measured under different settings. Our own IFEval experience earlier in this post shows that tooling
alone can shift a score by more than ten points, so the cross-model bars are indicative rather than
definitive. Second, the ranking is mixed, which is what we expected. Apertus 1.5 is strong on
instruction following and roughly mid-pack on grade-school math, and it trails the strongest open
weight models such as Qwen2.5 on MMLU-Pro and GPQA. For a fully open model this is a reasonable
position, and it fits the argument we started with. The value of an open model lies not only in a
single score, but in the fact that anyone can reproduce it and build on it.

{% include section.html %}

## Key Observations

- **A clear generational improvement.** Across the core text benchmarks, Apertus 1.5 improves on
  1.0 by roughly 9 to 19 points on MMLU-Pro, GSM8K, IFEval, and LongBench. The gains are consistent
  across different task types.
- **Broadly multilingual.** Reading comprehension (Belebele) and knowledge (Global-MMLU) improve in
  every language tested. Portuguese shows the single largest improvement.
- **Thinking mode is a targeted tool.** It produces large gains on multi-step math (MATH-500 by 17
  points, AIME roughly doubled, MGSM by 10.5). It is neutral to slightly negative on knowledge and
  strict-format tasks. Enable it for reasoning, and disable it for instruction-following.
- **One regression.** Multilingual HellaSwag falls by about 8 points. It is worth monitoring, but it
  is narrow compared to the breadth of the gains.
- **Evaluation tooling matters.** Matching the model's own chat template and using a semantics-aware
  metric moved our 1.0 IFEval into agreement with the Technical Report, and turned an apparent
  MATH-500 collapse back into a 71 percent result. A fair comparison depends on careful, identical
  settings.

{% include section.html %}

{% include figure.html image="images/posts/Apertus15Bench-summary_average_performance_by_benchmark.png" %}

{% include section.html %}

## References

[1] Swiss AI Initiative. (2025). *Apertus Technical Report*. GitHub Repository. [https://github.com/swiss-ai/apertus-tech-report](https://github.com/swiss-ai/apertus-tech-report)

[2] Gao, L., Tow, J., Biderman, S., et al. (2023). *A framework for few-shot language model evaluation*. Zenodo. [https://doi.org/10.5281/zenodo.10256836](https://doi.org/10.5281/zenodo.10256836)

[3] Zhou, J., Lu, T., Mishra, S., et al. (2023). *Instruction-Following Evaluation for Large Language Models (IFEval)*. arXiv. [https://arxiv.org/abs/2311.07911](https://arxiv.org/abs/2311.07911)

[4] Wang, Y., Ma, X., Zhang, G., et al. (2024). *MMLU-Pro: A More Robust and Challenging Multi-Task Language Understanding Benchmark*. arXiv. [https://arxiv.org/abs/2406.01574](https://arxiv.org/abs/2406.01574)

[5] Bandarkar, L., Liang, D., Muller, B., et al. (2023). *The Belebele Benchmark: a Parallel Reading Comprehension Dataset in 122 Language Variants*. arXiv. [https://arxiv.org/abs/2308.16884](https://arxiv.org/abs/2308.16884)

[6] Singh, S., Romanou, A., Fourrier, C., et al. (2024). *Global-MMLU: Understanding and Addressing Cultural and Linguistic Biases in Multilingual Evaluation*. arXiv. [https://arxiv.org/abs/2412.03304](https://arxiv.org/abs/2412.03304)

[7] Hendrycks, D., Burns, C., Kadavath, S., et al. (2021). *Measuring Mathematical Problem Solving With the MATH Dataset*. arXiv. [https://arxiv.org/abs/2103.03874](https://arxiv.org/abs/2103.03874)

[8] Shi, F., Suzgun, M., Freitag, M., et al. (2022). *Language Models are Multilingual Chain-of-Thought Reasoners (MGSM)*. arXiv. [https://arxiv.org/abs/2210.03057](https://arxiv.org/abs/2210.03057)

[9] Rein, D., Hou, B. L., Stickland, A. C., et al. (2023). *GPQA: A Graduate-Level Google-Proof Q&A Benchmark*. arXiv. [https://arxiv.org/abs/2311.12022](https://arxiv.org/abs/2311.12022)

{% include section.html %}

{% include figure.html image="images/posts/Apertus15Bench-summary_average_performance.png" width="50%" %}
