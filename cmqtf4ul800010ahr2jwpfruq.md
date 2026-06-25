---
title: "How LLMs Actually Work (and How to Build AI with Them)"
seoDescription: "Learn how LLMs work and build AI apps with JavaScript, APIs, prompting, tool calling, and AI agents."
datePublished: 2026-06-25T11:28:10.496Z
cuid: cmqtf4ul800010ahr2jwpfruq
slug: how-llms-actually-work-and-how-to-build-ai-with-them
cover: https://cdn.hashnode.com/uploads/covers/69b5b16d1f4d4527ed9008ad/e9884ce0-89b2-43ad-921d-ecd77c9e6fc1.jpg
tags: artificial-intelligence, backend-development, generative-ai, prompt-engineering, large-language-models-llms, javascript-ai-engineering

---

> **Everything you need to go from "what even is GenAI?" to shipping real AI-powered apps with JavaScript — no PhD required.**

---

## Who Is This For?

You're a developer. You build things. You've been hearing "AI" everywhere and you want to actually *understand* it — not just paste ChatGPT prompts. You want to know how it works under the hood, how to talk to it from code, and what the difference is between all these terms people keep throwing around: GenAI, LLMs, Agentic AI, Agents, Reasoning models...

This is a two-part guide:

- **Part 1** — The Foundations: What GenAI is, how it works, the history, the terminology, and how to write great prompts.
- **Part 2** — The Code: Hands-on with JavaScript, real API calls, function/tool calling, and knowing which model to pick for what job.

Let's go.

---

# PART 1: The Foundations

---

## 1. What is Generative AI?

Generative AI is a class of artificial intelligence systems that can *create new content* — text, images, audio, code, video — based on patterns they learned from training data.

The word "generative" is the key. Traditional AI was mostly about *classification* — spam or not spam, cat or dog, fraud or legitimate. Generative AI does something fundamentally different: given an input (called a **prompt**), it generates a brand-new output that didn't exist before.

Some concrete examples:
- You give it a topic → it writes a blog post
- You give it a bug description → it writes the fix
- You give it a sketch → it generates a full image
- You give it text → it generates lifelike speech audio

At the heart of most modern GenAI — especially for text — is a type of model called a **Large Language Model (LLM)**. We'll dig deep into those shortly.

---

## 2. A Brief History: How We Got Here

Understanding where we are requires a quick look at where we came from. Generative AI didn't appear out of nowhere in 2022 — it's the result of decades of research, several dead ends, and a few genuinely paradigm-shifting breakthroughs.

### 2.1 Text Generation Before LLMs

Before the transformer era, generating text was painful and rigid. Here's how it evolved:

**Rule-Based Systems (1950s–1980s)**
Early systems were explicitly programmed with grammar rules. ELIZA (1966) — the first chatbot — worked by pattern matching: it detected keywords and filled slots in pre-written templates. Ask it about your mother, it'd respond "Tell me more about your family." No understanding, just pattern substitution.

**N-gram Language Models (1980s–2000s)**
An N-gram model predicts the next word based on the N-1 words before it. For example, a trigram model that sees "the quick brown..." would look up what words typically follow that sequence in its training corpus and pick the most likely one. Simple, fast, but limited: it couldn't capture long-range dependencies. If your sentence is 50 words long, the N-gram model has forgotten what it was about by word 20.

**Recurrent Neural Networks & LSTMs (2010s)**
RNNs processed text sequentially — word by word — maintaining a hidden "memory" state. LSTMs (Long Short-Term Memory networks) were a major improvement, capable of retaining information over longer sequences. But they still processed tokens one at a time, which made them slow to train and limited in how far back they could "remember."

The key insight nobody had solved yet was: **how do you let every word directly attend to every other word in a sequence, simultaneously, and efficiently?**

### 2.2 The Transformer Revolution (2017)

In 2017, a team of Google researchers published a paper titled ***"Attention Is All You Need"*** — one of the most influential papers in the history of AI. They proposed a new architecture called the **Transformer**, which replaced recurrence entirely with a mechanism called **self-attention**.

The core idea: instead of reading a sentence left-to-right like an RNN, a Transformer can look at *all words at once* and dynamically decide which ones are relevant to understanding each other word. Processing "The bank by the river was steep" vs. "I went to the bank to deposit money" — the word "bank" means something different in each. Self-attention lets the model figure that out by looking at the surrounding context all at once.

This was a game changer for three reasons:
1. **Parallelizable** — unlike RNNs, all positions can be processed simultaneously, making training on massive datasets feasible.
2. **Long-range dependencies** — every token can attend to every other token directly.
3. **Scalable** — more compute + more data = better results, almost without ceiling.

### 2.3 The GPT Era & the Rise of Pre-training (2018–2022)

OpenAI took the Transformer decoder and turned it into GPT (Generative Pre-trained Transformer) in 2018. The idea: pre-train a giant model on massive amounts of internet text to predict the next word, then fine-tune it for specific tasks.

GPT-2 (2019) could write coherent paragraphs. GPT-3 (2020) with 175 billion parameters could write essays, translate languages, answer questions, and write working code — all from a single pre-trained model with no fine-tuning. Researchers were stunned.

Meanwhile, Google released BERT (2018) and later T5, showing that bidirectional transformers could achieve state-of-the-art on nearly every NLP benchmark.

### 2.4 The ChatGPT Moment (November 2022)

OpenAI released ChatGPT in November 2022. It reached **100 million users in two months** — faster than TikTok (9 months) or Instagram (2+ years). This wasn't a breakthrough in the underlying technology so much as a breakthrough in *accessibility* — a clean chat interface that anyone could use, powered by GPT-3.5 fine-tuned with Reinforcement Learning from Human Feedback (RLHF) to be helpful and safe.

The world changed that month. Developers who had been quietly using the OpenAI API suddenly had massive corporate attention, VC funding, and a billion users to build for.

### 2.5 The Modern Era: Multimodal + Reasoning (2023–2025)

**2023:** GPT-4 added multimodal inputs (images + text). Anthropic released Claude. Google's Bard launched. Meta open-sourced Llama, democratizing LLM access for anyone with a GPU.

**2024:** Reasoning models emerged. OpenAI's o1 introduced a new paradigm — models that "think" before they answer, spending more compute on step-by-step internal reasoning. This dramatically improved performance on math, code, and logic tasks.

**2025:** Reasoning became standard. Long context windows exploded (1M+ tokens became normal). Models became multimodal by default — text, image, audio, video, all in one. Open-source models like Llama 4 and Qwen closed the gap with proprietary models significantly.

Here's a condensed timeline:

| Year | Milestone |
|------|-----------|
| 1966 | ELIZA — first chatbot, rule-based |
| 1986 | Backpropagation enables neural networks |
| 1997 | IBM Deep Blue beats Kasparov at chess |
| 2012 | AlexNet proves deep learning on ImageNet |
| 2017 | "Attention Is All You Need" — Transformer paper |
| 2018 | GPT-1, BERT released |
| 2019 | GPT-2 — first model OpenAI was scared to release |
| 2020 | GPT-3 (175B params) — few-shot learning shines |
| 2021 | GitHub Copilot, DALL-E, Codex |
| 2022 | ChatGPT launches, 100M users in 2 months |
| 2023 | GPT-4 multimodal, Claude, Llama 1 open-sourced |
| 2024 | o1 reasoning model, Llama 3, Gemini 1.5 |
| 2025 | Reasoning standard, 1M+ context windows, Llama 4, Qwen3 |

