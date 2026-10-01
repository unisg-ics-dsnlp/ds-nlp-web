---
title: "Getting Started with Apertus 1.5: Architecture, API, and Local Deployment"
author:
  - Götz-Henrik Wiegand
date: 2026-10-01
tags:
  - transformers
  - machine-learning
  - large language models
  - artificial intelligence
  - switzerland
---

You have an idea for Hack Apertus. Now you need a model endpoint, a working prompt, and a way to
connect the two. This post introduces Apertus 1.5 and takes you through the practical steps: calling
a hosted API, running the model on your own hardware, and enabling its thinking mode.

It accompanies our hackathon presentation on benchmarking Apertus. For the evaluation results,
see [our Apertus 1.5 benchmark post]({% post_url 2026-07-29-Apertus15Bench %}). Here, we focus on
getting the model into your application.

**Version note:** The deployment instructions and provider details below were checked on
1 October 2026. Apertus 1.5 integrations are still being upstreamed, so use the linked official
instructions when updating your environment.

{% include section.html %}

## What Is Apertus?

Apertus is a family of open models developed by the Swiss AI Initiative, a collaboration between
ETH Zurich, EPFL, and the Swiss National Supercomputing Centre (CSCS). Its defining ambition is
transparency across the model's development: weights, data sources, training recipes, and the
principles used to align its behavior. The original models were trained on the Alps supercomputer.
The [official overview](https://apertus-ai.org/docs/overview/) explains the project and its goals.

For developers, that openness makes it possible to inspect, adapt, and host the model yourself.
Multilingual representation is another central design choice. The original training mixture
contained approximately 40% non-English data, including FineWeb-2's coverage of 1,811 languages.
That describes the breadth of the data, rather than a guarantee of equal performance in every
language. See the [Apertus 1.0 technical report](https://arxiv.org/html/2509.14233v1) for the
underlying architecture and training recipe.

### What Changes in 1.5?

Apertus 1.5 continues pretraining the original models and adds a new post-training pipeline. It
comes in two sizes:

| Model | Hugging Face checkpoint | Starting point |
|---|---|---|
| 8B | [`swiss-ai/Apertus-v1.5-8B`](https://huggingface.co/swiss-ai/Apertus-v1.5-8B) | Local experiments and applications with smaller compute budgets |
| 70B | [`swiss-ai/Apertus-v1.5-70B`](https://huggingface.co/swiss-ai/Apertus-v1.5-70B) | Hosted APIs or server hardware with more memory |

The main additions are **image and audio inputs**, **optional thinking mode**, **improved instruction
following and tool use**, and a **262,144-token context window**, up from 65,536 tokens. The output
is text; Apertus 1.5 does not generate images or audio. Audio understanding remains experimental.
The [1.5 release announcement](https://apertus-ai.org/articles/2026-07-apertus-1-5/) describes these
changes and the continued-pretraining stage: an additional 4T tokens for 8B and 2T for 70B, bringing
the reported totals to 19T and 17T respectively.

Useful official entry points are the [Apertus website](https://apertus-ai.org/),
[documentation](https://apertus-ai.org/docs/), and [Swiss AI model hub](https://huggingface.co/swiss-ai).
For available hosted services, consult the
[inference ecosystem overview](https://apertus-ai.org/articles/2026-09-apertus-1-5-ga/).

{% include section.html %}

## How Does the Architecture Differ from Llama and Qwen?

Apertus uses a **dense decoder-only Transformer**. Like familiar Llama and Qwen models, it generates
tokens autoregressively and uses grouped-query attention (GQA), rotary positional embeddings
(RoPE), and RMSNorm. It is not a mixture-of-experts model: the dense language backbone is active
for each token.

The following comparison expands the table from our presentation. It includes both architecture
and training choices; an optimizer or alignment method affects how the model is trained, but is
not a component you run during inference. Llama and Qwen also span several generations, so the
middle column is a broad reference rather than a specification for every release.

| Component | Typical Llama/Qwen-style choice | Apertus choice |
|---|---|---|
| Feed-forward activation | SwiGLU, a gated activation | **xIELU**, a non-gated activation extending Squared ReLU to negative inputs, with learned coefficients |
| Pretraining optimizer | AdamW is a common baseline | **AdEMAMix**, combining gradient averages at different timescales |
| Attention | GQA and RoPE; some newer models also use QK normalization | **GQA + scaled RoPE + QK-Norm**, normalizing queries and keys to stabilize attention logits |
| Alignment | Methods such as RLHF or DPO, depending on the release | **QRPO** (Quantile Reward Policy Optimization) in the published Apertus 1.0 recipe; 1.5 adds second-generation post-training |
| Text vocabulary | Model-dependent, often roughly 32k–150k tokens | **131,072 text tokens**, based on a multilingual byte-level BPE tokenizer |

The architecture and QRPO entries are documented in the
[original technical report](https://arxiv.org/html/2509.14233v1). The QRPO row describes that published
recipe; it should not be read as a complete account of the newer 1.5 alignment pipeline.

The change you will encounter most directly when deploying 1.5 is its multimodal input processing.
Images and audio are converted into discrete tokens and fed into the language backbone alongside
text. The [Transformers integration proposal](https://github.com/huggingface/transformers/pull/47662)
identifies Emu3.5 for image tokenization and WavTokenizer for audio. This is why a runtime that
supports text-only Apertus 1.0 does not automatically support the complete 1.5 checkpoint.

The **131,072-token text vocabulary** should also be distinguished from the **266,752-token total
vocabulary** in the multimodal checkpoint. The latter includes input tokens for other modalities;
generation is restricted to text tokens. These details are recorded in the
[1.5 model card](https://huggingface.co/swiss-ai/Apertus-v1.5-8B#input-notes).

{% include section.html %}

## Use Apertus Through a Hosted API

A hosted endpoint is a convenient first step for a hackathon: you can build your application
before setting up a GPU. The presentation uses [Public AI](https://publicai.co/) as an example.
Its gateway exposes an OpenAI-compatible API, so existing chat-completion clients need only a
different base URL, key, and model identifier.

Create an account and API key through the [Public AI developer portal](https://platform.publicai.co/docs).
Check the portal for current credits and rate limits. Its documentation requires both a bearer
token and a `User-Agent` header.

Set your key in the shell where you will run the examples:

```bash
export PUBLICAI_API_KEY="your-api-key"
```

First, list the available models:

```bash
curl https://api.publicai.co/v1/models \
  -H "Authorization: Bearer $PUBLICAI_API_KEY" \
  -H "User-Agent: HackApertusDemo/1.0"
```

Provider model names can differ from Hugging Face repository names. Public AI's documented example
uses the lowercase identifier `swiss-ai/apertus-v1.5-8b`. Use the catalog response to select the
exact identifier for another size or configuration.

```bash
curl https://api.publicai.co/v1/chat/completions \
  -H "Authorization: Bearer $PUBLICAI_API_KEY" \
  -H "User-Agent: HackApertusDemo/1.0" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "swiss-ai/apertus-v1.5-8b",
    "messages": [
      {"role": "system", "content": "Give concise, practical answers."},
      {"role": "user", "content": "Suggest a multilingual hackathon project using Apertus."}
    ],
    "max_tokens": 512
  }'
```

For a Python application, install the client with `pip install openai` and save this as `chat.py`:

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://api.publicai.co/v1",
    api_key=os.environ["PUBLICAI_API_KEY"],
    default_headers={"User-Agent": "HackApertusDemo/1.0"},
)

result = client.chat.completions.create(
    model="swiss-ai/apertus-v1.5-8b",
    messages=[
        {"role": "system", "content": "Give concise, practical answers."},
        {"role": "user", "content": "Explain retrieval-augmented generation in German."},
    ],
    max_tokens=512,
)
print(result.choices[0].message.content)
```

Run it with `python chat.py`. Your application sends structured messages; the server applies the
model's chat template. Support for images, audio, tools, and thinking controls depends on the
provider's deployment, even when the underlying model supports them. Check those capabilities
before choosing an endpoint for a multimodal or agent project.

{% include section.html %}

## Deploy Apertus Locally

There are two useful development paths: **vLLM** gives your application a local API server;
**Transformers** lets you work directly with the model and processor in Python. For a laptop text
demo, there is also a community GGUF option below.

### Plan for Memory, Not Just Parameter Count

As a rough lower bound, two bytes per parameter means approximately **16 GB for 8B** or **140 GB
for 70B** in BF16. These are arithmetic estimates for weights, not total memory requirements.
The full deployment also needs memory for multimodal components, runtime buffers, and the
key-value (KV) cache used to retain previous tokens' attention state.

Longer contexts and concurrent requests increase KV-cache demand. A model's advertised 262k
context does not mean you should reserve that capacity for your first demo. Start with a shorter
server limit, such as 8,192 tokens, and increase it when your workload and hardware require it.
Quantization can reduce weight memory, but support must match the checkpoint and runtime.

### Option 1: Serve the Model with vLLM

As of the version date above, the official instructions recommend the Swiss AI integration and
prebuilt container for Apertus 1.5. Support for a parser or the original Apertus architecture in
upstream vLLM is not sufficient evidence of full multimodal 1.5 support. Follow the
[official vLLM guide](https://apertus-ai.org/docs/guides/vllm/) for the supported setup.

The example below targets a **Linux NVIDIA GPU machine** with a compatible driver, Docker, and
the NVIDIA Container Toolkit configured. The current
[image build](https://github.com/swiss-ai/model-launch/blob/main/images/vllm_apertus_1.5_release/Dockerfile)
uses CUDA 13. Check driver compatibility with the image you pull. The ARM64 image is for ARM
machines with compatible NVIDIA hardware; Docker on an Apple Silicon laptop does not provide
this CUDA setup.

The Hugging Face repositories currently require accepting the access conditions on the model
page. Do that first, then supply a Hugging Face read token with access to the checkpoint:

```bash
export HF_TOKEN="your-hugging-face-read-token"
docker pull ghcr.io/swiss-ai/vllm_apertus_1.5_release:latest-amd64
```

Start an 8B server:

```bash
docker run --rm --gpus all --ipc=host \
  -p 127.0.0.1:8000:8000 \
  -e HF_TOKEN \
  -v "$HOME/.cache/huggingface:/root/.cache/huggingface" \
  ghcr.io/swiss-ai/vllm_apertus_1.5_release:latest-amd64 \
  vllm serve swiss-ai/Apertus-v1.5-8B \
    --host 0.0.0.0 \
    --port 8000 \
    --chat-template-content-format string \
    --gpu-memory-utilization 0.9 \
    --max-model-len 8192
```

The cache mount keeps downloaded weights between runs. Port 8000 is exposed on your machine's
loopback interface. The memory utilization setting lets vLLM budget 90% of GPU memory; this is
a starting configuration for a dedicated GPU, not a guarantee that the model fits.

Once the server has loaded the checkpoint, verify the endpoint:

```bash
curl http://localhost:8000/v1/models
```

You can now use the same Python client pattern as above:

```python
client = OpenAI(base_url="http://localhost:8000/v1", api_key="local")
# In chat.completions.create, use model="swiss-ai/Apertus-v1.5-8B".
```

The `local` key is a client placeholder; the command above does not enable server authentication.
For the official 70B example, switch the checkpoint to `swiss-ai/Apertus-v1.5-70B` and add
`--tensor-parallel-size 4` to split it across four GPUs. The required GPU memory still depends on
precision, context, and concurrency. For repeatable experiments, record the container digest and
model revision rather than relying indefinitely on a moving `latest` tag.

If startup fails with an unknown `apertus1p5` architecture, check that you are running the Swiss AI
image. If vLLM cannot allocate a KV cache, reduce `--max-model-len`; if the weights themselves do
not fit, a shorter context alone will not solve the problem.

### Option 2: Run Inference Directly with Transformers

Use a separate Python environment so that the Apertus integration does not replace a library
version needed by another project. The official model card pins this Transformers revision:

```bash
python3 -m venv .venv-apertus
source .venv-apertus/bin/activate
pip install accelerate
pip install "transformers[torch,vision,audio] @ git+https://github.com/swiss-ai/transformers.git@3797303dda74844e3d1f8977ff5518bb91f818b4"
```

With `HF_TOKEN` set as above, save the following as `local_apertus.py`. It follows the
[official Transformers guide](https://apertus-ai.org/docs/guides/transformers/) and uses the
processor to prepare the model input:

```python
import torch
from transformers import AutoModelForMultimodalLM, AutoProcessor

MODEL_ID = "swiss-ai/Apertus-v1.5-8B"
processor = AutoProcessor.from_pretrained(MODEL_ID)
model = AutoModelForMultimodalLM.from_pretrained(
    MODEL_ID, dtype="auto", device_map="auto"
).eval()

def answer(messages, thinking=False, budget=512):
    batch = processor.apply_chat_template(
        messages,
        enable_thinking=thinking,
        add_generation_prompt=True,
        tokenize=True,
        return_dict=True,
        return_tensors="pt",
    ).to(model.device)
    with torch.inference_mode():
        generated = model.generate(**batch, max_new_tokens=budget)
    new_tokens = generated[0, batch["input_ids"].shape[-1]:]
    return processor.decode(new_tokens, skip_special_tokens=True)

messages = [{"role": "user", "content": "Explain xIELU in one sentence."}]
print(answer(messages))
```

Run `python local_apertus.py`. The slicing step removes the input prompt from the decoded output.
`device_map="auto"` selects device placement; it does not guarantee that the model fits entirely
on your GPU, and offloading can be slow.

To try an image, replace `messages` with:

```python
messages = [{
    "role": "user",
    "content": [
        {"type": "image", "path": "diagram.png"},
        {"type": "text", "text": "Explain the main components in this diagram."},
    ],
}]
```

Supply a real local `diagram.png`. For an audio experiment, use an audio block such as
`{"type": "audio", "path": "recording.wav"}` and a text instruction to transcribe or summarize it.
The processor handles media loading. Avoid calling `.half()` on the entire loaded model: the
image and audio tokenizers retain FP32 precision deliberately. The
[model card's media examples](https://huggingface.co/swiss-ai/Apertus-v1.5-8B#image)
document the accepted inputs and precision requirements.

### A Laptop Option: Text-Only GGUF

The [official Ollama guide](https://apertus-ai.org/docs/guides/ollama/) lists community conversions,
including Andreas Martin's Apertus 1.5 8B text-only build. Its
[Q8_0 model card](https://huggingface.co/andreasmartin/apertus-v1.5-8b-text-Q8_0-GGUF)
provides this llama.cpp route for macOS and Linux:

```bash
brew install llama.cpp
llama-server \
  --hf-repo andreasmartin/apertus-v1.5-8b-text-Q8_0-GGUF \
  --hf-file apertus-v1.5-8b-text-q8_0.gguf \
  -c 2048
```

The model file is approximately 8.57 GB, with additional memory needed at runtime. This is a
community text-only conversion: use it for a text demo, and use the Swiss AI stack above when
your hack needs the original checkpoint's image or audio processing. Check the conversion's
chat template and reasoning support before depending on thinking mode.

{% include section.html %}

## Thinking Mode and the Chat Template

Thinking mode is selected through the **chat template**. The template serializes your messages
and configuration into the token sequence the model was trained to interpret. Asking it to
"think carefully" in an ordinary user message is not the same configuration switch.

### From Alpaca Instructions to Structured Conversations

The familiar [Stanford Alpaca format](https://github.com/tatsu-lab/stanford_alpaca#data-release)
uses readable headings for an instruction, optional input, and response. A shortened example is:

```text
### Instruction:
Summarize this document.

### Input:
[document text]

### Response:
```

Apertus instead uses special tokens to delimit roles and carries deliberation and tool
configuration in a developer block. For a simple text conversation, the structure looks like
this, with a leading beginning-of-sequence token omitted and spacing added for readability:

```text
<|system_start|>
Give concise, practical answers.
<|system_end|>
<|developer_start|>
Deliberation: disabled
Tool Capabilities: disabled
<|developer_end|>
<|user_start|>
Summarize this document: [document text]
<|user_end|>
<|assistant_start|>
```

The assistant's reply ends with `<|assistant_end|>`. Unlike Alpaca's single instruction/input
pair, these role boundaries can represent a conversation with multiple turns. The developer block
also gives model behavior an explicit configuration location. Let `apply_chat_template` or your
API server produce this sequence; do not wrap it again in Alpaca headings.

With thinking enabled, the developer line becomes `Deliberation: enabled`. The assistant can then
generate a reasoning span before its final answer:

```text
<|assistant_start|>
<|inner_prefix|>
[generated reasoning]
<|inner_suffix|>
[final answer]
<|assistant_end|>
```

These delimiters are Apertus-specific; a parser expecting Qwen-style `<think>` tags will not match
them. The [canonical chat template](https://github.com/swiss-ai/apertus-omni-tokenizer/blob/main/chat_templates/Apertus_1p5/chat_template.jinja)
defines the role blocks, reasoning markers, media placeholders, and tool serialization.

### Enable Thinking in Python or vLLM

Using the Transformers helper from above:

```python
messages = [{
    "role": "user",
    "content": "Find all positive integers n for which n squared plus n equals 210.",
}]
print(answer(messages, thinking=True, budget=2048))
```

This passes `enable_thinking=True` to the template. Give generation enough tokens for both
deliberation and the final answer. If it exhausts the budget during reasoning, your application
may receive no completed answer. The model card notes that the reasoning markers remain in
decoded text even with `skip_special_tokens=True`, so handle that span explicitly when displaying
the result.

For vLLM, stop the previous container and repeat its launch command with these additional flags
after `--max-model-len 8192`:

```bash
--served-model-name swiss-ai/Apertus-v1.5-8B-thinking \
--reasoning-parser apertus \
--default-chat-template-kwargs.enable_thinking true
```

Use `swiss-ai/Apertus-v1.5-8B-thinking` as the model name in subsequent API requests and increase
`max_tokens`, for example to 2048. The alias identifies this server configuration; it does not
download a different set of weights. The template flag enables deliberation, while the reasoning
parser separates the reasoning span from the answer. See the
[official thinking-mode setup](https://apertus-ai.org/docs/guides/vllm/#thinking-mode).

### What About Tools?

For a tool-enabled, non-thinking vLLM server, add these flags to the original launch command:

```bash
--enable-auto-tool-choice --tool-call-parser apertus
```

Your application supplies function definitions in the request's `tools` field. The template
serializes their schemas into the developer block. The model proposes a call; your application
executes the function and sends its result back as a tool message before requesting the next
response. Serving a model with a tool parser does not execute application functions by itself.

**The current official setup does not support tool calling in thinking mode.** Choose the
non-thinking configuration for a tool-based agent, or use separate requests for reasoning and
tool execution. The [model card](https://huggingface.co/swiss-ai/Apertus-v1.5-8B#vllm)
explicitly omits tool flags from its thinking examples.

{% include section.html %}

## A Starting Point for Your Hack

Start with a hosted API if you want to concentrate on your application, or the 8B vLLM server if
you need local control. Use the model's own chat template throughout. Enable thinking for tasks
that benefit from a longer reasoning process, and budget for the additional generation time.
For a tool-based workflow, begin with the supported non-thinking configuration.

Before building a larger interface, try a small set of real examples from your intended task:
your target languages, your documents or images, and the output format your code expects. That
will tell you more about the fit for your hack than parameter count alone.

### What Matters in Each Track

**Track 1A, Red-Teaming:** Read the chat template section carefully. System prompt, developer block
and thinking mode all influence how the model responds, and many interesting behaviors only appear
under specific configurations. A finding is only useful to the Apertus team if others can reproduce
it, so record the exact model, template settings and sampling parameters for every result, and
compare 8B and 70B where you can.

**Track 1B, Swiss Voices:** If your project works with spoken Swiss German or other audio, keep in
mind that audio understanding is still experimental in 1.5. Test a handful of your own recordings
on day one, before you commit to an audio-based design, and have a text-based fallback ready. For
text-based dialect work, check early how the model handles your target dialects in both input
and output.

**Track 2, Adoption (Academia and Own Project):** Decide early whether your application needs
thinking mode or tool calling, because the current setup does not combine the two. Agents that
call tools should use the non-thinking configuration. If your task benefits from longer reasoning,
budget enough output tokens and handle the reasoning span explicitly in your interface.

For our measured comparison of Apertus 1.5, Apertus 1.0, and thinking mode, continue with
[**LLM Benchmark Evaluation - Apertus 1.5-8B**]({% post_url 2026-07-29-Apertus15Bench %}).
