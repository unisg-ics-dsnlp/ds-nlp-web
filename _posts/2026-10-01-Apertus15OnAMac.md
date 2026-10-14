---
title: Running Apertus 1.5 8B on a Mac
author:
  - Siegfried Handschuh
date: 2026-10-01 12:00:00 +0200
tags:
  - large language models
  - local deployment
  - apple silicon
  - multimodal
  - artificial intelligence
  - switzerland
---

*Update, 14 October 2026: Swiss AI has published the
[Apertus 1.5 technical report](https://arxiv.org/abs/2610.15542). We added what it says about image and
audio input, batching and quantisation (see [Quantisation](#quantisation)).*

## Why Run It Locally

For banks, insurers and public administrations, the main reason to consider an open Swiss model is that
no document has to leave the building. Apertus 1.5 8B is small enough to make that practical: it runs on a
single Mac, with no API cost and no data sent anywhere.

It is also the only way to use the 8B model at the moment. When we ran our tests on 30 September, no
Hugging Face Inference Provider served Apertus 1.5 8B any more, and the standard `transformers` and MLX
releases did not support the 1.5 architecture yet.

{% include section.html %}

## Setup

The Swiss AI fork of `transformers`, which the model card points to, does work on Apple Silicon:

```bash
pip install "transformers[torch,vision,audio] @ git+https://github.com/swiss-ai/transformers.git@3797303dda74844e3d1f8977ff5518bb91f818b4"
```

The model is gated: accept the conditions on its Hugging Face page and use a token with access to gated
repositories.

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

For a benchmark you will want an API rather than a script. We wrapped the same code in a small
OpenAI-compatible server, so that the benchmark talks to the local model exactly as it talks to hosted
ones.

{% include section.html %}

## How Fast It Is

On an Apple M5 Max with 128 GB of memory, in bf16:

1. **Text.** About 3,100 benchmark cases with a median of 1.3 seconds per request and no failed requests.
2. **Images.** About 9 seconds per scanned receipt. Apertus turns a scan into about 5,800 input tokens,
   four times as many as Qwen 3.5 9B, which needed about 3 seconds on the same machine through Ollama.
   The technical report explains the number: Apertus has no separate image encoder. A tokenizer turns
   every block of 16 × 16 pixels into one token, and the processor scales a scan to at most about
   2 megapixels, which allows up to about 7,600 tokens.
3. **Thinking mode.** 15 to 50 seconds per case, which makes it impractical for large test runs.

The 70B model does not fit: in full precision it needs about 145 GB.

{% include section.html %}

## Two Things to Watch

**Memory with images.** Our first image run sent four scans to the model at once. Memory grew past 100 GB
and the Mac started swapping. Sending one image per batch and clearing the GPU cache after each batch
(`torch.mps.empty_cache()`) fixed it. According to the technical report there is a second reason to send
one image or recording at a time: in an early vLLM integration, encoding several of them as one padded
batch changed their tokens. That integration has been fixed, but one item per batch avoids the problem
in any serving setup.

**Audio format.** Apertus expects mono audio at 24 kHz. Phone recordings at 8 kHz have to be resampled
first, for example with `librosa.load(path, sr=24000, mono=True)`, which probably costs some accuracy.
You do not need to adjust the volume: the processor in the Swiss AI fork scales every clip to the same
peak level, as the report prescribes. The audio tokenizer turns a second of sound into 40 tokens from a
vocabulary of 4,096, and most of the training audio is parliamentary, broadcast and read speech, so
phone calls are unfamiliar ground for the model.

{% include section.html %}

## Quantisation

*Added 14 October 2026.*

We ran Apertus 1.5 8B in full precision (bf16). The technical report includes first quantisation results
from OnPrem.ai, one of the partners that tested the models before release, on Nvidia hardware:

1. 8-bit (FP8) weights and activations are close to lossless.
2. 4-bit (NVFP4) in the attention layers of the 8B model produces invalid output (NaN logits). The
   report attributes this to the model's activation function (xIELU), which amplifies quantisation
   noise; the activations also grow about a hundredfold towards the deeper layers.
3. With attention at 8 bit and only the MLP layers at 4 bit, the 70B model fits into 49 GiB and runs on a
   single 96 GB GPU.

For Apple Silicon this is a lead, not a recipe. NVFP4 is an Nvidia format, and the 4-bit formats of MLX
or llama.cpp quantise differently; we have not tested them, and when we ran our tests neither supported
Apertus 1.5. If a quantised Apertus build for the Mac appears, we would keep attention at 8 bit and
compare its output with bf16 before trusting it. At 8 bit, the 70B model would need roughly 70 GB and
would fit on a 128 GB Mac.

This also matters for comparisons. In our benchmark Qwen 3.5 9B ran 4-bit quantised through Ollama.
That works for Qwen; it does not mean that 4-bit works for Apertus, whose architecture is different.

{% include section.html %}

## What It Can Do

How well the 8B model handles customer requests, receipts, invoices, contracts, scans and recorded calls,
and how it compares with the 70B model and other models, is the subject of
[our business benchmark post]({% post_url 2026-10-01-Apertus1-5-BusinessBenchmarks %}). Every 8B result
there was produced with the setup above.