---

## 3. LLMs: How They Actually Work

An LLM is a neural network trained on massive amounts of text to predict the next token in a sequence. Let's unpack everything in that sentence.

### 3.1 Tokens: The Atomic Unit

Models don't read text the way humans do. They read **tokens**.

A token is roughly **3–4 characters or about ¾ of a word** on average. The tokenizer splits your text into these subword units before the model ever sees it. Some examples:

```
"Hello, world!"  →  ["Hello", ",", " world", "!"]  = 4 tokens
"internationalization"  →  ["intern", "ation", "al", "ization"]  = 4 tokens
"Hello" (in some tokenizers)  →  ["Hello"] = 1 token
"GPT-4" →  ["G", "PT", "-", "4"] = 4 tokens
```

Why does this matter to you as a developer?
- **API cost** is measured in tokens (input + output). More tokens = more cost.
- **Context limits** are measured in tokens, not words or characters.
- Models can struggle with character-level tasks ("how many 'r's in 'strawberry'?") because they never see individual characters — only subword tokens.
- Unusual words, code, or non-English text often tokenize less efficiently (more tokens per word).

> 💡 **Rule of thumb:** 1,000 tokens ≈ 750 words. A standard A4 page ≈ ~500–700 tokens.

### 3.2 Context & Context Window

The **context** is everything the model can "see" when generating a response — your system prompt, the conversation history, any documents you've provided, and the message you just sent.

The **context window** is the maximum number of tokens the model can process at once. Think of it as the model's working memory. Anything outside this window doesn't exist to the model.

Modern context window sizes (2025):
- GPT-4o: 128K tokens
- Claude 3.5 Sonnet: 200K tokens
- Gemini 1.5 Pro: 1M tokens
- Llama 3.3 70B on Groq: 128K tokens
- GPT-OSS 120B on Groq: 128K tokens

One critical thing research has shown: **more context ≠ always better performance**. Models pay more attention to tokens at the *beginning and end* of the context window, with a drop-off in the middle (the "Lost in the Middle" problem). Stuffing 200K tokens of text doesn't guarantee the model uses all of it equally well.

### 3.3 Inference: How the Model Generates Text

Inference is the process of the model generating a response. Here's what actually happens when you send a message:

1. Your text is **tokenized** into a sequence of token IDs.
2. Each token ID is converted into a dense numerical vector (called an **embedding**).
3. The embeddings pass through many **Transformer layers**, where self-attention is applied.
4. The final layer outputs a probability distribution over the entire vocabulary (~50,000+ possible next tokens).
5. A token is **sampled** from this distribution (controlled by **temperature**).
6. That token is appended to the sequence, and the process repeats from step 1.

This is called **autoregressive generation** — the model generates one token at a time, conditioning each new token on all previous tokens.

**Temperature** controls how "creative" or "random" the sampling is:
- `temperature = 0`: Always pick the highest-probability token. Deterministic, but repetitive.
- `temperature = 0.7`: Balanced. Good for most use cases.
- `temperature = 1.0`: More random, more creative, potentially less coherent.
- `temperature > 1.0`: Very random, often incoherent.

### 3.4 The Transformer Architecture (Deep Dive)

Let's look at what happens inside a Transformer layer. This is where the magic lives.

#### Token Embeddings

Before entering the Transformer, each token is converted to a high-dimensional vector (e.g., 4096 dimensions for a 7B parameter model). These embeddings are *learned* during training and encode semantic meaning — similar words end up with similar vectors.

But there's a problem: the Transformer processes all tokens in parallel, so it has no inherent sense of order. We need to tell it that "dog bites man" ≠ "man bites dog."

**Positional Encoding** solves this by adding position information to each token's embedding, either through fixed sinusoidal functions (original Transformer) or learned position embeddings.

#### Self-Attention: The Core Innovation

Self-attention lets each token "look at" every other token in the sequence and decide how much to attend to it.

For each token, three vectors are computed:
- **Query (Q)** — "What am I looking for?"
- **Key (K)** — "What do I contain?"
- **Value (V)** — "What information should I pass on?"

The attention score between token A and token B is computed as:

```
attention_score(A, B) = dot_product(Query_A, Key_B) / sqrt(d_k)
```

These scores are passed through softmax to become weights (0 to 1, summing to 1 across all tokens). Then the output for each token is the weighted sum of all Value vectors.

The result: each token gets a new representation that incorporates context from all other relevant tokens. "bank" in a financial context attends strongly to "deposit" and "money"; in a nature context, it attends to "river" and "steep."

**Multi-Head Attention** runs this process multiple times in parallel with different learned Q/K/V projections. Each "head" captures different types of relationships — one might focus on syntactic dependencies, another on semantic similarity, another on positional patterns.

#### Feed-Forward Networks & Residual Connections

After attention, each token passes through a small fully-connected neural network (the Feed-Forward layer), applied identically and independently to each position.

**Residual connections** (skip connections) add the input of each sublayer to its output. This is critical for training deep networks — it prevents gradients from vanishing and lets information flow directly through many layers.

**Layer Normalization** is applied to stabilize training.

#### Stacking Layers

A full LLM stacks many of these Transformer layers (GPT-3 has 96 layers, Llama 3 70B has 80 layers). Each layer refines the token representations, adding progressively higher-level contextual understanding.

The final layer's output is passed through a linear projection (the "language model head") to produce logits over the vocabulary, then softmax for probabilities.

---

## 4. Types of Models and Their Capabilities

Not all AI models are created equal. Here's a breakdown of what exists and what each is good for.

### 4.1 General-Purpose Chat/Completion Models

These are the workhorses — trained to be helpful conversational assistants across a wide range of tasks.

**Examples:** GPT-4o, Claude Sonnet, Llama 3.3 70B, GPT-OSS 20B, Gemini 1.5 Pro

**Good for:** Content generation, summarization, translation, Q&A, coding help, brainstorming, customer support, classification.

**Tradeoffs:** Very capable across the board, but not specialized. If you need deep mathematical reasoning or you need to process 10,000 files overnight, you may want a different model type.

### 4.2 Reasoning Models

Reasoning models introduce a "think before you answer" approach. Before producing output, they generate an internal chain-of-thought — a scratchpad of reasoning steps — that isn't always visible to the user but heavily influences the final answer.

**Examples:** OpenAI o1, o3, Claude claude-3-5-sonnet-20241022 (extended thinking), DeepSeek R1, GPT-OSS 120B, Qwen3

**Good for:** Complex mathematical problems, multi-step coding challenges, legal/logical analysis, anything requiring careful multi-step reasoning.

**Tradeoffs:** Slower and more expensive (more tokens generated internally). Overkill for simple factual questions.

