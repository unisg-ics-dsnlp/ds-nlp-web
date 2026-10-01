---
title: Apertus 1.5 Under Test - Real Business Tasks
author:
  - Siegfried Handschuh
image: images/posts/Apertus15Business-radar.png
date: 2026-10-01
tags:
  - large language models
  - evaluation
  - benchmarks
  - artificial intelligence
  - switzerland
---

## Why Another Benchmark Post?

In [our previous post]({% link _posts/2026-07-29-Apertus15Bench.md %}), we looked at Apertus 1.5 through the lens
of academic benchmarks: MMLU-Pro, IFEval, GSM8K, multilingual reading comprehension, and so on. The
generation step from 1.0 to 1.5 turned out to be substantial.

But academic benchmarks answer a specific question: *what does the model know?* When companies ask us
about Apertus, they usually ask a different one: *can it do the job?* Can it route a customer request
to the right team, read the total off a receipt, pre-sort contract clauses for a legal review, or fill
in an invoice form?

So, in this post, we take Apertus 1.5 into the back office. We ran six public benchmarks built from
real business data, about 3,000 cases per model, against six other models, including two frontier
systems. And since this is an Apertus post, we then take the model apart: which mistakes does it
make, and what can you do about them?

If you are short on time, here is the summary:

1. **Against the frontier**, Apertus 1.5 8B reaches 79 percent of GPT-5.6-Luna's overall score, the 70B
   model 83 percent. The gap is small on documents and social media, and large on long category lists.
2. **Among open models of its size**, Apertus 1.5 8B ranks behind Qwen 3.5 9B, Qwen 2.5 7B and Llama 3.1
   8B, and ahead of Apertus 1.0. It is the only fully open model in that group.
3. **Its strength is documents.** On invoices it is level with Qwen 3.5 9B and with its own 70B sibling;
   on receipts it reads the date correctly 99 times out of 100.
4. **Its errors fall into three fixable types**: answers outside the allowed list, favourite labels, and
   inventing values when information is missing.
5. **It runs on a Mac.** Apertus 1.5 8B ran locally on an Apple M5 Max, about 1.3 seconds per request,
   without a single failed request.

{% include section.html %}

{% include figure.html image="images/posts/Apertus15Business-overall.png" caption="Overall score across six business benchmarks. The line shows the 95 % bootstrap interval; differences larger than about two points are real." %}

{% include section.html %}

## The Setup in Brief

We compared eight models: Apertus 1.5 8B and 70B, Apertus 1.0 8B, Qwen 3.5 9B, Qwen 2.5 7B, Llama 3.1 8B,
and two frontier models via their APIs, GPT-5.6-Luna and Claude Haiku 4.5.

A few design choices are worth spelling out, because they matter for how to read the numbers:

- **Zero-shot, one prompt per task, identical for all models.** We did not tune prompts for any model.
- **Temperature 0** wherever the API allows it. GPT-5.6 only accepts its default temperature.
- **Thinking modes off** for every model. We come back to thinking at the end of the post.
- **Official metrics.** Each benchmark is scored with the metric its authors defined (accuracy,
  macro-F1, F1 of the ironic class, and so on). The overall score is the mean of the six benchmarks.
- **Where the models ran.** Most open models ran through Hugging Face Inference Providers. Apertus 1.5 8B
  ran locally on a Mac, because it was no longer served by any provider when we ran our tests (more on
  that below). Qwen 3.5 9B also ran locally, 4-bit quantised through Ollama, which if anything puts it at
  a disadvantage.

{% include section.html dark=true %}

## Six Benchmarks, Six Business Questions

Before looking at results, it helps to know what each benchmark actually asks. Here is a short tour,
with one real example from each.

**Banking77** (PolyAI, 2020) contains 13,083 real customer requests to an online bank, each labelled with
one of 77 intents. The difficulty is the neighbourhood: «My card hasn't arrived yet» (`card_arrival`)
and «Ordered awhile back, what is the ETA?» (`card_delivery_estimate`) are two different intents.
*Business question: can the model route a request to the right process?* We used all 77 intents,
616 requests.

**SROIE** (ICDAR 2019) consists of about 1,000 scanned receipts from Malaysian shops, run through OCR. The
model extracts company, date, address and total from noisy text such as
`... SUB-TOTAL 61.99 ... GST 3.72 ... TOTAL 65.70`, where it has to pick the amount actually paid.
*Business question: expense booking from a photo.* All 361 test receipts.

