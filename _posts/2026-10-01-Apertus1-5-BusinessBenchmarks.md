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
  - vision
  - multimodal
  - document understanding
  - artificial intelligence
  - switzerland
---

## From Exams to the Back Office

Apertus 1.5 8B is a solid document model that you can deploy privately, on a single Mac if need be. Its
weaknesses are specific, and they can be fixed. This post shows both, measured on real business data.

We wrote it with the participants of [«Hack Apertus»](https://hackapertus.ch/) in mind, who are building
on Apertus 1.5 until 16 October. If you are looking for a project, the error types below are a good place
to start, and the section [Ideas for Hack Apertus](#ideas-for-hack-apertus) lists a few concrete ones.

In [our previous post]({% link _posts/2026-07-29-Apertus15Bench.md %}) we tested Apertus 1.5 on academic
benchmarks such as MMLU-Pro, IFEval, GSM8K and multilingual reading comprehension. The step from 1.0 to
1.5 was large.

Academic benchmarks measure what a model knows. Companies that ask us about Apertus mostly want to know
something else: can it route a customer request to the right team, read the total off a receipt, sort
contract clauses before a legal review, or fill in an invoice form?

For this post we ran six public benchmarks built from real business data, roughly 3,000 cases per
model, and compared Apertus with six other models, two of them frontier systems. Because this is an
Apertus post, we then went through its mistakes one by one.

The short version:

1. Apertus 1.5 8B reaches 79 percent of GPT-5.6-Luna's overall score, the 70B model 83 percent. On
   documents and social media the gap is small; on long category lists it is large.
2. Among open models of its size, Apertus 1.5 8B comes after Qwen 3.5 9B, Qwen 2.5 7B and Llama 3.1 8B,
   and before Apertus 1.0. It is the only fully open model in that group.
3. Documents are its strong side. On invoices it matches Qwen 3.5 9B and its own 70B sibling, and on
   receipts it gets the date right 99 times out of 100.
4. Most of its errors are of three kinds, all fixable: answers that are not on the allowed list,
   a few labels it uses far too often, and values it invents when information is missing.
5. It runs on a Mac: about 1.3 seconds per request on an Apple M5 Max, with no failed requests.
6. Reading scanned receipts directly is its weak spot: it misreads dates and totals, which Qwen 3.5 9B
   reads almost without error.

{% include section.html %}

{% include figure.html image="images/posts/Apertus15Business-overall.png" caption="Overall score across six business benchmarks. The line shows the 95 % bootstrap interval; differences above about two points are real." %}

{% include section.html %}

## Setup

We compared eight models: Apertus 1.5 8B and 70B, Apertus 1.0 8B, Qwen 3.5 9B, Qwen 2.5 7B, Llama 3.1 8B,
and, through their APIs, GPT-5.6-Luna and Claude Haiku 4.5.

Some choices affect how the numbers should be read:

- All tasks are zero-shot, with one prompt per task that is the same for every model. We did not tune
  prompts for anyone.
- Temperature is 0 wherever the API allows it. GPT-5.6 only accepts its default temperature.
- Thinking modes are off for all models. We look at thinking separately near the end.
- Each benchmark is scored with its authors' metric (accuracy, macro-F1, F1 of the ironic class and so
  on). The overall score is the mean over the six benchmarks.
- Most open models ran through Hugging Face Inference Providers. Apertus 1.5 8B ran locally on a Mac,
  because no provider served it any more when we ran the tests. Qwen 3.5 9B also ran locally, 4-bit
  quantised through Ollama; if anything, that works against it.

{% include section.html dark=true %}

## Six Benchmarks, Six Business Questions

Each benchmark asks a different practical question. Below is one real example per benchmark.

{% include figure.html image="images/posts/Apertus15Business-benchmarks.png" caption="The six benchmarks, each with one real example from our test set and the sample size we used." %}

**Banking77** (PolyAI, 2020) has 13,083 real customer requests to an online bank, each labelled with one
of 77 intents. Many intents sit close together: «My card hasn't arrived yet» (`card_arrival`) and
«Ordered awhile back, what is the ETA?» (`card_delivery_estimate`) count as different. In business terms:
can the model send a request to the right process? We used all 77 intents and 616 requests.

**SROIE** (ICDAR 2019) has about 1,000 scanned receipts from Malaysian shops, passed through OCR. The
model reads company, date, address and total from noisy text like
`... SUB-TOTAL 61.99 ... GST 3.72 ... TOTAL 65.70` and has to pick the amount actually paid. Think expense
booking from a photo. We used all 361 test receipts.

**CUAD** (The Atticus Project and UC Berkeley, NeurIPS 2021) covers 510 commercial contracts filed with the
US SEC, annotated by lawyers in 41 clause types. We show the model a marked clause and ask for its type,
for example «... governed by the laws ... of the State of Florida ...» → `Governing Law`. This is the
first pass of a legal review. 294 clauses.

**RAFT** (Ought and partners, NeurIPS 2021) collects eleven classification jobs that organisations pay
people to do: is this tweet a complaint, does this sentence overrule a precedent, is this clause in the
terms of service unfair to consumers? It shows how a model copes with an unfamiliar back-office task on
the first day. All 11 tasks, 50 labelled cases each.

**TweetEval** (Cardiff University and Snap, Findings of EMNLP 2020) gathers seven SemEval tasks on tweets:
irony, emotion, hate speech, offensive language, sentiment, stance and emoji prediction. «Leaving whilst
its dark is fun. #not» is irony. Use case: monitoring social media and customer feedback. 150 tweets per
task.

**DocILE** (Rossum with the Czech Technical University, ICDAR 2023) has about 6,700 real business documents,
mostly invoices and orders. The model fills in fifteen fields: vendor and customer, document and order
numbers, dates, payment terms, net, tax, gross and amount due. This is invoice capture without typing.
150 documents.

> **Note:** RAFT and DocILE keep their test labels private, so we used their public labelled splits. For
> CUAD we classify a marked clause; the original task asks models to *find* the clause in a full contract.

{% include section.html %}

## Results

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

We read this table in two ways: against the frontier, and against the open models Apertus competes with
directly.

### Distance to the frontier

The distance is larger than on academic benchmarks. Apertus 1.5 8B reaches 79 percent of GPT-5.6-Luna's
overall score and the 70B model 83 percent, a gap of 14 to 16 points for the 8B model and 11 to 13 points
for the 70B model. These differences are well outside the statistical noise.

How large the gap is depends very much on the task:

{% include figure.html image="images/posts/Apertus15Business-frontier-share.png" caption="Apertus as a share of GPT-5.6-Luna's score, per benchmark. Close on receipts, back-office tasks, social media and invoices; far behind on the long category lists of Banking77 and CUAD." %}

On receipts, back-office tasks, social media and invoices, Apertus is within 10 to 15 percent of the
frontier, and on TweetEval the 70B model is level with GPT-5.6. Most of the gap comes from the two tasks
with long lists of fine-grained categories, contracts with 42 types and banking intents with 77. We
would watch these two in the next release.

In an earlier pilot with fewer, partly synthetic cases, frontier models scored only about 60 percent, and
we concluded that the tasks were "hard for everyone". With a larger and cleaner test set, that no longer
holds (see the section on evaluation below).

### Among open models of similar size

Here Apertus 1.5 8B is at the back of the group, with one clear strength. The chart compares it with the
other open models of similar size, each difference measured on the same cases with a 95 percent
bootstrap interval.

{% include figure.html image="images/posts/Apertus15Business-peers.png" caption="Open models of similar size compared with Apertus 1.5 8B on the same cases. Qwen 3.5 9B leads by 7.4 points; Apertus 1.5 is 2.6 points ahead of its predecessor." %}

All of these differences are significant. Overall, Apertus 1.5 8B comes after Qwen 3.5, Qwen 2.5 and
Llama 3.1 and before its predecessor. Benchmark by benchmark it looks different. Among the five models of
similar size, Apertus 1.5 8B is

- second on documents: on DocILE (invoices) it is level with Qwen 3.5 (62.3 against 62.8), and on SROIE
  (receipts) it is ahead of Qwen 2.5 and Llama 3.1;
- fourth on RAFT, TweetEval and Banking77;
- last on contracts, behind Apertus 1.0 as well.

The 70B model does not change this much. It is 4.3 points behind Qwen 3.5 9B, level with Qwen 2.5 7B (a
model about two years older) and 1.4 points ahead of Llama 3.1 8B.

What counts as a peer matters here. In [our previous post]({% link _posts/2026-07-29-Apertus15Bench.md %})
we separated *open-weight* models such as Qwen and Llama, which publish their weights, from *fully open*
models such as Apertus and OLMo, which also publish data and training pipeline. Every competitor in this
comparison is open-weight; Apertus is the only fully open model, and a run with OLMo 3, its natural peer,
is still missing. Full openness has a cost that open-weight models avoid, for example in which training
data can be used, and the numbers should be read with that in mind.

Size needs a second look as well. Apertus 1.5 is the first Apertus that takes images and audio as input,
and part of its 8.9 billion parameters serve those: a vision tokenizer, an audio tokenizer and the
embeddings for image and audio tokens, about 0.85 billion in all. That leaves about 8 billion for text.
Qwen 3.5 9B reads images but not audio, and its text path has roughly 9 billion parameters. This explains
part of the 7.4-point gap, but not all of it: Llama 3.1 8B, a text-only model of similar size, is also
ahead of Apertus 1.5 8B. Training may matter more. Apertus 1.5 continued pretraining on 4 trillion tokens
of mixed text, images and audio, and a text benchmark only sees one part of what it learned. Our image
test further down shows that, at least for reading receipts, the image side does not make up the gap.

Within the Apertus family, 1.5 is 2.6 points ahead of 1.0 at 8B. The more useful information is where it
gains and where it loses.

{% include section.html dark=true %}

## Apertus 1.5 8B at a Glance

{% include figure.html image="images/posts/Apertus15Business-radar.png" caption="Apertus 1.5 8B (blue) against Qwen 3.5 9B of the same size class (orange) and GPT-5.6-Luna (dashed). The bottom axis shows how often a model answers null when we delete the date or total from a receipt." %}

At the top of the chart are invoices and receipts, where Apertus is level with Qwen 3.5 and not far from
GPT. On the left are the long category lists, contracts and customer requests, where the gap is widest.
The axis at the bottom matters a lot in practice: does the model answer `null` when information is
missing? Apertus 1.5 8B never does.

The following five cases from the data show what this looks like.

{% include section.html %}

## Five Cases from the Data

### 1. Customer requests: clear ones work, look-alikes do not

Requests with one clear reason are routed perfectly. Every test case for intents such as «Please
terminate my account» (`terminate_account`), «How old do I have to be?» (`age_limit`) or «What can I do if
contactless doesn't work?» (`contactless_not_working`) came back correct.

Banking77 also has three intents about identity checks: `verify_my_identity`, `why_verify_identity` and
`unable_to_verify_identity`. Apertus put all 24 of these requests into the first one, so «I'd rather not
verify my identity» and «I can't verify my ID» both became `verify_my_identity`. Qwen 3.5 keeps them apart
much better.

In practice, Apertus 8B works well as a router when the categories are clearly distinct. When several
categories sound alike, merge them or add one example per category to the prompt.

### 2. A purchase order: right numbers, wrong roles

This is a real purchase order from DocILE, sent by a small media company to a radio station, together
with what Apertus 1.5 8B extracted:

{% include figure.html image="images/posts/Apertus15Business-purchase-order.png" caption="A real purchase order from DocILE (OCR text, shortened). Green: read correctly. Red: the letterhead taken as the vendor, and the words Purchase Order returned as the order number." %}

The date and the total are exactly right, which is typical. On invoices the model gets net, tax and due
amounts right in 83 to 94 percent of cases. It took the letterhead for the vendor, though. On a purchase
order the letterhead belongs to the *buyer*, and the vendor is the one listed under «VENDOR». As the order
number it returned the words «Purchase Order» rather than 7601.

Dates and amounts can mostly be trusted. For roles, tell the model what kind of document it is reading,
or check the result with a rule.

### 3. Contracts: one label for everything

> «The Reseller shall be entitled to enter into agreements with its subsidiaries and affiliates to act
> as sub-distributors and/or selling agents of the Products in the Territory.»

CUAD labels this clause `Affiliate License-Licensee`. Apertus 1.5 70B gets it; Apertus 1.5 8B answers
`License Grant`, the most general licence category. That happens a lot: across 294 clauses the 8B model
says `License Grant` 62 times, while only six clauses are actual license grants. Now and then it also
invents types that are not on the list, such as «Severability» or «Limited Liability».

{% include figure.html image="images/posts/Apertus15Business-favourite-labels.png" caption="How often Apertus 1.5 8B gives an answer (blue) against how often that answer is actually correct (grey). When unsure, it falls back on a general category." %}

With 40 or more categories it helps to classify in two steps, first the broad area (licence, liability,
termination), then the exact type. And any label should be checked against the list before it is used.

### 4. Irony: understood, but not in the expected words

> «Just paid $2.59 for gas! #ThanksObama #sarcasm»

The allowed answers are `irony` and `non_irony`. Apertus 1.5 8B answers `sarcasm`. It understood the
tweet, but the word is not on the list, so the answer is scored as wrong. This happened in 9 of 150 irony
cases. Apertus 70B answered `irony` in all nine.

This is the cheapest error to fix: map obvious synonyms in code, or use constrained decoding so the model
can only produce allowed labels.

### 5. The deleted date: Apertus 8B makes one up

To see how models deal with missing information, we took 80 real receipts and deleted either the date or
the total from the text. The correct answer is `null`.

On a bakery receipt from 2017 without its date, Apertus 1.5 8B answered `10/04/2024`, a date that appears
nowhere in the document. On other receipts it gave the time of purchase as the date, or a cash amount as
the total. It did not answer `null` once.

{% include figure.html image="images/posts/Apertus15Business-null.png" caption="Share of 80 receipts with a deleted date or total where the model correctly answered null. The small Apertus models, Llama 3.1 and Qwen 2.5 almost never do." %}

Qwen 3.5 9B, which is about the same size as Apertus 8B, gets this right in 40 percent of cases. So it is
not a hard limit of small models; it can be learned.

In a booking workflow, a confidently wrong value does more damage than an empty field. Check every
extracted value against the source document before it is booked.

{% include section.html dark=true %}

## The 70B Model Compared with the 8B

{% include figure.html image="images/posts/Apertus15Business-8b-vs-70b.png" caption="Apertus 1.5 8B (blue) against Apertus 1.5 70B (orange)." %}

The 70B model has about nine times the parameters, yet on routine document work the two are close. On
invoices, receipts and the RAFT back-office tasks they are about two points apart, and on invoices the 8B
model is slightly ahead.

The 70B model is better at judgement calls. It answers `null` when information is missing (34 percent
against 0), it recognises irony and uses the right word (78 against 61 on the irony task), and it picks
the specific contract category more often than the general one.

For extraction and routing with clear categories we would start with the 8B model. Where the model has to
notice that something is *absent*, or read between the lines, the 70B model or explicit checks are the
safer choice.

{% include section.html %}

## Bonus: The Small Apertus Runs on a Mac

When we reran our tests on 30 September, no Hugging Face Inference Provider served Apertus 1.5 8B any
more, and the standard `transformers` and MLX releases do not support the 1.5 architecture yet. The Swiss
AI fork of `transformers`, which the model card points to, does work:

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

On an Apple M5 Max with 128 GB of memory, in bf16, the model processed about 3,100 benchmark cases with a
median of 1.3 seconds per request and no failed requests. There is no API cost, and no document leaves
the machine. For banks, insurers and public administrations, that is the main reason to consider an open
Swiss model. Image input works on the Mac as well, as the next section shows; audio we have not tested.

{% include section.html %}

## Reading the Scan Instead of the Text

Apertus 1.5 accepts images, so we gave it the receipts a second time: only the scan, without any text.
Same 361 SROIE receipts, same four fields, same scoring. We ran it on the Mac and compared it with Qwen 3.5
9B, which also reads images and runs on the same machine, and with the two frontier models.

As a reference we show the text results from above. That text is the transcript that comes with the
dataset, made as ground truth for an OCR competition. Its characters are clean, but it is flat, a run of
words without layout, and it is incomplete: on a typical receipt almost a third of the address words are
missing. The scan holds more information than the transcript, and a model that reads it well can do better
than with the text.

{% include figure.html image="images/posts/Apertus15Business-scan.png" caption="The same 361 receipts as the dataset's reference transcript (hollow) and as a scanned image (filled). Qwen 3.5 9B and the frontier models read the scan well, Apertus 1.5 8B misreads dates and totals." %}

What we found:

1. From the scan, Apertus gets 61.6 percent of fields right, with dates at 71 percent and totals at 64.
   The model misreads digits: 15/01/2019 becomes 13/01/2019, and a total of 193.00 becomes 198. It never
   answers null for these fields. It returns a plausible wrong number, which is harder to catch than an
   empty field.
2. The other three models do better from the scan than from the transcript, mostly on the address, where
   the layout shows which lines belong together and nothing is missing. Qwen 3.5 9B goes from 29 to 93 percent on the address, GPT-5.6 and Haiku from under
   40 to about 80. Qwen 3.5 reaches 94.4 percent overall and gets 81 percent of receipts fully right;
   Apertus gets 17 percent.
3. Apertus uses about 5,800 input tokens per scan, four times as many as Qwen. On our Mac it needed about
   9 seconds per receipt, Qwen about 3. The two ran on different software (our `transformers` server and
   Ollama), so take the speed difference as a rough guide.

Two cautions. SROIE has been public since 2019, and we cannot rule out that Qwen saw these receipts during
training; a test on fresh receipts would settle that. It would not explain the Apertus result, though.
And Qwen ran 4-bit quantised, Apertus in full precision.

One practical note if you try this yourself: our first run sent four scans to the model at once, memory
grew past 100 GB and the Mac started swapping. One image per batch, with the GPU cache cleared after each
batch, fixed it.

If you build document capture on Apertus 1.5 8B, check every date and total it reads from a scan, for
example against the sum of the line items, or let a model that reads scans well do that part. Whether a
classic OCR step in front of Apertus works better, we have not tested.

{% include section.html dark=true %}

## A Quick Look at Thinking Mode

Apertus 1.5 has a thinking mode, so we tested whether reasoning before answering removes the errors
described above. We used 409 hard cases (look-alike banking intents, contract clauses, irony and missing
data) and ran them with and without thinking.

The first finding was that Apertus decides on its own whether to think. A strict format instruction such
as «answer only with the category» suppressed reasoning even with thinking enabled, and we had to add a
sentence asking the model to think step by step first.

The second was that thinking barely helps Apertus 1.5 70B in this setting. The change ranges from −4 to
+5 points depending on the task (−1 to +5 if we only count answers where the reasoning finished). The
answer time for contract clauses grows from 1.5 to 27 seconds, and in one of nine contract cases the
reasoning did not finish within 3,072 tokens. GPT-5.6 and Claude Haiku 4.5 gain more, mostly on irony
(+5 to +12 points).

{% include figure.html image="images/posts/Apertus15Business-thinking.png" caption="The same hard cases with thinking off (hollow) and on (filled). Thinking helps the frontier models on irony and missing data; for Apertus 1.5 70B it changes little." %}

Thinking still has its uses. Our tasks are short classification and extraction problems, where the answer
comes from recognising a pattern rather than from several steps of inference. On multi-step tasks the
picture is different: in [our academic evaluation]({% link _posts/2026-07-29-Apertus15Bench.md %}),
thinking mode raised Apertus 1.5 8B by 17 points on MATH-500 and by 10.5 points on multilingual
grade-school math (MGSM).

We did not finish the run for Apertus 1.5 8B. With thinking it needed 15 to 50 seconds per case on our Mac,
which made the full test impractical. Whether thinking fixes the small model's errors is still open, and a
good project for anyone with a GPU and an afternoon.

{% include section.html %}

## Ideas for Hack Apertus

We did not build this benchmark for the hackathon, but some of its findings connect to the challenges.
The document results fit 2B and parts of 2A directly; for 1A and 1B they are starting points rather than
finished findings.

**Challenge 1A, Red-Teaming.** The missing-data test is a small red-team probe: delete a field and see
whether the model admits that it does not know. Apertus 1.5 8B filled in a value every time, in one case a
date that appears nowhere on the receipt. In a booking workflow that means wrong entries and possible
financial loss, which counts for severity in the rubric. A second lead is content moderation: in the RAFT
hate-speech task, Apertus 1.5 8B answered «not hate speech» for 13 of the 21 tweets that annotators had
labelled as hate speech. A third lead comes from images: on scanned receipts Apertus 1.5 8B misreads dates
and totals and returns the wrong number rather than null. Our setup also matches what the rubric rewards
under reproducibility: fixed prompts, temperature 0 and a scripted run over many variations. Speech, which
the challenge also accepts, we have not tested.

**Challenge 1B, Swiss Voices.** All our documents were in English. For the *core task intelligence*
dimension, a question set on Swiss legal, administrative or business knowledge, scored as correct,
incorrect or not attempted, would test Apertus where its multilingual training should help. Watch the
«not attempted» share: our results suggest the 8B model rarely declines to answer, and that is worth
measuring on Swiss facts too.

**Challenge 2A, Academia.** The extraction results are relevant to the document challenges, for example
OpenParlData's task of turning parliamentary PDFs into one common structure. Apertus 1.5 8B reads dates and
amounts reliably but confuses roles and fills in missing fields, so a solution should give it the document
type and check every extracted value against the source. Wherever the answer comes from a fixed set, such as
a verdict in a fact check, validate it against the allowed set before using it.

**Challenge 2B, Own Project.** Invoice and receipt capture with Apertus 1.5 8B on a local machine, so that
no document leaves the company, together with the checks from this post. Customer-request routing works as
well if look-alike categories are merged. Three research questions are also open: does thinking mode fix
the 8B model's error types, how much does constrained decoding recover on CUAD and Banking77, and how does
Apertus compare with OLMo 3, the other fully open model?

For the hackathon, both 1.5 models (8B and 70B) are provided through the partners CSCS and Phoeniqs; the
Getting Started Guide explains access under «Resources & Tools». If you also want to run the 8B model on
your own machine, for example to show that documents never leave it, use the Swiss AI fork of
`transformers` as in the bonus section. If you would like our task definitions and scoring code as a
starting point, get in touch.

{% include section.html %}

## What We Learned About Evaluation

Building this benchmark taught us about evaluation as much as about the models. Four things moved our
numbers by more than many of the gaps between models:

1. Scoring. In the pilot we compared extracted amounts as strings, so `133.7` did not match `"133.70"`.
   Comparing amounts as numbers and dates as dates moved Llama 3.1 by twelve points.
2. Data. A support-ticket dataset we had treated as real turned out to be synthetic, and we removed it.
3. Providers. One hosting provider silently dropped tokens from Gemma 2 outputs (`declinedard_payment`
   instead of `declined_card_payment`, `8480` instead of `84.80`). The model was fine and the serving stack
   was not, so Gemma is not in our table.
4. Gold labels. SROIE receipts often lack part of the address in the OCR text, and DocILE does not annotate
   every printed field. We adjusted the scoring to what can actually be read from the text.

Our main advice: look at the raw outputs before trusting a score.

{% include section.html dark=true %}

## Limitations

This is an independent snapshot, not an exhaustive evaluation:

- All prompts are zero-shot. Few-shot prompting or fine-tuning would probably change the results,
  especially for long category lists.
- Instructions were in German and documents in English. That is a common Swiss setup, but it may favour
  multilingual models.
- We tested images only on SROIE receipts, with four models, and audio not at all.
- Qwen 3.5 9B ran 4-bit quantised, all Apertus models in full precision.
- Each model ran once. GPT-5.6 cannot run at temperature 0, so its numbers are a single sample.
- RAFT and DocILE results come from their public labelled splits, not from the hidden test sets.

{% include section.html %}

## Summary

Apertus 1.5 8B is a solid document model that you can run privately on a single Mac; it reads dates and
amounts on invoices and receipts reliably. Its weak spots are specific and can be worked around: check
labels against the allowed list, split long category lists into two steps, and verify every extracted
field against the source. The 70B model mainly adds the ability to notice what is missing. Reading scans
directly is its weakest point so far: it misreads digits, so check every date and total. Qwen 3.5 9B shows that open models of this
size can do better, on text and even more on images, which leaves room for the next Apertus release.

We are presenting these results at the «Hack Apertus» online session on 2 October 2026. If you pick up one of
the open questions during the hackathon, whether format guards, null handling, thinking mode, Swiss
languages, reading scans or audio input, we would like to hear what you find.

**Quicklinks:**
- **Our academic benchmark post**: [LLM Benchmark Evaluation - Apertus 1.5-8B]({% link _posts/2026-07-29-Apertus15Bench.md %})
- **Model Hub**: [Hugging Face: Swiss AI Models](https://huggingface.co/swiss-ai/)
- **Benchmarks**: [Banking77](https://huggingface.co/datasets/legacy-datasets/banking77) ·
  [SROIE](https://huggingface.co/datasets/jsdnrs/ICDAR2019-SROIE) ·
  [CUAD](https://huggingface.co/datasets/theatticusproject/cuad) ·
  [RAFT](https://huggingface.co/datasets/ought/raft) ·
  [TweetEval](https://huggingface.co/datasets/cardiffnlp/tweet_eval) ·
  [DocILE](https://github.com/rossumai/docile)
