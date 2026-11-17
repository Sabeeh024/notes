# LLM Fundamentals: Study Notes

Covers how large language models are built, how they behave, and the common building blocks around them (embeddings, vector databases, RAG, APIs, security).

---

## 1. What is an LLM?

A **Large Language Model** is a neural network trained to **predict the next token** in a sequence of text. Everything else (chat, code, summarising) emerges from doing that one task at huge scale.

An LLM, concretely, is **two files**:

1. **Parameters (weights)**: a big file of numbers.
2. **Run code**: a small program (a few hundred lines of C or Python) that loads the weights and runs the network.

Example: **Llama 2 70B**
- Naming: `model family - version - parameter count` → Llama, v2, 70 billion parameters.
- Stored as float16 (2 bytes per parameter) → 70B × 2 bytes ≈ **140 GB**.

### Parameter data types (precision)

| Type | Size | Notes |
|---|---|---|
| float32 (F32) | 4 bytes | Full precision, mostly used in training |
| float16 / bfloat16 (F16) | 2 bytes | Common for training and inference |
| int8 / int4 (quantized) | 1 / 0.5 bytes | Smaller and faster, slight quality loss; lets big models run on consumer hardware |

Rule of thumb for memory: `parameters × bytes per parameter`.

### Open vs closed weights

| | Open weight | Closed weight |
|---|---|---|
| Examples | Llama, Mistral, Qwen, DeepSeek | GPT, Claude, Gemini |
| Weights downloadable | Yes | No |
| Access | Run locally or host yourself, fine-tune freely | Only through an API or product |
| Control and privacy | High (data stays with you) | Lower (data goes to the provider) |
| Typical peak capability | Slightly behind frontier (gap varies) | Usually frontier |
| Cost model | Hardware and ops | Pay per token |

Note: "open weight" is not the same as "open source". Weights are released, but training data and code often are not.

---

## 2. Training Pipeline Overview

| Stage | Goal | Data | Cost / frequency | Output |
|---|---|---|---|---|
| **1. Pre-training** | Learn language and world knowledge | Huge internet-scale text (quantity) | Very expensive; done rarely (months to a year) | **Base model** |
| **2. Fine-tuning (SFT)** | Learn to behave like an assistant | Small, high-quality Q&A conversations (quality) | Cheap; can be repeated often | **Assistant model** |
| **3. RLHF / preference tuning** (optional) | Align with human preferences | Human comparisons of answers | Moderate | Better-aligned assistant |

---

## 3. Stage 1: Pre-training

**Idea:** compress a large chunk of the internet into the network's parameters. This is **lossy compression**: the model keeps the gist and patterns, not exact copies.

Rough scale for Llama 2 70B (approximate figures from the 2023 era; frontier models today are far larger):
- Training text: ~10 TB (about 10 trillion tokens)
- Hardware: cluster of ~6,000 GPUs
- Time: ~12 days
- Cost: ~$2 million
- Compute is roughly 1,000× what is needed for inference on the same model.

### Terms

- **Training**: adjusting parameters so predictions improve. Expensive, done once (per model).
- **Inference**: using the finished model to generate text. Comparatively cheap, done constantly.
- **Token**: a chunk of text (a word, part of a word, or punctuation). Models read and write tokens, not words. Roughly 1 token ≈ ¾ of an English word.
- **Loss**: a number measuring how wrong the next-token prediction was. Training lowers it.

### How the model generates text

1. Input text → tokens.
2. The network outputs a **probability distribution** over every possible next token.
3. One token is **sampled** from that distribution (temperature controls randomness).
4. That token is appended to the input and fed back in.
5. Repeat. This is the **autoregressive** loop.

Because it samples from learned patterns, a base model "dreams" plausible internet documents (web pages, Wikipedia-style articles, code), depending on the prompt.

### Hallucination

The model produces **plausible text, not verified facts**. When it lacks knowledge, it still generates something that looks right. Mitigations: grounding with retrieved documents (RAG), tool use, asking for citations, verification, and training for calibrated "I don't know".

### Transformer architecture

- All LLMs today use the **Transformer** (Vaswani et al., 2017, "Attention Is All You Need").
- Core ingredient: **self-attention**, which lets each token weigh how relevant every other token is.
- Architecture, maths and training algorithm are fully known.
- What is **not** well understood: how billions of parameters work together to store knowledge and produce behaviour. We can adjust parameters to reduce loss and measure that it works, but the internal "knowledge" is hard to inspect (the field of **interpretability** studies this).

### Quirk: the Reversal Curse

A model trained on "A is B" often fails to infer "B is A". Example: it may answer "Who is Tom Cruise's mother?" but fail "Who is Mary Lee Pfeiffer's son?". Knowledge is stored in a one-directional, oddly shaped way.

### Takeaway

Treat LLMs as **empirical, inscrutable artifacts**, not engineered machines with understandable parts. That means you need good **evaluations** (evals) to know what they can and cannot do.