**CUAD** (The Atticus Project and UC Berkeley, NeurIPS 2021) covers 510 commercial contracts filed with the
US SEC, annotated by lawyers in 41 clause types. We give the model a marked clause and ask for its type,
for example «... governed by the laws ... of the State of Florida ...» → `Governing Law`. *Business
question: pre-sorting contracts for a legal review.* 294 clauses.

**RAFT** (Ought and partners, NeurIPS 2021) bundles eleven classification jobs that organisations actually
pay people to do: is a tweet a complaint, does a sentence overrule a precedent, is a clause in the terms
of service unfair to consumers? *Business question: how well does the model handle a new back-office
task on day one?* All 11 tasks, 50 labelled cases each.

**TweetEval** (Cardiff University and Snap, EMNLP Findings 2020) collects seven SemEval tasks on tweets:
irony, emotion, hate speech, offensive language, sentiment, stance and emoji prediction. «Leaving whilst
its dark is fun. #not» is irony. *Business question: social media and customer-voice monitoring.*
150 tweets per task.

**DocILE** (Rossum with the Czech Technical University, ICDAR 2023) contains about 6,700 real business
documents, mostly invoices and orders. The model fills fifteen fields: vendor and customer, document and
order numbers, dates, payment terms, net, tax, gross and amount due. *Business question: invoice capture
without manual typing.* 150 documents.

> **Note:** Two benchmarks keep their test labels private (RAFT and DocILE), so we used their public
> labelled splits. And we test CUAD as classification of a marked clause, while the original task asks
> models to *find* the clause in a full contract.

{% include section.html %}

## The Results

| Model | Overall | Banking77 | SROIE | CUAD | RAFT | TweetEval | DocILE |
|---|--:|--:|--:|--:|--:|--:|--:|
| GPT-5.6-Luna | **73.5** | **85** | **80** | **67** | 74 | 62 | **73** |
| Claude Haiku 4.5 | 71.8 | 78 | **80** | 61 | **76** | **65** | 71 |
| Qwen 3.5 9B (4-bit) | 65.3 | 72 | 74 | 54 | 71 | 58 | 63 |
| Qwen 2.5 7B | 61.1 | 65 | 69 | 47 | 68 | 60 | 56 |
| **Apertus 1.5 70B** | 61.0 | 62 | 73 | 42 | 67 | 62 | 60 |
| Llama 3.1 8B | 59.6 | 64 | 68 | 44 | 69 | 58 | 56 |
| **Apertus 1.5 8B** | 57.9 | 56 | 71 | 37 | 65 | 56 | 62 |
| Apertus 1.0 8B | 55.3 | 56 | 64 | 46 | 57 | 52 | 56 |

The table answers two different questions, and it is worth keeping them apart: how far is Apertus from
the frontier, and how does it compare to the open models it actually competes with?

### Question 1: How far is Apertus from the frontier?

Further than on academic benchmarks. Apertus 1.5 8B reaches 79 percent of GPT-5.6-Luna's overall score,
the 70B model 83 percent. In absolute terms, that is a gap of 14 to 16 points for the 8B model and 11 to
13 points for the 70B model, all clearly outside the statistical noise.

The gap is very uneven across tasks, though:

| Share of GPT-5.6-Luna's score | Banking77 | SROIE | CUAD | RAFT | TweetEval | DocILE | Overall |
|---|--:|--:|--:|--:|--:|--:|--:|
| Apertus 1.5 8B | 66 % | 88 % | 55 % | 89 % | 90 % | 85 % | **79 %** |
| Apertus 1.5 70B | 74 % | 90 % | 63 % | 91 % | 99 % | 82 % | **83 %** |

On receipts, back-office tasks, social media and invoices, Apertus gets within 10 to 15 percent of the
frontier; the 70B model even matches GPT-5.6 on TweetEval. The real gap sits in the two tasks with long
lists of fine-grained categories: contracts (42 types) and banking intents (77 types). That is the area to
watch in the next release.

As a side note, in an earlier pilot with fewer, partly synthetic cases, we had seen frontier models at only
about 60 percent and concluded that the tasks were "hard for everyone". That conclusion did not survive a
larger and cleaner test set (more on this below).

### Question 2: How does Apertus do among open models of its size?

Here the honest answer is: behind the pack, but with a clear strength. The table below compares Apertus
1.5 8B to the other open models of similar size. Each difference comes from a paired comparison on the
same cases, with a 95 percent bootstrap interval.

| Compared to Apertus 1.5 8B | Overall | Difference | 95 % interval |
|---|--:|--:|--:|
| Qwen 3.5 9B (4-bit) | 65.3 | +7.4 | +5.8 to +9.0 |
| Qwen 2.5 7B | 61.1 | +3.1 | +1.6 to +4.6 |
| Llama 3.1 8B | 59.6 | +1.6 | +0.2 to +3.2 |
| **Apertus 1.5 8B** | **57.9** |  |  |
| Apertus 1.0 8B | 55.3 | −2.6 | −4.2 to −0.9 |