```
Reasoning model response to "What is 247 × 13?":
<thinking>
Let me compute this step by step:
247 × 13
= 247 × (10 + 3)
= 2470 + 741
= 3211
</thinking>
Answer: 3211
```

vs. a general model that might just say "3211" (hopefully correctly).

### 4.3 Embedding Models

These models don't generate text — they convert text into numerical vectors that capture semantic meaning. These vectors can then be compared for similarity.

**Examples:** text-embedding-3-small (OpenAI), all-MiniLM-L6-v2 (open-source)

**Good for:** Semantic search, RAG (Retrieval-Augmented Generation), clustering, recommendation systems, duplicate detection.

**Key concept:** Two sentences with similar meaning will have vectors that are "close" in embedding space, even if they use completely different words. "The car broke down" and "The vehicle stopped working" would be near-neighbors.

### 4.4 Vision/Multimodal Models

These models accept images (and sometimes audio and video) as input alongside text.

**Examples:** GPT-4o, Claude claude-3-5-sonnet-20241022, Gemini 1.5 Pro, Llama 4 Scout/Maverick

**Good for:** Document parsing (invoices, screenshots), image analysis, visual Q&A, OCR-like tasks, generating alt text, analyzing charts.

### 4.5 Instruction-Tuned vs. Base Models

A **base model** is pre-trained only on next-token prediction. It will continue whatever you give it — give it the beginning of a story, it continues the story. Give it "Q: What is 2+2?", it might output "A: 4" or it might output more questions.

An **instruction-tuned model** (also called a chat model or instruct model) has been fine-tuned using RLHF or similar techniques to follow instructions. It understands the difference between a system prompt and a user message, knows to answer questions, and refuses harmful requests.

For 99% of application development, you want an **instruction-tuned model**.

---

## 5. Prompt Engineering: The Art of Talking to AI

Your model's output quality is almost entirely determined by your prompt quality. This is not an exaggeration. The same model with a bad prompt and a great prompt can produce wildly different results. Prompt engineering is the discipline of crafting inputs that reliably produce the outputs you need.

### 5.1 The Elements of a Good Prompt

A high-quality prompt typically contains some or all of these elements:

#### Role / Persona
Tell the model who it is. This shapes its tone, knowledge focus, and response style.

```
You are a senior software engineer specializing in Node.js backend systems. 
You write clean, performant, well-commented code.
```

#### Task / Instruction
Be explicit about exactly what you want. Vague prompts produce vague results.

```
❌ "Write code for authentication"
✅ "Write a JWT authentication middleware for Express.js that validates the Authorization header, 
    decodes the token, attaches the user object to req.user, and returns 401 with a proper 
    error message if the token is invalid or expired."
```

#### Context
Provide relevant background the model needs to know.

```
I'm building a multi-tenant SaaS application. Each user belongs to one organization. 
The existing database schema has: users (id, email, org_id), organizations (id, name, plan).
```

#### Format / Output Structure
Tell the model exactly how you want the output structured.

```
Respond in this format:
1. Brief explanation (2-3 sentences)
2. Code block with full implementation
3. Example usage
4. Common pitfalls to avoid
```

#### Constraints
What the model should NOT do.

```
Do NOT use any external libraries. Only use Node.js built-ins and Express.
Keep the response under 200 words.
```

#### Examples
Show, don't just tell. This is often the most powerful element.

```
Here's an example of the style I want:
Input: "user login failed"
Output: "[2025-06-15T10:23:45Z] ERROR auth.login | user_id=null | reason=invalid_credentials | ip=192.168.1.1"

Now apply this format to log these events: ...
```

> 📖 For a comprehensive reference on prompt structure, check out Anthropic's official **[Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)** — it's one of the most thorough and practical resources available.

### 5.2 Prompting Strategies

#### Zero-Shot Prompting

No examples provided — just the instruction. Works well when the task is simple and well-defined.

```
Classify the sentiment of this review as Positive, Negative, or Neutral:
"The product arrived on time but the packaging was damaged."
```

Output: `Negative`

Simple, fast, effective for clear tasks. Can fail on nuanced or unusual requests.

#### Few-Shot Prompting

Provide 2–5 input-output examples before your actual request. Dramatically improves consistency and format adherence.

```
Classify the sentiment of these product reviews:

Review: "Amazing quality, will definitely buy again!" → Positive
Review: "Arrived broken, customer service was no help." → Negative
Review: "Product is okay, nothing special." → Neutral
Review: "Fast shipping but the color was slightly off." → Mixed

Review: "Absolute garbage. Returned immediately."
```

Output: `Negative`

Few-shot prompting is especially powerful for tasks with a specific output format, tone, or domain-specific classification scheme.

#### Chain-of-Thought (CoT) Prompting

Tell the model to reason step-by-step before giving the final answer. Dramatically improves performance on math, logic, and multi-step reasoning tasks.

**Zero-Shot CoT:** Just add "Think step by step." — that's it. This simple addition, discovered by researchers in 2022, can dramatically improve reasoning accuracy.

```
I have 3 boxes. Box A has 4 apples. Box B has twice as many as Box A. 
Box C has 3 fewer than Box B. How many apples total?
Think step by step.
```

Model output:
```
Step 1: Box A = 4 apples
Step 2: Box B = 2 × 4 = 8 apples
Step 3: Box C = 8 - 3 = 5 apples
Step 4: Total = 4 + 8 + 5 = 17 apples

Answer: 17 apples
```

Without "Think step by step," the model might just output "17" or even hallucinate a wrong answer. With it, you get traceable, verifiable reasoning.

**Few-Shot CoT:** Provide examples that themselves include step-by-step reasoning:

```
Q: If a train goes 60mph for 2 hours, how far does it travel?
A: Distance = speed × time = 60 × 2 = 120 miles.

Q: A store sells apples for $0.50 each. If you buy 7, what's the total after a 10% discount?
A: Base cost = 7 × $0.50 = $3.50. Discount = 10% of $3.50 = $0.35. Total = $3.50 - $0.35 = $3.15.

Q: I have a recipe for 4 people. I need to serve 10. The recipe uses 2 cups of flour. How much do I need?
```

#### Role Prompting

Assign the model a specific expert persona. This shapes not just tone but the actual knowledge and reasoning style the model applies.

```
You are an experienced DevOps engineer who has worked at multiple high-growth startups. 
You prioritize reliability, scalability, and security. When reviewing infrastructure decisions, 
you always consider cost, maintenance overhead, and failure modes.

Review this architecture and identify risks: [architecture description]
```

Vs just asking "Review this architecture" — the persona prompt produces a more focused, expert-level response.

#### System Prompt vs. User Prompt

When calling an LLM via API, you have two distinct input channels:

**System Prompt:** Sets the model's overall behavior, persona, and constraints. Persists across the entire conversation. This is where you put:
- Who the model is (role/persona)
- What it should/shouldn't do
- Formatting preferences
- Business context

**User Prompt:** The actual message from the user. Can be a single question, a document to analyze, a task to complete.