---

## 4. Stage 2: Fine-tuning (Assistant Model)

- Same training algorithm, but the dataset is swapped.
- The internet text is replaced by **curated question-and-answer conversations**.
- Companies write **labeling instructions** and hire people (or vendors such as Scale AI) to write ideal responses.
- **Quality over quantity**: ~100K high-quality conversations can be enough.
- Result: the model keeps its pre-trained knowledge but adopts the **format and style of a helpful assistant**.

| | Pre-training | Fine-tuning |
|---|---|---|
| Purpose | Knowledge | Alignment / behaviour |
| Data | Internet text, enormous | Curated conversations, small |
| Priority | Quantity | Quality |
| Cost | Millions of dollars | Far cheaper |
| Frequency | Rare | Frequent (e.g. weekly) |
| Product | Base model (autocompletes text) | Assistant model (answers questions) |

### Stage 3: Comparisons and RLHF

- Writing ideal answers is hard, but **comparing** two answers is easier for humans.
- **RLHF** (Reinforcement Learning from Human Feedback): train a **reward model** on human rankings, then optimise the LLM to score well on it.
- Variants: **DPO** (direct preference optimisation), RLAIF (AI feedback), Constitutional AI.
- Risk: models can learn to please graders (sycophancy) rather than be correct.

### The iterative loop used by labs

1. Write labeling instructions.
2. Collect high-quality demonstrations and comparisons.
3. Fine-tune the base model.
4. Run many **evaluations**.
5. Deploy.
6. Monitor, collect misbehaviours, fix them in the data, repeat.

---

## 5. Scaling Laws

LLM performance (next-token loss) is a **smooth, predictable function** of two things:

- **N**: number of parameters
- **D**: amount of training data (tokens)

(Compute is the third factor, roughly `N × D`.)

- Trends have not shown a clear ceiling, so more scale has reliably meant more capability.
- **Chinchilla** (DeepMind, 2022): for a given compute budget, parameters and data should scale together; many early models were under-trained on data.
- Practical caveats: high-quality data is finite, and compute and energy costs are enormous. Newer gains also come from better data, post-training, reasoning and test-time compute.

---

## 6. Where Things Are Heading (Related Concepts)

- **Self-improvement:** AlphaGo first learned from human games (capped at human level), then improved through **self-play** with a clear reward (win/lose). Open question: what is the equivalent of an unambiguous reward in open-ended language tasks?
- **Reasoning models:** trained with reinforcement learning to think step by step before answering.
- **Tool use / agents:** the model calls search, code execution, APIs, browsers.
- **Multimodality:** text plus images, audio and video.
- **Context window:** how many tokens the model can consider at once (its working memory). Models do not remember past chats unless the history is sent again.
- **Mixture of Experts (MoE):** only part of the network is active per token, giving a large model at lower inference cost.

---

## 7. Glossary

### Embedding

A numerical representation of the **meaning** of a piece of text (also images, audio), produced by an **embedding model**.

```
"The user can reset their password from the account settings."
                    ↓
             Embedding model
                    ↓
     [0.021, -0.183, 0.742, 0.091, ..., -0.331]
```

- Typically hundreds to thousands of dimensions (e.g. 384, 768, 1536, 3072).
- **Similar meaning → nearby vectors.** "Reset my password" and "I forgot my login" end up close together even though they share few words.

### Vector

A list of numbers, such as `[0.021, -0.183, 0.742, ...]`. Think of it as a point (or arrow) in a high-dimensional space.

### Similarity measures

| Metric | What it measures | Notes |
|---|---|---|
| **Cosine similarity** | Angle between vectors | Most common for text |
| **Dot product** | Angle and magnitude | Fast; same as cosine if vectors are normalised |
| **Euclidean (L2) distance** | Straight-line distance | Smaller means more similar |

### Vector database

A database built to **store embeddings and quickly find the nearest ones** to a query vector.

```
Document chunk 1: "The user can reset their password..."  → [0.02, -0.18, 0.74, ...]
Document chunk 2: "Orders can be cancelled within 30 days..." → [0.81, 0.12, -0.32, ...]
Document chunk 3: "Two-factor authentication can be enabled..." → [0.05, -0.21, 0.71, ...]
...
Document chunk 100,000 → [...]
```

Search flow:
1. Embed the user's query with the **same** embedding model.
2. Find the closest stored vectors (**nearest-neighbour search**).
3. Return the matching chunks.

- At scale uses **approximate nearest neighbour (ANN)** indexes such as HNSW or IVF for speed.
- Examples: Pinecone, Weaviate, Qdrant, Milvus, Chroma, pgvector (Postgres extension), FAISS (library).
- Chunk size matters: chunks too big dilute meaning, too small lose context. Overlap between chunks helps.

### RAG (Retrieval-Augmented Generation)

Give the LLM relevant external knowledge **at question time** instead of relying on what it memorised.