All differences are significant. Overall, Apertus 1.5 8B ranks behind Qwen 3.5, Qwen 2.5 and Llama 3.1,
and ahead of its predecessor. Per benchmark, the picture is more nuanced. Among these five models of
similar size, Apertus 1.5 8B ranks:

- **2nd on documents**: on DocILE (invoices) it is level with Qwen 3.5 (62.3 against 62.8), and on SROIE
  (receipts) it beats Qwen 2.5 and Llama 3.1;
- **4th** on RAFT, TweetEval and Banking77;
- **5th on contracts**, even behind Apertus 1.0.

The larger Apertus 1.5 70B does not change the overall picture: it is 4.3 points behind Qwen 3.5 9B,
level with Qwen 2.5 7B (a model about two years older), and 1.4 points ahead of Llama 3.1 8B.

One caveat on what "peer group" means. In [our previous post]({% link _posts/2026-07-29-Apertus15Bench.md %})
we distinguished *open-weight* models (Qwen, Llama: weights published) from *fully open* models (Apertus,
OLMo: weights, data and training pipeline published). All competitors in this table are open-weight.
Apertus is the only fully open model in our comparison; a run with OLMo 3, its natural peer, is still
missing. Being fully open is a cost that open-weight models do not pay, for example in which training
data can be used, and it should be part of how these numbers are read.

Finally, the generation step within Apertus is real but modest: 1.5 is 2.6 points ahead of 1.0 at 8B.
The interesting part is *where* it gains and loses, which brings us to the profile.

{% include section.html dark=true %}

## Apertus 1.5 8B at a Glance

{% include figure.html image="images/posts/Apertus15Business-radar.png" caption="Apertus 1.5 8B (blue) against Qwen 3.5 9B of the same size class (orange) and GPT-5.6-Luna as the ceiling (dashed). The bottom axis shows how often a model answers null when we delete the date or total from a receipt." %}

The radar chart tells the story in one picture. At the top, invoices and receipts: Apertus is level with
Qwen 3.5 and not far from GPT. On the left, the long category lists: contracts and customer requests,
where the gap is largest. And at the bottom, an axis that matters a lot in practice: does the model say
`null` when a piece of information is missing? Apertus 1.5 8B never does.

Let's make this concrete with five real cases.

{% include section.html %}

## Five Cases from the Data

### 1. Customer requests: clear ones work, look-alikes do not

Apertus 1.5 8B routes requests with a clear, unique reason perfectly. Every test case for intents such as
«Please terminate my account» (`terminate_account`), «How old do I have to be?» (`age_limit`) or «What can
I do if contactless doesn't work?» (`contactless_not_working`) was correct.

But Banking77 has three intents about identity checks: `verify_my_identity`, `why_verify_identity` and
`unable_to_verify_identity`. Apertus puts all 24 such requests into the first one. «I'd rather not verify
my identity» and «I can't verify my ID» both end up as `verify_my_identity`. Qwen 3.5 separates them
much better.

**Tip:** if your categories are distinct, Apertus 8B is a good router. If several sound alike, merge them
or put one example per category into the prompt.

### 2. A purchase order: numbers yes, roles no

Here is the OCR text of a real purchase order from DocILE, sent by a small media company to a radio
station:

```text
Neighborhood Research and Media
PO BOX297, RODANTHE, NC 27968 US
Purchase Order
VENDOR ... P.O.NO. 7601
Mark Turcotte · WSB · DATE 08/06/2020
...
TOTAL $3,485.00
```

Apertus 1.5 8B extracts the date and the total exactly right. That is typical: on invoices it gets the
net, tax and due amounts right in 83 to 94 percent of cases. But it takes the letterhead as the vendor.
On a purchase order, the letterhead is the *buyer*; the vendor is listed under «VENDOR». And as the order
number, it returns the words «Purchase Order» instead of 7601.

**Tip:** amounts and dates you can largely trust. Roles need a hint about the document type in the
prompt, or a rule that checks them.

### 3. Contracts: one label for everything

> «The Reseller shall be entitled to enter into agreements with its subsidiaries and affiliates to act
> as sub-distributors and/or selling agents of the Products in the Territory.»

In CUAD, this is an `Affiliate License-Licensee` clause. Apertus 1.5 70B gets it right. Apertus 1.5 8B
answers `License Grant`, the most general licence category. This is a pattern: across 294 clauses, the 8B
model says `License Grant` 62 times, for only six real license grants. Occasionally it also invents types
that are not on the list, such as «Severability» or «Limited Liability».