```javascript
const response = await groq.chat.completions.create({
  model: "openai/gpt-oss-20b",
  messages: [
    {
      role: "system",  // ← Sets behavior
      content: "You are a helpful assistant for a legal tech company. Always recommend consulting a licensed attorney. Respond in plain English, not legal jargon."
    },
    {
      role: "user",   // ← The actual request
      content: "What are the main differences between an LLC and a C-Corp?"
    }
  ]
});
```

#### Prompt Chaining

Break complex tasks into a sequence of simpler prompts, where the output of one becomes the input of the next.

```
Step 1 prompt: "Extract all action items from this meeting transcript: [transcript]"
→ Output: List of action items

Step 2 prompt: "Given these action items: [output from step 1], assign a priority (High/Medium/Low) 
and an estimated effort (hours) to each."
→ Output: Prioritized list

Step 3 prompt: "Given these prioritized items: [output from step 2], draft a follow-up email 
to the team."
→ Output: Email draft
```

This is far more reliable than trying to do all three steps in one giant prompt.

#### Self-Consistency

For high-stakes answers (math, logic, code), ask the model to solve the problem multiple times independently, then pick the most common answer. This reduces the impact of individual runs going wrong.

---

## 6. GenAI vs. Agentic AI vs. AI Agents — Clearing Up the Confusion

These terms get used interchangeably, but they mean different things. Here's the clear breakdown.

### Generative AI (GenAI)

**What it is:** A class of AI that *creates new content* from a prompt.

**How it works:** You send a prompt → model generates output → done. One turn, one response.

**Analogy:** A very knowledgeable employee who answers your questions perfectly but just sits there waiting for you to ask something new.

**Examples:** ChatGPT answering a question, Claude writing code, Midjourney generating an image, GitHub Copilot suggesting a completion.

**Key characteristic:** Reactive. It only does something when prompted. It has no memory, no persistent state (by default), and takes no action in the world.

### AI Agent

**What it is:** A system that uses an LLM as its "brain" but *also has tools* — the ability to take actions, fetch information, write files, call APIs, browse the web, etc.

**How it works:** You give it a goal → it plans, uses tools, checks results, replans, iterates — until the goal is achieved.

**Analogy:** An employee with both expertise *and* the ability to actually do things — look up documents, make phone calls, write emails, run code.

**Examples:** An AI that can search the web and then write a report based on the results. A coding agent that writes code, runs tests, fixes errors, and submits a PR.

**Key characteristics:**
- Uses LLM for reasoning and language
- Has access to external tools (web search, code execution, APIs, databases)
- Can take multi-step actions
- Has some form of memory (conversation history at minimum)

### Agentic AI

**What it is:** The broader framework or system that orchestrates one or more AI agents working autonomously toward goals.

**How it works:** Instead of one agent doing one task, agentic AI coordinates multiple specialized agents, manages workflow, maintains state across complex multi-step processes, and pursues objectives with minimal human intervention.

**Analogy:** An entire AI-powered team — one agent researches, one writes, one reviews, one deploys — all coordinated toward a single goal.

**Examples:** An autonomous software development system where one agent plans the architecture, another writes code, another runs tests, another handles deployment. An autonomous research assistant that spends hours searching, reading, and synthesizing information.

Here's a table that makes the distinctions crystal clear:

| Dimension | GenAI | AI Agent | Agentic AI |
|-----------|-------|----------|------------|
| **Core function** | Content creation | Goal-directed task execution | Autonomous multi-step workflow |
| **Autonomy** | None — reactive only | Limited — executes predefined tools | High — plans, adapts, orchestrates |
| **Memory** | No (per turn) | Short-term (conversation) | Long-term, cross-session |
| **Tools** | None | Yes (search, code, APIs) | Yes, plus orchestration |
| **Interaction** | Single prompt → response | Goal → multi-step action loop | High-level objective → autonomous workflow |
| **Human involvement** | Every interaction | Sets goal, reviews result | Sets objective, oversees |
| **Example** | "Write a blog post" | "Research this topic and write a blog post" | "Run my content marketing pipeline this week" |

> **The key mental model:** GenAI is a tool. An AI Agent is a worker with tools. Agentic AI is a system of workers with coordination.

Think of it this way:
- GenAI can *write code*.
- An AI Agent can *write code, run it, fix errors, and test it*.
- Agentic AI can *manage the entire software development lifecycle* — planning, writing, testing, deploying, monitoring — with minimal human input.

---

# PART 2: Hands-On with JavaScript

---

## Why JavaScript? Not Python?

This is a totally fair question because Python is the default language for everything AI/ML-related in most tutorials. Let's be honest about why:

**Python is the right choice for:**
- Training and fine-tuning models (PyTorch, TensorFlow, JAX)
- ML research and experimentation (Jupyter notebooks, numpy, pandas)
- Building ML pipelines and data processing
- Working with models at the weight level (Hugging Face transformers)
- Academic AI/ML work

Python is the undisputed king of the ML stack. If you're training models, PyTorch is in Python. If you're working with CUDA kernels, it's Python. If you're writing research papers with code, it's Python. Period.

**But JavaScript is the right choice for:**
- Building AI-native web applications
- Integrating LLM APIs into existing Node.js backends
- Creating real-time chat interfaces (React, Next.js)
- Building browser-based AI tools
- Shipping production AI features fast

And here's the practical reality for developers building apps (not models): **virtually every major AI SDK is available for both Python and JavaScript**. OpenAI SDK: both. Anthropic SDK: both. Groq SDK: both. LangChain: both (JS version is LangChain.js). The ecosystem gap that existed in 2022 is mostly gone.

If you're already a JS/TS developer building web apps and you want to add AI — JavaScript is the right tool. You don't need to context-switch to Python just to call an API.

---

## 7. Getting Hands-On with Groq Cloud

We're going to use **Groq Cloud** for all our hands-on examples. Here's why:

- **Free tier with no credit card required** — you get access to all models immediately.
- **Fastest inference available** — Groq's custom LPU (Language Processing Unit) chip runs models at 280–1,000+ tokens/second (3x–10x faster than GPU-based providers). This matters when you're iterating fast.
- **OpenAI-compatible API** — code you write for Groq works with minimal changes on OpenAI, Together AI, or any other provider.
- **Wide model selection** — Llama, GPT-OSS, Qwen3, Whisper, and more.

### Free Tier Rate Limits

On the free tier, you get access to all models with these general limits (as of mid-2026):

| Model | Requests/Day | Tokens/Minute |
|-------|-------------|---------------|
| GPT-OSS 20B | ~1,000 RPD | ~6,000 TPM |
| GPT-OSS 120B | ~1,000 RPD | ~6,000 TPM |
| Qwen3.6 27B | ~1,000 RPD | ~6,000 TPM |
| Whisper Large v3 | ~2,000 RPD | — |

This is more than enough to prototype, learn, and build small applications. The free tier's only constraints are rate limits — you get full access to production-quality models.

