# AI & ML Engineering Interview Questions: Deep Technical Breakdowns

**Your cheat sheet for AI engineering interviews.**

A growing collection of real AI and ML engineering interview questions, broken down with detailed technical answers. No surface-level definitions; each question walks through the actual mechanics: how transformers process tokens at inference time, why LLMs hallucinate, how RLHF shapes model behavior, what makes RAG retrieval fail, and the engineering decisions behind serving models in production.

---

## Who This Is For

These interview questions and answers are helpful for roles such as:

- **AI Engineer**
- **Gen AI Engineer**
- **LLM Engineer**
- **Agentic AI Engineer / AI Agent Engineer**
- **Forward Deployed Engineer**
- **AI Solutions Architect**
- **AI Platform Engineer**
- **Applied AI Engineer**
- **MLOps Engineer / LLMOps Engineer**

---

## How Each Question Is Structured

Every question includes:

- **The actual interview question** (as you'd hear it in a loop)
- **What they're really testing** (the hidden evaluation criteria)
- **A full technical breakdown** (the mechanical, under-the-hood answer)
- **Production implications** (why it matters beyond theory)
- **The curveball follow-up** (and how to handle it)

---

## Deep Dives (Detailed Breakdowns)

| # | Topic | Category | Link |
|---|-------|----------|------|
| 01 | **The Inference Loop** — What happens under the hood when a user prompts an LLM? | LLM Fundamentals | [Read →](questions/01-the-inference-loop/README.md) |
| 02 | **Foundation Models** — How they changed AI engineering | LLM Fundamentals | [Read →](llm-fundamentals/01-foundation-models.md) |
| 03 | **What Is an LLM** — Architecture, training, and how it works | LLM Fundamentals | [Read →](llm-fundamentals/02-what-is-an-llm.md) |
| 04 | **Inside ChatGPT** — Full request lifecycle from Enter to response | LLM Fundamentals | [Read →](llm-fundamentals/03-inside-chatgpt.md) |
| 05 | **Transformer Architecture** — Complete data flow walkthrough | LLM Fundamentals | [Read →](llm-fundamentals/04-transformer-architecture.md) |
| 06 | **Transformer Components** — Every building block explained | LLM Fundamentals | [Read →](llm-fundamentals/05-transformer-components.md) |
| 07 | **Tokenization** — BPE, subwords, and why tokens ≠ words | LLM Fundamentals | [Read →](llm-fundamentals/06-tokenization.md) |
| 08 | **Byte Pair Encoding** — The algorithm step by step | LLM Fundamentals | [Read →](llm-fundamentals/07-bpe.md) |
| 09 | **WordPiece & SentencePiece** — Tokenizer comparison | LLM Fundamentals | [Read →](llm-fundamentals/08-wordpiece-sentencepiece.md) |
| 10 | **Positional Encoding** — Sinusoidal, learned, RoPE, ALiBi | LLM Fundamentals | [Read →](llm-fundamentals/09-positional-encoding.md) |
| 11 | **Embeddings** — Vectors, similarity, and semantic geometry | LLM Fundamentals | [Read →](llm-fundamentals/10-embeddings.md) |
| 12 | **Q/K/V Attention** — The database lookup analogy | LLM Fundamentals | [Read →](llm-fundamentals/11-qkv-attention.md) |
| 13 | **Self-Attention** — How tokens relate to each other | LLM Fundamentals | [Read →](llm-fundamentals/12-self-attention.md) |
| 14 | **Cross Attention** — Connecting encoder and decoder | LLM Fundamentals | [Read →](llm-fundamentals/13-cross-attention.md) |
| 15 | **√dₖ Scaling Factor** — Why we scale attention scores | LLM Fundamentals | [Read →](llm-fundamentals/14-scaling-factor.md) |
| 16 | **Causal Masking** — Preventing future token leakage | LLM Fundamentals | [Read →](llm-fundamentals/15-causal-masking.md) |
| 17 | **Multi-Head Attention** — Parallel specialized heads | LLM Fundamentals | [Read →](llm-fundamentals/16-multi-head-attention.md) |
| 18 | **Feed-Forward Networks** — The knowledge store in Transformers | LLM Fundamentals | [Read →](llm-fundamentals/17-feed-forward-networks.md) |
| 19 | **Context Window** — Limits, costs, and optimization | LLM Fundamentals | [Read →](llm-fundamentals/18-context-window.md) |
| 20 | **Temperature** — Controlling randomness in generation | LLM Fundamentals | [Read →](llm-fundamentals/19-temperature.md) |
| 21 | **First Token Latency** — Prefill vs decode phases | LLM Fundamentals | [Read →](llm-fundamentals/20-first-token-latency.md) |
| 22 | **Top-p & Top-k Sampling** — Adaptive vs fixed decoding | LLM Fundamentals | [Read →](llm-fundamentals/21-top-p-top-k.md) |
| 23 | **Logits** — Raw scores, log-probs, constrained decoding | LLM Fundamentals | [Read →](llm-fundamentals/22-logits.md) |
| 24 | **Residual Connections** — Skip connections and gradient flow | LLM Fundamentals | [Read →](llm-fundamentals/23-residual-connections.md) |
| 25 | **Open vs Closed-Source LLMs** — Cost, privacy, quality trade-offs | LLM Fundamentals | [Read →](llm-fundamentals/24-open-vs-closed-source.md) |
| 26 | **Architecture Variants** — Encoder-only, decoder-only, encoder-decoder | LLM Fundamentals | [Read →](llm-fundamentals/25-encoder-decoder-architectures.md) |
| 27 | **KV Cache** — Eliminating redundant computation | LLM Fundamentals | [Read →](llm-fundamentals/26-kv-cache.md) |
| 28 | **Model Distillation** — Teacher-student knowledge transfer | LLM Fundamentals | [Read →](llm-fundamentals/27-model-distillation.md) |
| 29 | **Mixture of Experts (MoE)** — Sparse expert routing | LLM Fundamentals | [Read →](llm-fundamentals/28-mixture-of-experts.md) |
| 30 | **Dense vs Sparse Models** — Activation pattern trade-offs | LLM Fundamentals | [Read →](llm-fundamentals/29-dense-vs-sparse.md) |
| 31 | **Flash Attention** — IO-aware tiled attention computation | LLM Fundamentals | [Read →](llm-fundamentals/30-flash-attention.md) |
| 32 | **Cross-Entropy Loss** — The LLM training objective | LLM Fundamentals | [Read →](llm-fundamentals/31-cross-entropy-loss.md) |
| 33 | **GQA** — Grouped-Query Attention for inference efficiency | LLM Fundamentals | [Read →](llm-fundamentals/32-gqa.md) |
| 34 | **RoPE** — Rotary Position Embedding mechanics | LLM Fundamentals | [Read →](llm-fundamentals/33-rope.md) |
| 35 | **Layer Normalization** — Stabilizing deep network training | LLM Fundamentals | [Read →](llm-fundamentals/34-layer-normalization.md) |
| 36 | **RMSNorm** — Simplified normalization for modern LLMs | LLM Fundamentals | [Read →](llm-fundamentals/35-rmsnorm.md) |
| 37 | **Autoregressive Models** — Sequential token generation | LLM Fundamentals | [Read →](llm-fundamentals/36-autoregressive-models.md) |
| 38 | **AR vs MLM** — Autoregressive vs masked language modeling | LLM Fundamentals | [Read →](llm-fundamentals/37-autoregressive-vs-mlm.md) |
| 39 | **PPO** — Proximal Policy Optimization in RLHF | LLM Fundamentals | [Read →](llm-fundamentals/38-ppo.md) |
| 40 | **DPO** — Direct Preference Optimization | LLM Fundamentals | [Read →](llm-fundamentals/39-dpo.md) |
| 41 | **GRPO** — Group Relative Policy Optimization | LLM Fundamentals | [Read →](llm-fundamentals/40-grpo.md) |
| 42 | **Recursive Language Models** — Iterative refinement | LLM Fundamentals | [Read →](llm-fundamentals/41-recursive-language-models.md) |
| 43 | **Continual Learning** — Updating LLMs without forgetting | LLM Fundamentals | [Read →](llm-fundamentals/42-continual-learning.md) |
| 44 | **Diffusion Language Models** — Non-autoregressive generation | LLM Fundamentals | [Read →](llm-fundamentals/43-diffusion-language-models.md) |
| 45 | **Small Language Models** — Efficient 1-7B models | LLM Fundamentals | [Read →](llm-fundamentals/44-small-language-models.md) |
| 46 | **Large Reasoning Models** — Test-time compute scaling | LLM Fundamentals | [Read →](llm-fundamentals/45-large-reasoning-models.md) |
| 47 | **What Is Prompt Engineering** — The prompt stack and discipline | Prompt Engineering | [Read →](prompt-engineering/01-what-is-prompt-engineering.md) |
| 48 | **Zero/One/Few-Shot** — In-context learning spectrum | Prompt Engineering | [Read →](prompt-engineering/02-zero-one-few-shot.md) |
| 49 | **Chain-of-Thought** — Step-by-step reasoning | Prompt Engineering | [Read →](prompt-engineering/03-chain-of-thought.md) |
| 50 | **Self-Consistency** — Majority voting over CoT chains | Prompt Engineering | [Read →](prompt-engineering/04-self-consistency.md) |
| 51 | **Tree-of-Thought** — Branching search with evaluation | Prompt Engineering | [Read →](prompt-engineering/05-tree-of-thought.md) |
| 52 | **ReAct Prompting** — Reasoning + Acting loops | Prompt Engineering | [Read →](prompt-engineering/06-react-prompting.md) |
| 53 | **System Prompts** — Persistent behavioral instructions | Prompt Engineering | [Read →](prompt-engineering/07-system-prompts.md) |
| 54 | **Structured Output** — JSON/XML schema enforcement | Prompt Engineering | [Read →](prompt-engineering/08-structured-output-prompting.md) |
| 55 | **Prompt Injection** — Attack vectors and defenses | Prompt Engineering | [Read →](prompt-engineering/09-prompt-injection.md) |
| 56 | **Jailbreaking** — Safety bypass techniques | Prompt Engineering | [Read →](prompt-engineering/10-jailbreaking.md) |
| 57 | **Cost & Latency** — Optimizing prompt economics | Prompt Engineering | [Read →](prompt-engineering/11-cost-latency-optimization.md) |
| 58 | **Engineering vs Tuning** — Hard vs soft prompts | Prompt Engineering | [Read →](prompt-engineering/12-engineering-vs-tuning.md) |
| 59 | **Prompt Templates** — Parameterized production prompts | Prompt Engineering | [Read →](prompt-engineering/13-prompt-templates.md) |
| 60 | **Multi-Turn** — Conversation context management | Prompt Engineering | [Read →](prompt-engineering/14-multi-turn-conversations.md) |
| 61 | **Role Prompting** — Persona-based behavior control | Prompt Engineering | [Read →](prompt-engineering/15-role-prompting.md) |
| 62 | **Prompt Chaining** — Sequential task decomposition | Prompt Engineering | [Read →](prompt-engineering/16-prompt-chaining.md) |
| 63 | **Evaluating Prompts** — Metrics, LLM-as-judge, DSPy | Prompt Engineering | [Read →](prompt-engineering/17-evaluating-prompt-quality.md) |
| 64 | **Meta-Prompts** — Prompts that generate prompts | Prompt Engineering | [Read →](prompt-engineering/18-meta-prompts.md) |
| 65 | **Failure Modes** — Debugging prompt issues | Prompt Engineering | [Read →](prompt-engineering/19-failure-modes.md) |
| 66 | **Edge Cases** — Adversarial and boundary inputs | Prompt Engineering | [Read →](prompt-engineering/20-edge-cases.md) |
| 67 | **Lost in the Middle** — Long-context attention bias | Prompt Engineering | [Read →](prompt-engineering/21-lost-in-the-middle.md) |
| 68 | **Output Parsers** — Text to structured data | Prompt Engineering | [Read →](prompt-engineering/22-output-parsers.md) |
| 69 | **Multi-Language** — Cross-lingual prompting | Prompt Engineering | [Read →](prompt-engineering/23-multi-language-prompting.md) |

> More deep dives are added regularly. ⭐ Star the repo to stay updated.

---

## Table of Contents

- [Must Know](#must-know)
- [LLM Fundamentals](#llm-fundamentals)
- [Prompt Engineering](#prompt-engineering)
- [Retrieval-Augmented Generation (RAG)](#retrieval-augmented-generation-rag)
- [AI Agents and Agentic Systems](#ai-agents-and-agentic-systems)
- [Fine-Tuning and Model Adaptation](#fine-tuning-and-model-adaptation)
- [Vector Databases and Embeddings](#vector-databases-and-embeddings)
- [AI System Design](#ai-system-design)
- [LLMOps and Production AI](#llmops-and-production-ai)
- [Evaluation and Testing](#evaluation-and-testing)
- [AI Safety, Ethics, and Responsible AI](#ai-safety-ethics-and-responsible-ai)
- [Multimodal AI](#multimodal-ai)
- [AI Infrastructure and Scalability](#ai-infrastructure-and-scalability)
- [Coding and Practical Implementation](#coding-and-practical-implementation)
- [Behavioral and Scenario-Based Questions](#behavioral-and-scenario-based-questions)

---

## Must Know

| Topic | Status |
|-------|--------|
| LLM | 🔜 |
| RAG | 🔜 |
| MCP | 🔜 |
| Agent | 🔜 |
| Fine-tuning | 🔜 |
| Quantization | 🔜 |

---

## LLM Fundamentals

### Conceptual Questions

- What are foundation models, and how have they changed AI engineering? — [Read Answer →](llm-fundamentals/01-foundation-models.md)
- What is a Large Language Model (LLM), and how does it work? — [Read Answer →](llm-fundamentals/02-what-is-an-llm.md)
- Inside ChatGPT: What Happens After You Hit Enter? — [Read Answer →](llm-fundamentals/03-inside-chatgpt.md)
- What is the Transformer architecture and how does it work? — [Read Answer →](llm-fundamentals/04-transformer-architecture.md)
- What are the key components of the Transformer architecture? — [Read Answer →](llm-fundamentals/05-transformer-components.md)
- What is tokenization in LLMs? — [Read Answer →](llm-fundamentals/06-tokenization.md)
- Explain BPE (Byte Pair Encoding). — [Read Answer →](llm-fundamentals/07-bpe.md)
- Explain WordPiece and SentencePiece. — [Read Answer →](llm-fundamentals/08-wordpiece-sentencepiece.md)
- What is positional encoding, and why is it needed in Transformers? — [Read Answer →](llm-fundamentals/09-positional-encoding.md)
- What are embeddings? — [Read Answer →](llm-fundamentals/10-embeddings.md)
- Explain the Query(Q), Key(K), and Value(V) in attention. — [Read Answer →](llm-fundamentals/11-qkv-attention.md)
- What is self-attention, and how does it work in Transformers? — [Read Answer →](llm-fundamentals/12-self-attention.md)
- What is Cross Attention in Transformers? — [Read Answer →](llm-fundamentals/13-cross-attention.md)
- Why do we scale the dot product attention by √dₖ in the Transformer architecture? — [Read Answer →](llm-fundamentals/14-scaling-factor.md)
- What is causal masking? — [Read Answer →](llm-fundamentals/15-causal-masking.md)
- What are multi-head attention mechanisms? Why use multiple attention heads? — [Read Answer →](llm-fundamentals/16-multi-head-attention.md)
- What are Feed-Forward Networks in LLMs? — [Read Answer →](llm-fundamentals/17-feed-forward-networks.md)
- What is the context window in LLMs, and why does it matter? — [Read Answer →](llm-fundamentals/18-context-window.md)
- Why is the context window limited in LLMs? — [Read Answer →](llm-fundamentals/18-context-window.md)
- What is temperature in the context of LLMs, and how does it affect output? — [Read Answer →](llm-fundamentals/19-temperature.md)
- Why is the first token slower than the rest in an LLM? — [Read Answer →](llm-fundamentals/20-first-token-latency.md)
- Explain Top-p (nucleus) sampling and Top-k sampling. How do they differ? — [Read Answer →](llm-fundamentals/21-top-p-top-k.md)
- What are logits, and how are they used in text generation? — [Read Answer →](llm-fundamentals/22-logits.md)
- What are skip connections (residual connections) in Transformers? — [Read Answer →](llm-fundamentals/23-residual-connections.md)
- What is the difference between open-source and closed-source LLMs? When would you choose one over the other? — [Read Answer →](llm-fundamentals/24-open-vs-closed-source.md)
- What is the difference between encoder-only, decoder-only, and encoder-decoder Transformer architectures? — [Read Answer →](llm-fundamentals/25-encoder-decoder-architectures.md)
- What is KV cache, and how does it speed up inference? — [Read Answer →](llm-fundamentals/26-kv-cache.md)
- What is model distillation, and how is it used with LLMs? — [Read Answer →](llm-fundamentals/27-model-distillation.md)
- What is Mixture of Experts (MoE), and how does it work in models like Mixtral? — [Read Answer →](llm-fundamentals/28-mixture-of-experts.md)
- What is the difference between dense and sparse models? — [Read Answer →](llm-fundamentals/29-dense-vs-sparse.md)
- What is Flash Attention? — [Read Answer →](llm-fundamentals/30-flash-attention.md)
- What is Cross-Entropy Loss? — [Read Answer →](llm-fundamentals/31-cross-entropy-loss.md)
- What is Grouped-Query Attention (GQA), and how does it differ from Multi-Head Attention (MHA)? — [Read Answer →](llm-fundamentals/32-gqa.md)
- How does Rotary Position Embedding (RoPE) work, and why is it preferred over learned positional embeddings? — [Read Answer →](llm-fundamentals/33-rope.md)
- Explain Layer Normalization. — [Read Answer →](llm-fundamentals/34-layer-normalization.md)
- Explain RMSNorm (Root Mean Square Layer Normalization). — [Read Answer →](llm-fundamentals/35-rmsnorm.md)
- What are Autoregressive Models? — [Read Answer →](llm-fundamentals/36-autoregressive-models.md)
- Explain the difference between autoregressive and masked language modeling. — [Read Answer →](llm-fundamentals/37-autoregressive-vs-mlm.md)
- Proximal Policy Optimization (PPO). — [Read Answer →](llm-fundamentals/38-ppo.md)
- Direct Preference Optimization (DPO). — [Read Answer →](llm-fundamentals/39-dpo.md)
- Group Relative Policy Optimization (GRPO). — [Read Answer →](llm-fundamentals/40-grpo.md)
- Recursive Language Models (RLMs). — [Read Answer →](llm-fundamentals/41-recursive-language-models.md)
- Continual Learning in LLMs. — [Read Answer →](llm-fundamentals/42-continual-learning.md)
- How do Diffusion Language Models (DLMs) work? — [Read Answer →](llm-fundamentals/43-diffusion-language-models.md)
- Small Language Models (SLMs). — [Read Answer →](llm-fundamentals/44-small-language-models.md)
- Large Reasoning Models (LRMs). — [Read Answer →](llm-fundamentals/45-large-reasoning-models.md)

### Scenario-Based Questions

- Your LLM keeps ignoring your instructions. How do you make it follow structured output formats? — [Read Answer →](llm-fundamentals/46-scenario-structured-output.md)
- Your LLM-powered tool hits the context window limit on long documents. How do you handle it? — [Read Answer →](llm-fundamentals/47-scenario-context-overflow.md)
- Your LLM does not admit when it does not know the answer. How do you make it say "I don't know"? — [Read Answer →](llm-fundamentals/48-scenario-i-dont-know.md)
- Your LLM generates responses that are too verbose. How do you control response length? — [Read Answer →](llm-fundamentals/49-scenario-verbosity.md)
- Your LLM memorized proprietary training data and leaks it in responses. How do you prevent this? — [Read Answer →](llm-fundamentals/50-scenario-data-leakage.md)
- Your LLM coding assistant generates outdated code using deprecated libraries. How do you fix it? — [Read Answer →](llm-fundamentals/51-scenario-outdated-code.md)
- Your tokenizer splits important domain terms into meaningless subword pieces. How do you fix it? — [Read Answer →](llm-fundamentals/52-scenario-tokenizer-domain.md)
- Your Transformer's KV cache grows too large during long sequence generation. How do you manage memory? — [Read Answer →](llm-fundamentals/53-scenario-kv-cache-memory.md)
- Your Transformer runs out of memory on long documents due to quadratic self-attention. How do you scale it? — [Read Answer →](llm-fundamentals/54-scenario-quadratic-attention.md)
- Your distilled student model fails on the complex reasoning that the teacher model handled. How do you close the gap? — [Read Answer →](llm-fundamentals/55-scenario-distillation-gap.md)
- After RLHF alignment, your LLM became safer but lost capability on hard tasks. How do you manage the alignment tax? — [Read Answer →](llm-fundamentals/56-scenario-alignment-tax.md)
- Your RLHF-trained LLM is gaming the reward model instead of being genuinely helpful. How do you fix reward hacking? — [Read Answer →](llm-fundamentals/57-scenario-reward-hacking.md)
- Your chatbot loses context after 10 turns in a conversation. How do you maintain a long conversation context? — [Read Answer →](llm-fundamentals/58-scenario-conversation-context.md)
- Your chatbot fails when users switch topics mid-conversation. How do you handle topic switches? — [Read Answer →](llm-fundamentals/59-scenario-topic-switching.md)
- Your QA system always generates an answer even when no answer exists in the context. How do you detect unanswerable questions? — [Read Answer →](llm-fundamentals/60-scenario-unanswerable.md)
- Your summarization system hallucinated facts not in the original article. How do you fix it? — [Read Answer →](llm-fundamentals/61-scenario-summarization-hallucination.md)
- Your text generation repeats phrases in long outputs. How do you fix repetition? — [Read Answer →](llm-fundamentals/62-scenario-repetition.md)
- Transformers work on text, so can they also understand images? — [Read Answer →](llm-fundamentals/63-scenario-multimodal-vision.md)

---

## Prompt Engineering

### Conceptual Questions

- What is prompt engineering, and why is it critical for AI applications? — [Read Answer →](prompt-engineering/01-what-is-prompt-engineering.md)
- Explain zero-shot, one-shot, and few-shot prompting with examples. — [Read Answer →](prompt-engineering/02-zero-one-few-shot.md)
- What is chain-of-thought (CoT) prompting, and when should you use it? — [Read Answer →](prompt-engineering/03-chain-of-thought.md)
- Explain self-consistency prompting and how it improves reasoning. — [Read Answer →](prompt-engineering/04-self-consistency.md)
- What is tree-of-thought prompting? — [Read Answer →](prompt-engineering/05-tree-of-thought.md)
- What is ReAct (Reasoning + Acting) prompting, and how does it work? — [Read Answer →](prompt-engineering/06-react-prompting.md)
- What is a system prompt, and how does it influence model behavior? — [Read Answer →](prompt-engineering/07-system-prompts.md)
- How do you structure prompts for consistent structured output (JSON, XML)? — [Read Answer →](prompt-engineering/08-structured-output-prompting.md)
- What is prompt injection, and how do you defend against it? — [Read Answer →](prompt-engineering/09-prompt-injection.md)
- What is jailbreaking in LLMs, and what are common jailbreak techniques? — [Read Answer →](prompt-engineering/10-jailbreaking.md)
- How do you optimize prompts for cost and latency? — [Read Answer →](prompt-engineering/11-cost-latency-optimization.md)
- What is the difference between prompt engineering and prompt tuning? — [Read Answer →](prompt-engineering/12-engineering-vs-tuning.md)
- What is a prompt template, and how do you design one for production use? — [Read Answer →](prompt-engineering/13-prompt-templates.md)
- How do you handle multi-turn conversations with LLMs? — [Read Answer →](prompt-engineering/14-multi-turn-conversations.md)
- What is role prompting, and when is it effective? — [Read Answer →](prompt-engineering/15-role-prompting.md)
- What is prompt chaining, and how do you design a chain of prompts for complex tasks? — [Read Answer →](prompt-engineering/16-prompt-chaining.md)
- How do you evaluate and iterate on prompt quality? — [Read Answer →](prompt-engineering/17-evaluating-prompt-quality.md)
- What are meta-prompts, and how can they be used to generate prompts? — [Read Answer →](prompt-engineering/18-meta-prompts.md)
- What are the common failure modes in prompting, and how do you debug them? — [Read Answer →](prompt-engineering/19-failure-modes.md)
- How do you handle edge cases and adversarial inputs in prompt design? — [Read Answer →](prompt-engineering/20-edge-cases.md)
- What is the "lost in the middle" problem in long-context prompting? — [Read Answer →](prompt-engineering/21-lost-in-the-middle.md)
- What are output parsers, and why are they needed for production applications? — [Read Answer →](prompt-engineering/22-output-parsers.md)
- How do you handle multi-language prompting effectively? — [Read Answer →](prompt-engineering/23-multi-language-prompting.md)

### Scenario-Based Questions

- Your few-shot prompting gives inconsistent results across similar inputs. How do you stabilize it? — [Read Answer →](prompt-engineering/24-scenario-inconsistent-few-shot.md)
- Your LLM classification system is too sensitive to prompt wording changes. How do you reduce prompt sensitivity? — [Read Answer →](prompt-engineering/25-scenario-prompt-sensitivity.md)
- Your chatbot's system prompt containing proprietary business logic is being leaked by users. How do you prevent it? — [Read Answer →](prompt-engineering/26-scenario-system-prompt-leak.md)
- Your LLM agent is vulnerable to prompt injection that reveals the system prompt. How do you defend it? — [Read Answer →](prompt-engineering/27-scenario-injection-defense.md)
- Your chain-of-thought prompting is not improving LLM accuracy on reasoning tasks. What do you fix? — [Read Answer →](prompt-engineering/28-scenario-cot-not-working.md)
- Your AI system works in English but fails for other languages. How do you add multilingual support? — [Read Answer →](prompt-engineering/29-scenario-multilingual-support.md)
- Your zero-shot cross-lingual transfer from English fails on other languages. How do you fix it? — [Read Answer →](prompt-engineering/30-scenario-cross-lingual-failure.md)

---

## Retrieval-Augmented Generation (RAG)

### Conceptual Questions

- What is Retrieval-Augmented Generation (RAG), and why is it important?
- Explain the architecture of a basic RAG system.
- What are the key components of a RAG pipeline?
- What are chunking strategies, and how do you choose the right chunk size?
- Compare fixed-size chunking, semantic chunking, and recursive chunking.
- What are embedding models, and how do they convert text to vectors?
- How do you choose an embedding model for your RAG system?
- Explain Agentic RAG.
- What is hybrid search, and why is it better than pure vector search?
- What is re-ranking, and how does it improve RAG retrieval quality?
- How do you handle multi-document and multi-hop questions in RAG?
- What is the "lost in the middle" problem in RAG systems?
- How do you evaluate a RAG system? Explain faithfulness, relevance, and context precision/recall.
- Explain Self-RAG. How does the model decide when to retrieve?
- What is GraphRAG, and when would you use it over traditional RAG?
- How do you handle structured data (tables, SQL databases) in a RAG pipeline?
- What are the common failure modes of RAG systems, and how do you debug them?
- How do you handle document updates and maintain freshness in a RAG system?
- How do you optimize RAG for latency in production?
- What is the role of metadata filtering in RAG systems?
- Compare RAG vs fine-tuning. When would you use each?
- What is query transformation in RAG (HyDE, query decomposition, step-back prompting)?
- How do you implement citation and source attribution in RAG?
- How do you scale a RAG system to millions of documents?
- What is parent-child chunking, and how does it improve retrieval?

### Scenario-Based Questions

- Your RAG system is hallucinating despite having the right context. How do you fix it?
- Your RAG chunk overlap causes redundant results. How do you reduce redundancy?
- Your RAG retrieval is too slow with a large knowledge base. How do you speed it up?
- Your RAG system returns duplicate results. How do you deduplicate?
- Your RAG system needs per-user access control on internal documents. How do you implement it?
- Your RAG system fails on domain-specific jargon. How do you fix it?
- Your text-only RAG system now needs to handle images and tables. How do you extend it?
- Your RAG knowledge base gets updated frequently and needs versioning. How do you manage it?
- Your RAG system fails on multi-hop questions that require combining multiple facts. How do you fix it?
- Your enterprise RAG system returns contradictory answers from different source documents. How do you resolve conflicts?
- Your RAG system returns outdated answers from an evolving knowledge base. How do you keep it current?
- Your RAG system struggles with PDF documents containing tables and layouts. How do you fix PDF parsing?

---

## AI Agents and Agentic Systems

### Conceptual Questions

- What is an AI agent, and how does it differ from a simple LLM call?
- AI Agent Memory.
- Harness Engineering in AI.
- Explain the ReAct (Reasoning + Acting) agent architecture.
- What is the Plan-and-Execute agent pattern?
- What is tool use (function calling) in LLMs, and how does it enable agents?
- How do you design and define tools for an AI agent?
- What is the difference between single-agent and multi-agent systems?
- What is Model Context Protocol (MCP), and how does it standardize tool integration?
- What are AI SubAgents?
- What are the different types of agent memory (short-term, long-term, episodic)?
- How do you handle agent failures and implement error recovery?
- What is an agent loop, and how does it decide when to stop?
- Context Engineering.
- How AI Agents Communicate?
- How do you evaluate and test AI agents?
- What are the security risks of agentic systems, and how do you mitigate them?
- What is the difference between reactive and proactive agents?
- How do you manage token consumption and cost in long-running agent workflows?
- What is the human-in-the-loop pattern for agents, and when is it needed?
- How do you implement guardrails for AI agents to prevent harmful actions?
- What is agent reflection, and how does it improve agent performance?
- What is the difference between code-generating agents and tool-calling agents?
- How do you handle multi-modal inputs and outputs in agentic systems?
- How do you implement state management in complex agent workflows?
- How do you build a customer support agent with escalation logic?
- What is agent orchestration, and how do you implement it?
- How do you build a code execution agent safely using sandboxed environments?
- How do Computer-Use Agents work?
- How does LangChain work?
- How does LangGraph work?

### Scenario-Based Questions

- Your AI agent is stuck in an infinite loop. How do you detect and break the cycle?
- Your AI agent gets conflicting answers from different tools. How does it reconcile them?
- Your AI agent burns too many tokens per task. How do you reduce token consumption?
- Your AI agent keeps exceeding its budget per task. How do you enforce budget limits?
- Your AI agent hallucinates tool capabilities and passes wrong inputs. How do you fix it?
- Your AI agent deleted a production database. How do you prevent irreversible actions?
- Your AI agent has many tools, but keeps picking the wrong one. How do you improve tool selection?
- Your AI agent takes too long to complete a task. How do you speed it up?
- Your LLM selects the right tool but extracts the wrong parameters. How do you fix parameter extraction?

---

## Fine-Tuning and Model Adaptation

### Conceptual Questions

- What is fine-tuning, and when should you fine-tune an LLM?
- Explain the difference between full fine-tuning and parameter-efficient fine-tuning (PEFT).
- What is LoRA (Low-Rank Adaptation), and how does it work?
- What is QLoRA, and how does it enable fine-tuning on consumer hardware?
- Explain Prefix Tuning and Prompt Tuning. How are they different from LoRA?
- What is adapter-based fine-tuning?
- What is RLHF (Reinforcement Learning from Human Feedback), and how is it used to align LLMs?
- What is instruction tuning, and why is it important for chat models?
- How do you prepare a dataset for fine-tuning an LLM?
- What is catastrophic forgetting, and how do you prevent it during fine-tuning?
- When should you choose fine-tuning over RAG over prompt engineering?
- How do you evaluate a fine-tuned model's performance?
- What is synthetic data generation, and how do you use it for fine-tuning?
- What are the key hyperparameters for fine-tuning (learning rate, epochs, batch size, LoRA rank)?
- How do you fine-tune a model for a specific domain (legal, medical, finance)?
- What is continual pre-training, and when would you use it?
- How do you merge multiple LoRA adapters?
- What is the difference between SFT (Supervised Fine-Tuning) and alignment training?
- What is RLAIF (RL from AI Feedback), and how does it differ from RLHF?
- What is knowledge distillation for fine-tuning, and what are the legal considerations?

### Scenario-Based Questions

- Your fine-tuned LLM produces factually wrong outputs due to training data quality issues. How do you fix it?
- You must choose between LoRA and full fine-tuning for a domain-specific assistant. How do you decide?
- Your fine-tuned model memorized training data verbatim instead of learning patterns. How do you fix overfitting?
- Your fine-tuned LLM forgot its general capabilities after domain-specific fine-tuning. How do you fix catastrophic forgetting?
- Your RLHF preference data has low annotator agreement. How do you ensure data quality?

---

## Vector Databases and Embeddings

### Conceptual Questions

- What are embeddings in the context of AI engineering?
- How do embedding models convert text to vectors?
- What is the difference between sparse and dense embeddings?
- Explain cosine similarity, dot product, and Euclidean distance for vector search.
- What is a vector database, and how does it differ from a traditional database?
- How does Approximate Nearest Neighbor (ANN) search work?
- How do you choose the right embedding model for your use case?
- What is embedding dimensionality, and how does it affect performance and cost?
- How do you handle embedding drift when the embedding model is updated?
- What are multi-modal embeddings, and how are they generated?
- How do you index and query multi-tenant data in a vector database?
- What is quantization of embeddings, and how does it reduce storage costs?
- How do you benchmark and evaluate embedding model quality?
- What is the role of metadata in vector databases?
- How do you handle large-scale vector search with billions of vectors?
- What is hybrid search (combining keyword search with vector search)?
- How do you fine-tune an embedding model for a specific domain?

### Scenario-Based Questions

- Your vector database for RAG is consuming too much memory. How do you reduce it?
- Your vector database cannot scale to millions of embeddings. How do you fix the bottleneck?
- Your new embedding model has different dimensions from the existing vectors in production. How do you handle the mismatch?
- Your vector search returns irrelevant results despite high similarity scores. How do you fix it?
- You deployed a new embedding model, and search quality crashed overnight. How do you handle embedding drift?
- Your semantic search fails for short queries. How do you improve it?

---

## AI System Design

### End-to-End Design Questions

- Design ChatGPT: Training to Serving (End to End)
- Design a RAG System (Chat with Your Documents)
- Design Memory for a Personal AI Assistant
- Design a Deep Research Agent
- Design a Multi-Agent Customer Support System
- Design an On-Device AI Assistant
- Design a Multimodal Search System (Text, Image, Video)
- Design an LLM Inference Platform (vLLM-as-a-Service)
- Design an LLM Evaluation Platform
- Design a Text-to-Image Generation Service (Midjourney-like)
- Design a Music Generation Service (Suno-like)
- Design a Video Generation Service (Sora-like)
- Design an AI Coding Agent
- Design a code generation and review system
- Design a content moderation system using AI
- Design a real-time AI recommendation system
- Design an AI-powered email assistant
- Design a medical diagnosis assistant using AI
- Design a fraud detection system powered by LLMs
- Design an AI-powered data extraction pipeline from unstructured documents
- Design a personalized learning assistant
- Design an AI system for automated code migration
- Design an AI-powered legal document review system
- Design a conversational AI system with memory across sessions

### Architecture & Trade-off Questions

- How do you design for latency vs quality trade-offs in AI systems?
- How do you implement caching strategies for LLM applications?
- How do you design rate limiting and cost management for AI APIs?
- How do you handle failover and fallback strategies for AI systems?
- How do you design an AI system for high availability and fault tolerance?
- How do you design an AI system that gracefully degrades when the model is unavailable?
- What are the key considerations for multi-region deployment of AI systems?
- Design an AI-powered search engine for an e-commerce platform
- Design an AI gateway/proxy for managing LLM access across an organization
- How do you design a RAG system that handles conflicting information across sources?
- How do you approach capacity planning for an AI system?
- Design a multi-tenant AI chatbot platform where each business gets a custom chatbot
- Design an AI meeting summarizer system for thousands of meetings daily
- Design an AI notification system that prioritizes instead of broadcasting
- Design an AI-powered anomaly detection system for cloud infrastructure
- Design an AI-powered document processing pipeline for financial institutions
- Design an AI dynamic pricing engine
- Design an AI resume screening system that handles 100K applications per week
- Design an AI voice assistant architecture
- Design a multi-agent workflow system where agents collaborate on complex tasks
- Design a real-time AI transcription system for concurrent audio streams
- Design an AI-powered live streaming content moderation system

---

## LLMOps and Production AI

### Conceptual Questions

- How does Prompt Caching work?
- Prefill vs Decode.
- Explain the AI product lifecycle from ideation to production.
- What is LLMOps, and how does it differ from traditional MLOps?
- How do you serve LLMs in production?
- What is model quantization?
- How do you monitor LLM applications in production?
- What is LLM observability?
- What are guardrails for LLMs, and how do you implement them?
- How do you implement content filtering for AI outputs?
- How do you estimate the cost of running an AI-powered feature in production?
- How do you optimize LLM inference costs in production?
- How do you implement A/B testing for LLM systems?
- What is CI/CD for AI applications, and how does it differ from traditional CI/CD?
- How do you version and manage prompts in production?
- What is model versioning, and how do you handle model rollbacks?
- How do you implement rate limiting and throttling for LLM APIs?
- How do you handle model updates and migrations without downtime?
- What is the role of feature flags in AI deployments?
- How do you implement logging and tracing for LLM applications?
- How do you handle PII and sensitive data in LLM inputs and outputs?
- What is a gateway pattern for LLM API management?
- How does Token Streaming work?
- How do you implement streaming responses for real-time AI applications?
- How does vLLM work?
- How does SGLang work?
- What are the key SLAs and metrics for production AI systems (latency, throughput, availability)?
- Cloud vs on-device Model Deployment for AI applications.
- How do you implement fallback strategies when the primary model is unavailable or rate-limited?
- How do you implement structured output from LLMs reliably in production?
- How do you handle long contexts efficiently in production (context compression, prefix caching)?
- What is semantic routing, and how do you implement it in a multi-model system?
- How do you manage secrets and API keys securely in LLM applications?

### Scenario-Based Questions

- Your LLM API has latency spikes during peak hours. How do you stabilize it?
- Your LLM costs are too high in production. How do you reduce costs without degrading quality?
- Your application is hitting LLM provider rate limits during peak hours. How do you handle it?
- Your application depends on one LLM provider. How do you switch providers without downtime?
- Your AI system handles 100 requests/sec but crashes at 5000. How do you scale for concurrent requests?
- A traffic spike brings down your AI system. How do you handle peak traffic?
- One LLM provider outage took down your entire system. How do you eliminate single points of failure?
- Your multi-LLM pipeline fails when one model in the chain breaks. How do you handle orchestration failure?
- Your AI pipeline has zero visibility into which step is failing. How do you add observability?
- You quantized your LLM, but accuracy dropped significantly. How do you minimize quantization loss?
- One failing AI component can take down your entire platform. How do you design graceful degradation?

---

## Evaluation and Testing

### Conceptual Questions

- AI Agent Evaluation.
- LLM Evaluation.
- AI Agent Observability.
- What is evaluation-driven development for AI applications?
- How do you evaluate LLM outputs? What metrics do you use?
- Explain BLEU, ROUGE, and BERTScore. When would you use each?
- What is G-Eval, and how does it use LLMs for evaluation?
- What is LLM-as-a-judge evaluation, and what are its limitations?
- How do you conduct human evaluation for AI systems?
- What is red teaming, and how do you red team an LLM application?
- How do you detect and measure hallucinations in LLM outputs?
- What is adversarial testing for AI systems?
- How do you build a regression test suite for AI applications?
- What are benchmark suites (MMLU, HumanEval, GSM8K), and how do you interpret them?
- How do you evaluate a RAG system end-to-end?
- How do you evaluate the quality of AI agents?
- What is the difference between offline and online evaluation for AI systems?
- How do you measure factual consistency in LLM outputs?
- How do you evaluate multi-turn conversation quality?
- What is the role of golden datasets in AI evaluation?
- How do you implement continuous evaluation for production AI systems?
- How do you evaluate bias in AI model outputs?
- How do you compare two models or prompts in a statistically rigorous way?
- How do you evaluate the robustness of an LLM application across input variations?
- What are the key differences between evaluating traditional ML vs LLM applications?
- How do you set up an evaluation framework from scratch for a new LLM application?

### Scenario-Based Questions

- Your model passes one fairness metric but fails another. How do you handle conflicting audit results?
- Your model was fair at deployment, but became biased 6 months later. How do you monitor continuously?
- An external auditor cannot reproduce your model's results. How do you ensure audit reproducibility?
- How do you structure red teaming for an LLM chatbot before launch?
- How do you red team a multimodal model where text-only safety tests miss cross-modal attacks?

---

## AI Safety, Ethics, and Responsible AI

### Conceptual Questions

- What are hallucinations in LLMs, and how do you mitigate them?
- What is prompt injection, and what are the different types (direct, indirect)?
- How do you implement input and output guardrails for AI systems?
- What is AI alignment, and why is it important?
- How do you detect and mitigate bias in AI systems?
- What are the key data privacy considerations (GDPR, CCPA) when building AI applications?
- How do you handle PII in LLM inputs and outputs?
- What is explainability in AI, and why does it matter?
- What is the difference between interpretability and explainability?
- How do you build trust with users in AI-powered applications?
- What are adversarial attacks on AI systems, and how do you defend against them?
- What is data poisoning, and how can it affect AI models?
- How do you implement content safety filters for AI-generated content?
- What is responsible AI, and what frameworks exist for implementing it?
- How do you handle copyright and intellectual property concerns with AI-generated content?
- What is the EU AI Act, and how does it affect AI engineering?
- How do you implement audit trails and logging for AI decisions?
- What is model card documentation, and why is it important?
- How do you handle misuse and abuse of AI systems in production?
- What is differential privacy, and how can it be applied during model training?
- How would you design an AI incident response plan?
- What is the NIST AI Risk Management Framework (AI RMF)?

### Scenario-Based Questions

- Your healthcare chatbot gives medical diagnoses it should not make. How do you add safety guardrails?
- Your AI system is reproducing copyrighted material verbatim. How do you prevent this?
- Your resume screening AI rejects more female candidates for engineering roles. How do you fix gender bias?
- Your AI model passes bias checks by gender and race separately, but fails for intersectional groups. How do you handle it?
- Your AI denied a loan, and the customer demands a GDPR explanation. How do you provide one?
- A user invokes the right to be forgotten, but their data is in your model weights. How do you comply?
- The EU AI Act may classify your AI system as high-risk. How do you comply?
- Your differentially private model lost significant accuracy. How do you balance privacy and utility?
- One malicious participant is poisoning your federated learning model. How do you defend against it?
- Your AI hiring model uses proxy features for protected attributes. How do you eliminate proxy discrimination?
- Your predictive model creates a feedback loop of biased outcomes. How do you break it?
- Your AI generates fake news images. How do you implement watermarking for AI-generated content?
- Your AI denies a service, and the user has no way to challenge it. How do you design an appeals process?
- An auditor asks why your AI rejected a request 6 months ago, and you have no logs. How do you build audit trails?
- You removed PII, but users were re-identified from anonymized data. How do you prevent re-identification?
- A pre-trained model from an open-source repo may contain a hidden backdoor. How do you detect it?
- Your LLM's training data was deliberately poisoned by an adversary. How do you respond?
- Your AI mental health chatbot gave harmful advice to a user in crisis. How do you mitigate harm?
- Your AI system caused incorrect critical decisions. How do you run a blameless post-mortem?
- Radiologists agree with AI 98% of the time, even when it is wrong. How do you prevent human over-reliance on AI?
- Your content moderation flags normal cultural expressions as offensive in other markets. How do you adapt cross-culturally?
- Your AI training produces massive carbon emissions. How do you reduce environmental impact?

---

## Multimodal AI

### Conceptual Questions

- What are Multimodal AI models, and how do they process different types of data?
- How do vision-language models process images?
- How does CLIP work, and why is it important for multi-modal AI?
- What are the key architectures for multi-modal models?
- How does image generation work with diffusion models (Stable Diffusion, DALL-E, Flux)?
- What is text-to-speech (TTS), and what models are used for it?
- How does speech-to-text (Whisper) work?
- What is multi-modal RAG, and how does it differ from text-only RAG?
- How do you build a system that processes both images and text?
- What are multi-modal embeddings, and how are they used for cross-modal search?
- How do you evaluate multi-modal AI systems?
- What are the challenges of real-time multi-modal AI processing?
- How do you handle video understanding with AI?
- What is visual question answering (VQA)?
- What is document understanding, and how do models parse documents with layouts?
- How do you fine-tune a vision-language model?
- What are the latency and cost considerations for multi-modal AI in production?
- How do you handle multi-modal content moderation?
- What is text-to-video generation, and what are the current state-of-the-art approaches?
- Explain Multimodal Fusion Techniques: Early Fusion vs Late Fusion.

### Scenario-Based Questions

- Your vision-language model generates factually incorrect image descriptions. How do you fix it?
- Your VLM answers single-image questions but fails on multi-page documents. How do you fix it?
- Your multimodal LLM ignores the image and generates descriptions from text alone. How do you fix it?
- Your diffusion model ignores precise control requirements in text prompts. How do you improve controllability?
- Your diffusion model generates sharp but repetitive images. How do you balance quality vs diversity?
- Your diffusion model takes too long per image. How do you speed up sampling?

---

## AI Infrastructure and Scalability

### Conceptual Questions

- How do you improve inference speed in production LLM deployments?
- LLM optimization techniques.
- How do you select GPUs for LLM inference?
- What is model parallelism vs data parallelism in distributed training?
- What is tensor parallelism, and how does it help serve large models?
- What is pipeline parallelism?
- How does continuous batching improve LLM inference throughput?
- What is speculative decoding, and how does it speed up inference?
- What is KV cache, and how do you manage memory for it?
- What is Paged Attention?
- How does GGUF work?
- How do you optimize inference for edge and mobile deployment?
- What is model quantization (INT8, INT4, FP16, BF16), and how does it affect quality?
- How do you implement auto-scaling for AI workloads?
- What is the role of load balancing in AI serving infrastructure?
- How do you manage GPU memory for serving multiple models?
- What is model sharding, and when would you use it?
- How do you implement request queuing and priority scheduling for AI services?
- What are the cost trade-offs between self-hosted and API-based AI inference?
- How do you handle cold start latency for serverless AI deployments?
- How do you implement model caching to reduce redundant computations?
- What is the difference between synchronous and asynchronous inference, and when do you use each?
- What is FSDP (Fully Sharded Data Parallel), and how does it differ from DeepSpeed ZeRO?
- How do you monitor and profile LLM inference in production (TTFT, inter-token latency, GPU utilization)?
- What is model routing at the infrastructure level, and how do you route requests based on complexity and cost?

---

## Coding and Practical Implementation

- Implement a basic RAG pipeline using an embedding model and a vector database.
- Build a simple AI agent with tool use (e.g., calculator, web search).
- Implement semantic search using embeddings and cosine similarity.
- Write code for different text chunking strategies (fixed-size, recursive, semantic).
- Implement a prompt template system with variable substitution.
- Build an evaluation pipeline for LLM outputs using LLM-as-a-judge.
- Implement streaming responses for an LLM API.
- Build a simple vector similarity search from scratch.
- Implement a conversation memory system for a chatbot (sliding window, summary, buffer).
- Write code to detect and handle hallucinations in LLM outputs.
- Implement a retry mechanism with exponential backoff for LLM API calls.
- Write a function calling (tool use) handler for an LLM API.
- Implement a simple re-ranker for search results.
- Build a basic document parser that extracts text from PDFs and splits it into chunks.
- Implement cosine similarity, dot product, and Euclidean distance functions from scratch.
- Write code to implement token counting and context window management.
- Build a simple prompt versioning system.
- Implement a caching layer for LLM responses.
- Implement semantic caching for LLM queries (cache responses for semantically similar queries).
- Write code to detect prompt injection attempts in user inputs.
- Implement an LLM output guardrails system that checks for off-topic responses and PII leakage.
- Build a multi-agent system where agents have different roles and collaborate on a task.

---

## Behavioral and Scenario-Based Questions

- What is AI Engineering, and how does it differ from Machine Learning Engineering?
- How do you decide whether a problem needs AI or a traditional software solution?
- How do you measure the ROI of an AI feature?
- How do you handle hallucinations when they occur in a production AI system?
- How do you decide between using an LLM API vs self-hosting an open-source model?
- How do you manage stakeholder expectations for AI projects?
- Describe your approach to debugging a poor-performing RAG system.
- How do you stay current with the rapidly evolving AI landscape?
- How do you balance innovation with reliability in AI systems?
- Tell me about a challenging AI project you worked on. What was the problem? What approach did you take? What trade-offs did you make? What was the outcome?
- How would you handle a situation where an AI model produces biased or harmful outputs in production?
- How do you approach cost optimization for an AI system that's exceeding budget?
- Describe a time when you had to choose between model accuracy and latency. How did you make the decision?
- How would you handle a situation where your AI system's quality degrades over time?
- How do you communicate AI limitations to non-technical stakeholders?
- How would you approach building an AI feature with limited labeled data?
- Describe your experience working with cross-functional teams on AI projects.
- Where do you see AI engineering heading in the next 3-5 years?
- Why are you interested in this AI engineering role?
- Your PM wants to ship an AI feature with a 15% hallucination rate on edge cases. How do you communicate the risk?
- A non-technical executive asks why your AI feature cannot be 100% accurate. How do you explain LLM limitations?
- You need to choose between a complex agentic system that scores 15% better on benchmarks, or a simpler RAG pipeline that is easier to maintain. How do you decide?

---

## Repo Structure

```
ai-ml-interview-deep-dives/
├── README.md                              # You are here
├── CONTRIBUTING.md                        # How to add questions
├── questions/
│   ├── 01-the-inference-loop/
│   │   └── README.md                      # Full deep-dive breakdown
│   ├── 02-coming-soon/
│   │   └── ...
│   └── ...
```

---

## Contributing

Have a question you've been asked in an interview? Want to improve an existing breakdown?

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

---

## License

MIT — use it, share it, learn from it.

---

<p align="center">
  <strong>If this helped you prep, give it a ⭐</strong><br>
  <em>Built by <a href="https://www.linkedin.com/in/pubalisen/">Pubali Sen</a></em>
</p>