**Tip:** with 40+ categories, classify in two steps (first «licence, liability, termination?», then the
fine type), and never pass a label on without checking it against the list.

### 4. Irony: it understands, but uses its own word

> «Just paid $2.59 for gas! #ThanksObama #sarcasm»

The allowed answers are `irony` and `non_irony`. Apertus 1.5 8B answers `sarcasm`. It clearly understood
the tweet; it just did not use a word from the list. This happened in 9 of 150 irony cases, and each one
counts as wrong. Apertus 70B answers `irony` in all nine.

**Tip:** this is the cheapest error to fix. Map obvious synonyms in code, or use constrained decoding so
the model can only produce allowed labels.

### 5. The deleted date: Apertus 8B makes one up

To test how models handle missing information, we took 80 real receipts and deleted the date or the total
from the text. The correct answer is `null`.

On a bakery receipt from 2017 with the date removed, Apertus 1.5 8B answered `10/04/2024`, a date that
appears nowhere in the document. On other receipts it returned the time of purchase as the date, or a
cash amount as the total. It never answered `null`.

| Model | Correct `null` when data is missing |
|---|--:|
| Claude Haiku 4.5 | 53 % |
| GPT-5.6-Luna | 49 % |
| Qwen 3.5 9B | 40 % |
| Apertus 1.5 70B | 34 % |
| Llama 3.1 8B | 0 % |
| Apertus 1.5 8B | 0 % |
| Apertus 1.0 8B | 0 % |

Interestingly, Qwen 3.5 9B, the same size as Apertus 8B, gets it right in 40 percent of cases. So this is
not a hard size limit; it is something a small model can learn.

**Tip:** in a booking workflow, a confident wrong value is worse than an empty field. Check every
extracted value against the source document before it is booked.

{% include section.html dark=true %}

## What Does the 70B Model Add?

{% include figure.html image="images/posts/Apertus15Business-8b-vs-70b.png" caption="Apertus 1.5 8B (blue) against Apertus 1.5 70B (orange)." %}

Nine times the parameters, and on everyday document work surprisingly little difference: on invoices,
receipts and the RAFT back-office tasks, 8B and 70B are about two points apart, and on invoices the small
model is even slightly ahead.

What the large model adds is *judgement*: it answers `null` when information is missing (34 % against
0 %), it recognises irony and uses the right word (78 against 61 on the irony task), and it picks finer
contract categories instead of the general one.

**Rule of thumb:** for extraction and routing with clear categories, start with the 8B model. For tasks
where the model has to notice what is *absent* or read between the lines, use the 70B model or add
explicit checks.

{% include section.html %}

## Bonus: The Small Apertus Runs on a Mac

When we reran our tests on 30 September, Apertus 1.5 8B was no longer served by any Hugging Face Inference
Provider. Standard `transformers` and MLX releases do not support the 1.5 architecture yet either. What
does work is the Swiss AI fork of `transformers`, which the model card points to:

```bash
pip install "transformers[torch,vision,audio] @ git+https://github.com/swiss-ai/transformers.git@3797303dda74844e3d1f8977ff5518bb91f818b4"
```

```python
import torch
from transformers import AutoModelForMultimodalLM, AutoProcessor

model_id = "swiss-ai/Apertus-v1.5-8B"
processor = AutoProcessor.from_pretrained(model_id)
model = AutoModelForMultimodalLM.from_pretrained(model_id, dtype="auto").to("mps").eval()

messages = [{"role": "user", "content": "Which department handles a lost card?"}]
inputs = processor.apply_chat_template(
    messages, add_generation_prompt=True, tokenize=True,
    return_dict=True, return_tensors="pt", enable_thinking=False,
).to(model.device)

with torch.inference_mode():
    out = model.generate(**inputs, max_new_tokens=64, do_sample=False)
print(processor.decode(out[0, inputs["input_ids"].shape[-1]:], skip_special_tokens=True))
```

On an Apple M5 Max with 128 GB of memory, in bf16, the model handled about 3,100 benchmark cases with a
median of 1.3 seconds per request and not a single failed request. No API cost, and no document leaves the
machine, which, for banks, insurers and public administrations, is exactly the argument for an open Swiss
model. Note that we only tested text; image and audio input on Apple Silicon remains untested.

{% include section.html dark=true %}

## A Quick Look at Thinking Mode

Apertus 1.5 has a thinking mode. Does reasoning before answering fix the error types above? We ran a
targeted test on 409 hard cases (look-alike banking intents, contract clauses, irony, missing data), with
and without thinking. Two observations surprised us.