> ⚠️ **Note:** Groq regularly updates its model lineup. Always check [console.groq.com/docs/models](https://console.groq.com/docs/models) for the current list. As of June 2026, Groq deprecated `llama-3.1-8b-instant` and `llama-3.3-70b-versatile` in favor of the GPT-OSS model family.

### Setting Up

**Step 1: Create a Groq Account**
Go to [console.groq.com](https://console.groq.com) and sign up. No credit card required.

**Step 2: Get your API Key**
Navigate to API Keys in the console and create a new key. Copy it — you won't see it again.

**Step 3: Set up your project**

```bash
mkdir my-ai-app
cd my-ai-app
npm init -y
npm install groq-sdk dotenv
```

Create a `.env` file:
```
GROQ_API_KEY=gsk_your_key_here
```

> 🔒 Never commit your `.env` file. Add it to `.gitignore`.

---

## 8. Your First LLM Call: Two Ways

### Method 1: Using the Groq SDK

The SDK is the cleanest way. Full TypeScript types, automatic retries, and a familiar interface.

```javascript
// index.js
import Groq from "groq-sdk";
import "dotenv/config";

const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY,
});

async function main() {
  const response = await groq.chat.completions.create({
    model: "openai/gpt-oss-20b",
    messages: [
      {
        role: "system",
        content: "You are a helpful assistant. Be concise and clear.",
      },
      {
        role: "user",
        content: "What is the difference between REST and GraphQL?",
      },
    ],
    temperature: 0.7,
    max_tokens: 1024,
  });

  console.log(response.choices[0].message.content);
  
  // Token usage info
  console.log("\n--- Usage ---");
  console.log(`Prompt tokens: ${response.usage.prompt_tokens}`);
  console.log(`Completion tokens: ${response.usage.completion_tokens}`);
  console.log(`Total tokens: ${response.usage.total_tokens}`);
}

main();
```

Run it:
```bash
node index.js
```

Let's break down the key parts of the request:

```javascript
{
  model: "openai/gpt-oss-20b",        // Which model to use
  messages: [                          // The conversation history
    { role: "system", content: "..." }, // System prompt — model's instructions
    { role: "user", content: "..." }    // User's message
  ],
  temperature: 0.7,                    // Creativity/randomness (0 = deterministic, 1 = creative)
  max_tokens: 1024,                    // Max tokens to generate in the response
}
```

### Method 2: Raw REST API (no SDK)

Sometimes you want to understand what's actually happening under the hood, or you're in an environment without npm access. The raw API is just HTTP + JSON.

```javascript
// raw-api.js
import "dotenv/config";

async function callGroqAPI() {
  const response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.GROQ_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "openai/gpt-oss-20b",
      messages: [
        {
          role: "system",
          content: "You are an expert JavaScript developer.",
        },
        {
          role: "user",
          content: "Explain async/await in JavaScript in 3 bullet points.",
        },
      ],
      temperature: 0.5,
      max_tokens: 512,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Groq API error: ${error.error.message}`);
  }

  const data = await response.json();
  console.log(data.choices[0].message.content);
}

callGroqAPI();
```

Notice that the URL, headers, and body format are identical to what you'd send to the OpenAI API — just with a different base URL and your Groq key. This is intentional, and we'll talk about why shortly.

### Streaming Responses

For production apps (especially chat UIs), you want **streaming** — getting tokens as they're generated instead of waiting for the full response. This dramatically improves perceived latency.

```javascript
// streaming.js
import Groq from "groq-sdk";
import "dotenv/config";

const groq = new Groq({ apiKey: process.env.GROQ_API_KEY });

async function streamResponse() {
  const stream = await groq.chat.completions.create({
    model: "openai/gpt-oss-20b",
    messages: [
      {
        role: "user",
        content: "Write a short poem about software developers.",
      },
    ],
    stream: true,  // ← Enable streaming
  });

  process.stdout.write("Response: ");
  
  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || "";
    process.stdout.write(content);  // Print each token as it arrives
  }
  
  console.log("\n--- Stream complete ---");
}

streamResponse();
```

With streaming enabled, you receive Server-Sent Events (SSE) — a stream of partial chunks where `choices[0].delta.content` contains the next token(s).

### Multi-Turn Conversations

To have a real conversation, you pass the full message history with each request. The LLM has no persistent memory — you're responsible for maintaining history.

```javascript
// chat.js
import Groq from "groq-sdk";
import "dotenv/config";
import readline from "readline";

const groq = new Groq({ apiKey: process.env.GROQ_API_KEY });

// Maintain conversation history
const messages = [
  {
    role: "system",
    content: "You are a helpful coding assistant. Keep answers concise and practical.",
  },
];

async function chat(userMessage) {
  // Add the user's message to history
  messages.push({ role: "user", content: userMessage });

  const response = await groq.chat.completions.create({
    model: "openai/gpt-oss-20b",
    messages: messages,  // Send full history every time
    temperature: 0.7,
    max_tokens: 1024,
  });

  const assistantMessage = response.choices[0].message.content;
  
  // Add assistant's reply to history
  messages.push({ role: "assistant", content: assistantMessage });
  
  return assistantMessage;
}

// Simple CLI chat loop
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

console.log("Chat started. Type 'exit' to quit.\n");

function prompt() {
  rl.question("You: ", async (input) => {
    if (input.toLowerCase() === "exit") {
      rl.close();
      return;
    }

    try {
      const response = await chat(input);
      console.log(`\nAssistant: ${response}\n`);
    } catch (err) {
      console.error("Error:", err.message);
    }

    prompt();
  });
}

prompt();
```

---

## 9. The OpenAI-Compatible API Standard

You may have noticed that Groq's API format looks suspiciously like OpenAI's. That's completely intentional — and it's one of the most important things to understand about the modern AI ecosystem.

### What "OpenAI Compatible" Means

OpenAI was the first to define a clean, well-documented REST API for LLMs. Their format became ubiquitous:
- `POST /v1/chat/completions` endpoint
- Messages array with `role` and `content`
- Parameters like `temperature`, `max_tokens`, `stream`
- Response with `choices[0].message.content`

So many developers wrote code for this format, and so many frameworks (LangChain, LlamaIndex, etc.) built connectors for it, that it became the **de facto standard** for the entire industry — like HTTP for web, or SQL for databases.

Today, "OpenAI-compatible" means: *your existing OpenAI code works here, just change the base URL.*

Providers that are OpenAI-compatible: **Groq, Together AI, Fireworks, Mistral, Ollama (local), vLLM (self-hosted), Perplexity, and many more.**

### The Practical Power of This

With just two line changes, you can switch providers:

```javascript
// OpenAI
import OpenAI from "openai";
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// → Switch to Groq (for speed + free tier)
const client = new OpenAI({
  apiKey: process.env.GROQ_API_KEY,
  baseURL: "https://api.groq.com/openai/v1",
});

// → Switch to local Ollama (for privacy, no API cost)
const client = new OpenAI({
  apiKey: "ollama",  // Ollama ignores this
  baseURL: "http://localhost:11434/v1",
});