```
Documents → split into chunks → embed → store in vector DB     (indexing, done once)

User question → embed → search vector DB → top-k chunks         (retrieval)
             → prompt = instructions + chunks + question         (augmentation)
             → LLM answers using that context                    (generation)
```

Why use it:
- Fresh or private data without retraining.
- Fewer hallucinations, and answers can cite sources.
- Cheaper and easier to update than fine-tuning.

Improvements: hybrid search (keyword + vector), re-ranking, query rewriting, metadata filtering. Failure modes: poor chunking, wrong chunks retrieved, model ignoring context.

| Approach | Best for | Weakness |
|---|---|---|
| **Prompting alone** | Simple tasks, general knowledge | Limited to training knowledge and context size |
| **RAG** | Changing or private knowledge, citations | Retrieval quality limits answers |
| **Fine-tuning** | Style, format, specialised behaviour | Poor at adding fresh facts; needs data and retraining |
| **Long context** | Small document sets | Cost and latency grow; accuracy can drop in huge contexts |

### Prompt injection

An attack where **untrusted text contains instructions** that the model follows instead of (or in addition to) the developer's instructions. This is possible because LLMs treat instructions and data as one stream of text.

- **Direct**: the user types "Ignore previous instructions and ...".
- **Indirect**: malicious instructions hidden in a web page, email, PDF or retrieved document that the LLM reads (a big risk for RAG and agents).
- Related: **jailbreaking**, which tries to bypass safety training.

Mitigations (none is perfect):
- Treat all model input from outside as untrusted; separate system instructions from data.
- Least privilege: limit tools, permissions and data an agent can reach.
- Require human confirmation for sensitive actions (sending, paying, deleting).
- Filter and validate outputs; don't render or execute them blindly.
- Monitoring, red-teaming and guardrail models.

### LLM (as a term)

Large neural network trained on text to model language. "Large" refers to billions of parameters and trillions of training tokens.

### LLM API

A hosted interface to a model, usually over HTTP, so you can use it in your own apps without running it.

- Typical request: model name, a list of **messages** (system / user / assistant roles), and settings.
- Typical response: generated text, plus token usage.
- Common settings: `temperature` (randomness), `max_tokens` (output limit), `top_p`, `stop` sequences, streaming, tool/function definitions, structured (JSON) output.
- Pricing: per **input token** and **output token**, so long prompts and long answers cost more.
- The API is **stateless**: you resend the conversation each time.

Minimal shape of a request (conceptual):

```json
{
  "model": "some-model-name",
  "max_tokens": 500,
  "temperature": 0.2,
  "messages": [
    {"role": "system", "content": "You are a helpful support assistant."},
    {"role": "user", "content": "How do I reset my password?"}
  ]
}
```

---

## 8. Other Useful Terms

| Term | Meaning |
|---|---|
| **Prompt** | The input text given to the model |
| **System prompt** | Developer instructions that set behaviour and role |
| **Few-shot prompting** | Including examples in the prompt |
| **Chain-of-thought** | Having the model reason step by step before answering |
| **Temperature** | Sampling randomness; low is focused, high is creative |
| **Quantization** | Reducing numeric precision to shrink and speed up models |
| **LoRA / PEFT** | Cheap fine-tuning that trains small added weights, not the whole model |
| **Distillation** | Training a small model to imitate a larger one |
| **Eval / benchmark** | Tests to measure model capability |
| **Agent** | An LLM in a loop that plans and uses tools to complete tasks |
| **MCP** | Open protocol for connecting LLMs to external tools and data |
| **Guardrails** | Checks that constrain inputs or outputs |

---

## 9. Quick Revision Summary

1. An LLM = weights file + small run program; it **predicts the next token**.
2. **Pre-training** (internet text, very costly, rare) gives knowledge → **base model**.
3. **Fine-tuning** (small, curated conversations, cheap, frequent) gives assistant behaviour → **assistant model**; **RLHF** refines with human preferences.
4. Performance scales **predictably** with parameters and data (**scaling laws**).
5. Models **hallucinate** and are **not fully interpretable**; always evaluate.
6. **Embeddings** turn meaning into vectors; a **vector database** finds similar ones; **RAG** feeds retrieved text to the LLM.
7. **Prompt injection** is the key security risk when models read untrusted content.
8. **LLM APIs** are stateless, token-priced interfaces to hosted models.

---

## 10. Further Reading

- Andrej Karpathy, "Intro to Large Language Models" (video talk) and "Let's build GPT" series
- Vaswani et al., "Attention Is All You Need" (2017)
- Hoffmann et al., "Training Compute-Optimal Large Language Models" (Chinchilla, 2022)
- Ouyang et al., "Training language models to follow instructions with human feedback" (InstructGPT, 2022)
- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020)
- Berglund et al., "The Reversal Curse" (2023)
- OWASP Top 10 for LLM Applications (prompt injection and related risks)