First, **Apertus decides for itself whether to think**, and a strict format instruction («answer only with
the category») suppresses reasoning even when thinking is enabled. We had to add a sentence asking the
model to think step by step first.

Second, **for Apertus 1.5 70B, thinking barely helps in our benchmark setting**: between −4 and +5 points
depending on the task (−1 to +5 if we only count answers where the reasoning finished),
while the answer time on contract clauses grows from 1.5 to 27 seconds, and in one of nine contract
cases the reasoning did not finish within 3,072 tokens. GPT-5.6 and Claude Haiku 4.5 benefit more,
mainly on irony (+5 to +12 points).

This should not be read as "thinking is useless". Our tasks are short classification and extraction
problems, where the answer depends on recognising a pattern rather than on several steps of inference.
Where a task does require multi-step reasoning, thinking pays off clearly: in
[our academic evaluation]({% link _posts/2026-07-29-Apertus15Bench.md %}), thinking mode lifted Apertus 1.5 8B
by 17 points on MATH-500 and by 10.5 points on multilingual grade-school math (MGSM).

We did not complete the run for Apertus 1.5 8B: with thinking, it needed 15 to 50 seconds per case on our
Mac, which made the full test impractical. Whether thinking fixes the small model's error types remains
an open question, and a good one for anyone with a GPU and an afternoon to spare.

{% include section.html %}

## Lessons About Evaluation Itself

Building this benchmark taught us as much about evaluation as about models. Four things moved our
numbers by more than many of the gaps between models:

1. **Scoring.** In our pilot we compared extracted amounts as strings, so `133.7` did not match
   `"133.70"`. Comparing amounts as numbers and dates as dates moved Llama 3.1 by twelve points.
2. **Data.** A support-ticket dataset we initially treated as real turned out to be synthetic. We removed
   it.
3. **Providers.** One hosting provider silently dropped tokens from Gemma 2 outputs (`declinedard_payment`
   instead of `declined_card_payment`, `8480` instead of `84.80`). The model was fine; the serving stack
   was not. Gemma is therefore not in our table.
4. **Gold labels.** SROIE receipts often lack part of the address in the OCR text, and DocILE does not
   annotate every printed field. We adapted the scoring to what is actually reachable from the text.

If there is one general lesson, it is this: *read the raw outputs before you trust the score*.

{% include section.html dark=true %}

## Limitations

These benchmarks are an independent snapshot, not an exhaustive evaluation. In particular:

- All prompts are zero-shot. Few-shot prompting or fine-tuning would likely change the picture,
  especially for long category lists.
- The instructions were in German and the documents in English, which reflects a typical Swiss setup but
  may favour multilingual models.
- Qwen 3.5 9B ran 4-bit quantised, all Apertus models in full precision.
- We report a single run per model. GPT-5.6 cannot be run at temperature 0, so its numbers are one sample.
- RAFT and DocILE results come from their public labelled splits, not their hidden test sets.

{% include section.html %}

## Key Takeaways

- **Apertus 1.5 8B is a solid, privately deployable document model.** Dates and amounts on invoices and
  receipts are its strength, and it runs on a single Mac.
- **Its weaknesses are specific and fixable.** Validate labels against the allowed list, split long
  category lists into two steps, and never trust a filled field without checking it against the source.
- **Size buys judgement, not reading.** The 70B model mainly adds the ability to notice what is missing.
- **The open-model gap is real.** Qwen 3.5 9B shows what is possible at the same size, which is good news:
  it means there is headroom for the next Apertus release.

We are presenting these results at the «Hack Apertus» session on 2 October 2026. If you are working on any of
the open questions above, such as format guards, null handling or thinking mode, we would be happy to hear
from you.

**Quicklinks:**
- **Our academic benchmark post**: [LLM Benchmark Evaluation - Apertus 1.5-8B]({% link _posts/2026-07-29-Apertus15Bench.md %})
- **Model Hub**: [Hugging Face: Swiss AI Models](https://huggingface.co/swiss-ai/)
- **Benchmarks**: [Banking77](https://huggingface.co/datasets/legacy-datasets/banking77) ·
  [SROIE](https://huggingface.co/datasets/jsdnrs/ICDAR2019-SROIE) ·
  [CUAD](https://huggingface.co/datasets/theatticusproject/cuad) ·
  [RAFT](https://huggingface.co/datasets/ought/raft) ·
  [TweetEval](https://huggingface.co/datasets/cardiffnlp/tweet_eval) ·
  [DocILE](https://github.com/rossumai/docile)