// → Switch to Together AI
const client = new OpenAI({
  apiKey: process.env.TOGETHER_API_KEY,
  baseURL: "https://api.together.ai/v1",
});
```

All other code stays **exactly the same**. This is incredibly powerful for:
- **Cost optimization:** Run expensive tasks on a cheap/fast provider, complex reasoning on a premium model
- **Vendor independence:** If a provider goes down or prices spike, you can switch in minutes
- **Local development:** Use a local Ollama model during development, switch to cloud for production

---

## 10. Tool / Function Calling — The Bridge to Real Capabilities

This is where things get genuinely powerful. Tool calling (also called function calling) is what transforms a conversational model into an **agent that can take action**.

### The Problem It Solves

LLMs are trained on data with a knowledge cutoff. They can't:
- Check the current weather
- Query your database
- Read a file
- Call your internal APIs
- Get real-time stock prices

Tool calling gives the model a way to request that your code perform these actions.

### How Tool Calling Works (The Full Loop)

```
Your Code                    Groq API / LLM
─────────────                ──────────────────────────────
1. Send message + tool schemas  →
                              ← 2. Model decides to call a tool,
                                    returns tool_call object (NOT final answer yet)
3. Execute the tool in your code
4. Send tool result back      →
                              ← 5. Model uses result to form final answer
```

The model itself *never* executes your function. It outputs structured JSON saying "I want to call this function with these arguments." Your code runs the function and sends back the result.

### Step-by-Step Implementation

Let's build a practical example: an assistant that can get weather data and do currency conversions.

#### Step 1: Define Your Tool Schemas

```javascript
// tools.js
export const tools = [
  {
    type: "function",
    function: {
      name: "get_weather",
      description: "Get the current weather for a specified city. Use this whenever the user asks about weather conditions.",
      parameters: {
        type: "object",
        properties: {
          city: {
            type: "string",
            description: "The city name, e.g. 'Mumbai', 'New York', 'London'",
          },
          unit: {
            type: "string",
            enum: ["celsius", "fahrenheit"],
            description: "Temperature unit. Default to celsius.",
          },
        },
        required: ["city"],
      },
    },
  },
  {
    type: "function",
    function: {
      name: "convert_currency",
      description: "Convert an amount from one currency to another using current rates.",
      parameters: {
        type: "object",
        properties: {
          amount: {
            type: "number",
            description: "The amount to convert",
          },
          from_currency: {
            type: "string",
            description: "Source currency code (e.g., USD, EUR, INR)",
          },
          to_currency: {
            type: "string",
            description: "Target currency code (e.g., USD, EUR, INR)",
          },
        },
        required: ["amount", "from_currency", "to_currency"],
      },
    },
  },
];
```

#### Step 2: Implement the Actual Functions

```javascript
// functions.js

// In a real app, you'd call actual weather/forex APIs here.
// We're using mock data for clarity.

export function get_weather({ city, unit = "celsius" }) {
  // Simulated weather data — replace with real API call
  const mockWeather = {
    mumbai: { temp_c: 32, temp_f: 89, condition: "Humid and cloudy", humidity: 85 },
    "new york": { temp_c: 22, temp_f: 72, condition: "Partly sunny", humidity: 60 },
    london: { temp_c: 15, temp_f: 59, condition: "Overcast with light rain", humidity: 78 },
  };

  const cityKey = city.toLowerCase();
  const weather = mockWeather[cityKey] || { temp_c: 20, temp_f: 68, condition: "Clear", humidity: 50 };

  const temp = unit === "fahrenheit" ? weather.temp_f : weather.temp_c;
  const unitSymbol = unit === "fahrenheit" ? "°F" : "°C";

  return {
    city,
    temperature: `${temp}${unitSymbol}`,
    condition: weather.condition,
    humidity: `${weather.humidity}%`,
  };
}

export function convert_currency({ amount, from_currency, to_currency }) {
  // Mock exchange rates relative to USD
  const rates = {
    USD: 1,
    EUR: 0.92,
    GBP: 0.79,
    INR: 83.5,
    JPY: 157.2,
    CAD: 1.36,
  };

  const fromRate = rates[from_currency.toUpperCase()];
  const toRate = rates[to_currency.toUpperCase()];

  if (!fromRate || !toRate) {
    return { error: `Unsupported currency. Supported: ${Object.keys(rates).join(", ")}` };
  }

  const amountInUSD = amount / fromRate;
  const convertedAmount = amountInUSD * toRate;

  return {
    original: `${amount} ${from_currency.toUpperCase()}`,
    converted: `${convertedAmount.toFixed(2)} ${to_currency.toUpperCase()}`,
    rate: `1 ${from_currency.toUpperCase()} = ${(toRate / fromRate).toFixed(4)} ${to_currency.toUpperCase()}`,
  };
}
```

#### Step 3: The Tool Calling Loop

```javascript
// agent.js
import Groq from "groq-sdk";
import "dotenv/config";
import { tools } from "./tools.js";
import { get_weather, convert_currency } from "./functions.js";

const groq = new Groq({ apiKey: process.env.GROQ_API_KEY });

// Map tool names to actual functions
const availableFunctions = {
  get_weather,
  convert_currency,
};

async function runAgent(userMessage) {
  console.log(`\nUser: ${userMessage}`);

  const messages = [
    {
      role: "system",
      content:
        "You are a helpful assistant. Use the available tools to answer questions about weather and currency. Always use tools when relevant — don't guess.",
    },
    { role: "user", content: userMessage },
  ];

  // ── TURN 1: Send message + tool schemas to model ──
  let response = await groq.chat.completions.create({
    model: "openai/gpt-oss-20b",
    messages,
    tools,                   // ← Pass tool definitions
    tool_choice: "auto",     // ← Let the model decide when to use tools
    temperature: 0.3,
    max_tokens: 1024,
  });

  let assistantMessage = response.choices[0].message;
  messages.push(assistantMessage); // Add assistant's response to history

  // ── TOOL EXECUTION LOOP ──
  // The model may call multiple tools in sequence
  while (assistantMessage.tool_calls && assistantMessage.tool_calls.length > 0) {
    console.log(`\n[Agent] Model wants to call ${assistantMessage.tool_calls.length} tool(s)...`);

    // Execute each tool call
    for (const toolCall of assistantMessage.tool_calls) {
      const functionName = toolCall.function.name;
      const functionArgs = JSON.parse(toolCall.function.arguments);

      console.log(`[Tool] Calling: ${functionName}(${JSON.stringify(functionArgs)})`);

      // Execute the actual function
      const functionToCall = availableFunctions[functionName];
      if (!functionToCall) {
        throw new Error(`Unknown tool: ${functionName}`);
      }

      const functionResult = functionToCall(functionArgs);
      console.log(`[Tool] Result: ${JSON.stringify(functionResult)}`);

      // Add tool result to message history
      messages.push({
        role: "tool",
        tool_call_id: toolCall.id,    // ← Must match the tool_call's id
        name: functionName,
        content: JSON.stringify(functionResult),
      });
    }

    // ── TURN 2+: Send tool results back, get final answer ──
    response = await groq.chat.completions.create({
      model: "openai/gpt-oss-20b",
      messages,
      tools,
      tool_choice: "auto",
      temperature: 0.3,
      max_tokens: 1024,
    });

    assistantMessage = response.choices[0].message;
    messages.push(assistantMessage);
  }

  // At this point, the model has no more tools to call — return final answer
  console.log(`\nAssistant: ${assistantMessage.content}`);
  return assistantMessage.content;
}

// Test it!
await runAgent("What's the weather like in Mumbai and New York right now?");
await runAgent("I have $500 USD. How much is that in Indian Rupees?");
await runAgent("If I'm visiting London tomorrow, what weather should I expect? Also, I have €200 — how much is that in GBP?");
```

Run it:
```bash
node agent.js
```

Expected output flow:
```
User: What's the weather like in Mumbai and New York right now?

[Agent] Model wants to call 2 tool(s)...
[Tool] Calling: get_weather({"city":"Mumbai","unit":"celsius"})
[Tool] Result: {"city":"Mumbai","temperature":"32°C","condition":"Humid and cloudy","humidity":"85%"}
[Tool] Calling: get_weather({"city":"New York","unit":"celsius"})
[Tool] Result: {"city":"New York","temperature":"22°C","condition":"Partly sunny","humidity":"60%"}Assistant: Here's the current weather in both cities:
- **Mumbai:** 32°C, Humid and cloudy with 85% humidity
- **New York:** 22°C, Partly sunny with 60% humidity
```

Notice how:
- The model called **both tools in parallel** in a single turn (parallel tool calling).
- After getting both results, it formulated a **coherent natural language response**.
- Your code executed the actual functions — the model only produced structured JSON intent.

### The `tool_choice` Parameter

```javascript
tool_choice: "auto"      // Model decides when to use tools (recommended default)
tool_choice: "none"      // Never use tools; respond with text only
tool_choice: "required"  // Must call at least one tool every turn
tool_choice: {
  type: "function",
  function: { name: "get_weather" }  // Force a specific tool
}
```

### Structured Output with Tools

One underused pattern: use tool calling purely for **structured output** — forcing the model to return JSON in a specific schema, even when there's no external API to call.

```javascript
// structured-output.js
import Groq from "groq-sdk";
import "dotenv/config";

const groq = new Groq({ apiKey: process.env.GROQ_API_KEY });

const extractProductTool = {
  type: "function",
  function: {
    name: "extract_product_info",
    description: "Extract structured product information from unstructured text",
    parameters: {
      type: "object",
      properties: {
        product_name: { type: "string", description: "Name of the product" },
        price: { type: "number", description: "Price in USD" },
        category: {
          type: "string",
          enum: ["electronics", "clothing", "food", "books", "other"],
        },
        in_stock: { type: "boolean" },
        rating: {
          type: "number",
          description: "Rating out of 5, if mentioned",
        },
      },
      required: ["product_name", "price", "category", "in_stock"],
    },
  },
};

async function extractProductInfo(text) {
  const response = await groq.chat.completions.create({
    model: "openai/gpt-oss-20b",
    messages: [
      {
        role: "user",
        content: `Extract the product information from this text: "${text}"`,
      },
    ],
    tools: [extractProductTool],
    tool_choice: { type: "function", function: { name: "extract_product_info" } },
  });

  const toolCall = response.choices[0].message.tool_calls[0];
  return JSON.parse(toolCall.function.arguments); // Clean, validated JSON
}

// Test it
const result = await extractProductInfo(
  "The Sony WH-1000XM5 headphones are on sale for $279. Currently available, 4.8 stars."
);
console.log(result);
// Output:
// {
//   product_name: "Sony WH-1000XM5",
//   price: 279,
//   category: "electronics",
//   in_stock: true,
//   rating: 4.8
// }
```

This is extremely useful for parsing unstructured text into clean, typed data that your application can use directly.

---

## 11. Which Model Should You Use When?

This is one of the most practical questions you'll face. Here's a clear framework.

### Decision Matrix

| Scenario | Recommended Model | Why |
|----------|-----------------|-----|
| Simple Q&A, chat | GPT-OSS 20B | Fast, cheap, handles 95% of tasks |
| Complex reasoning, math | GPT-OSS 120B, Qwen3 | Extended thinking capability |
| Code generation | GPT-OSS 20B or 120B | Both excel at code |
| High-volume, cost-sensitive | Lighter/faster models first | Lower token cost |
| Long documents (100k+ tokens) | Models with large context windows | Don't lose content |
| Audio transcription | Whisper Large v3 | Purpose-built for speech |
| Image understanding | Multimodal models (vision-capable) | Text-only models can't see images |
| Real-time / low-latency apps | Groq (any model) | LPU gives 3-10x speed advantage |

### Real Examples

**"I need to build a customer support chatbot"**
→ Use **GPT-OSS 20B**. It's fast, handles conversational tasks well, and the latency advantage from Groq means users get instant-feeling responses.

**"I need to analyze a 200-page legal contract and extract key clauses"**
→ Use a model with a **large context window** (128K+) like GPT-OSS 120B. The entire document needs to fit in context.

**"I need to solve complex coding challenges or debug hard algorithmic problems"**
→ Use a **reasoning model** like GPT-OSS 120B with higher reasoning effort. The step-by-step thinking dramatically improves accuracy on hard problems.

**"I'm processing 10,000 documents overnight"**
→ Use Groq's **Batch API** (50% discount on processing). Latency doesn't matter for batch; cost does.

**"I need real-time speech-to-text transcription"**
→ Use **Whisper Large v3** on Groq. It's purpose-built for audio.

**"I'm building a prototype and just want the fastest iteration cycle"**
→ Start with **GPT-OSS 20B**. It's fast, generous free tier, and handles most things. Upgrade if you hit capability limits.

### Cost vs. Capability Tradeoff

Think of it as a 2x2 matrix:

```
              LOW COMPLEXITY          HIGH COMPLEXITY
              (simple Q&A, chat)     (reasoning, analysis)
             ─────────────────────────────────────────────
HIGH VOLUME  │  Fastest/cheapest    │  Reasoning model   │
             │  model available     │  + batching        │
             ─────────────────────────────────────────────
LOW VOLUME   │  Any capable model   │  Best model        │
             │  (convenience wins)  │  regardless cost   │
             ─────────────────────────────────────────────
```

---

## 12. Complete Project: AI-Powered Code Reviewer

Let's put everything together. Here's a complete, practical CLI tool that reviews code using the full stack of what we've learned — system prompts, structured output, and a deliberate model choice.

```javascript
// code-reviewer.js
import Groq from "groq-sdk";
import "dotenv/config";
import { readFileSync } from "fs";

const groq = new Groq({ apiKey: process.env.GROQ_API_KEY });

// Tool for structured review output
const codeReviewTool = {
  type: "function",
  function: {
    name: "submit_code_review",
    description: "Submit a structured code review with specific findings",
    parameters: {
      type: "object",
      properties: {
        overall_score: {
          type: "number",
          description: "Score from 1-10 (10 = excellent)",
        },
        summary: {
          type: "string",
          description: "Brief 2-3 sentence summary of the code quality",
        },
        issues: {
          type: "array",
          items: {
            type: "object",
            properties: {
              severity: { type: "string", enum: ["critical", "major", "minor", "suggestion"] },
              line_hint: { type: "string", description: "Which part of code this relates to" },
              description: { type: "string", description: "What the issue is" },
              fix: { type: "string", description: "How to fix it" },
            },
            required: ["severity", "description", "fix"],
          },
        },
        positive_aspects: {
          type: "array",
          items: { type: "string" },
          description: "Things done well in this code",
        },
      },
      required: ["overall_score", "summary", "issues", "positive_aspects"],
    },
  },
};

async function reviewCode(code, language = "javascript") {
  const response = await groq.chat.completions.create({
    model: "openai/gpt-oss-120b", // Use reasoning model for deep analysis
    messages: [
      {
        role: "system",
        content: `You are an expert senior software engineer conducting a thorough code review.
Analyze the provided ${language} code for:
- Security vulnerabilities (XSS, injection, auth issues)
- Performance problems (N+1 queries, memory leaks, unnecessary re-renders)
- Code quality (naming, complexity, readability)
- Error handling gaps
- Best practice violations
Be specific, actionable, and educational. Point to concrete parts of the code.`,
      },
      {
        role: "user",
        content: `Please review this ${language} code:\n\n\`\`\`${language}\n${code}\n\`\`\``,
      },
    ],
    tools: [codeReviewTool],
    tool_choice: { type: "function", function: { name: "submit_code_review" } },
    temperature: 0.3, // Low temp for analytical, consistent output
  });

  const toolCall = response.choices[0].message.tool_calls[0];
  return JSON.parse(toolCall.function.arguments);
}

function formatReview(review) {
  const severityEmoji = {
    critical: "🚨",
    major: "⚠️",
    minor: "📝",
    suggestion: "💡",
  };

  console.log("\n" + "═".repeat(60));
  console.log("  CODE REVIEW REPORT");
  console.log("═".repeat(60));
  console.log(`\nOverall Score: ${review.overall_score}/10`);
  console.log(`\nSummary: ${review.summary}`);

  if (review.positive_aspects.length > 0) {
    console.log("\n✅ Positive Aspects:");
    review.positive_aspects.forEach((p) => console.log(`   • ${p}`));
  }

  if (review.issues.length > 0) {
    console.log("\n🔍 Issues Found:");
    review.issues.forEach((issue, i) => {
      console.log(`\n${i + 1}. ${severityEmoji[issue.severity]} [${issue.severity.toUpperCase()}]`);
      if (issue.line_hint) console.log(`   Location: ${issue.line_hint}`);
      console.log(`   Problem:  ${issue.description}`);
      console.log(`   Fix:      ${issue.fix}`);
    });
  } else {
    console.log("\n✅ No significant issues found!");
  }

  console.log("\n" + "═".repeat(60));
}

// Example: review a code snippet
const exampleCode = `
async function getUserData(userId) {
  const query = "SELECT * FROM users WHERE id = " + userId;
  const user = await db.query(query);
  const password = user.password;
  console.log("User fetched:", user);
  return user;
}

app.get('/user/:id', async (req, res) => {
  const data = await getUserData(req.params.id);
  res.send(data);
});
`;

const review = await reviewCode(exampleCode, "javascript");
formatReview(review);
```

This would catch several issues: SQL injection vulnerability, leaking sensitive data (password in response), no error handling, and logging sensitive data.

---

## 13. What's Next?

You now have a solid foundation. Here's where to go from here:

### Level Up: Topics to Explore

**RAG (Retrieval-Augmented Generation)**
Instead of putting all your data in the context window (slow, expensive), embed documents into a vector database (Pinecone, Qdrant, pgvector) and retrieve only the relevant chunks for each query. This is how you build AI systems over your own private data.

**Streaming UIs**
Build React components that stream LLM responses token-by-token for a ChatGPT-like UX using Server-Sent Events or Next.js streaming.

**LangChain.js / AI SDK**
Higher-level frameworks that abstract common patterns — chains, agents, retrieval, memory. Vercel's AI SDK is especially good for Next.js apps.

**Multi-Agent Systems**
Build systems where multiple specialized LLM agents collaborate — a planner agent, an executor agent, a reviewer agent — each focused on what it does best.

**Fine-Tuning**
Once you know what you want a model to do and have examples of doing it well, fine-tuning a base model can produce a smaller, cheaper, more consistent model for your specific use case.

**MCP (Model Context Protocol)**
An emerging standard (popularized by Anthropic) for connecting AI models to external tools and data sources. Groq supports MCP natively, letting you connect to external servers for web search, code execution, databases, and more without writing the integration yourself.

### Resources to Bookmark

- **[Groq Documentation](https://console.groq.com/docs/overview)** — Official docs for everything Groq (models, tool use, rate limits)
- **[Groq API Cookbook](https://github.com/groq/groq-api-cookbook)** — Real examples: RAG, function calling, multi-agent, batch processing
- **[Anthropic Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)** — Best-in-class guide on prompt engineering
- **[Vercel AI SDK](https://sdk.vercel.ai/docs)** — Best JS/TS framework for building AI-powered apps
- **[LangChain.js](https://js.langchain.com/)** — Comprehensive framework for LLM chains and agents
- **[Hugging Face](https://huggingface.co/)** — For open-source models, datasets, and research papers

---

## Summary: What We Covered

### Part 1 — The Foundations
- **Generative AI** creates new content from patterns learned during training — text, images, code, audio.
- Text generation before LLMs: rule-based → N-grams → RNNs → Transformers.
- The **2017 Transformer paper** ("Attention Is All You Need") was the pivotal moment that made modern LLMs possible.
- **ChatGPT (2022)** brought it to the mainstream; reasoning models (2024–2025) pushed capabilities further.
- **Tokens** are the atomic unit (~¾ of a word); the **context window** is the model's working memory.
- **Inference** = autoregressive next-token prediction, one token at a time.
- The Transformer uses **self-attention** to let every token consider every other token simultaneously — that's the core breakthrough.
- **GenAI** = content creation. **AI Agent** = LLM + tools + action loop. **Agentic AI** = orchestrated system of agents.

### Part 2 — The Code
- Use **Groq Cloud** for free, ultra-fast LLM inference (no credit card required).
- The **Groq SDK** and **raw REST API** both work — same OpenAI-compatible format.
- The **OpenAI API format** is the de facto standard — most providers accept the same JSON structure, just change the base URL.
- **Tool calling** = the model outputs structured JSON intent, your code executes the actual function, results go back to the model.
- **Model selection**: lightweight fast models for simple/high-volume tasks, reasoning models for complex analysis.

The best way to learn all of this? Build something. Pick a real problem — something that annoys you or something you wish existed — and build it using what you learned here. The concepts will cement themselves through doing.

---

*If this article helped you, share it with a fellow developer who's trying to navigate the AI space. The ecosystem moves fast, but the fundamentals you learned here will stay relevant regardless of which new models drop next month.*

*Happy building. 🚀*