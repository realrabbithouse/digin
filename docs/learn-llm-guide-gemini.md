# Q1

This guide is built on one premise: **Companies hire engineers who can solve production bottlenecks, not those who can recite the Transformer paper from memory.**

To get hired in 2025, you need to demonstrate that you understand **cost, latency, and reliability.** This 12-week roadmap splits into two paths after the fundamentals: **Applied** (Product-focused) and **Infra** (Systems-focused).

------

## Phase 1: The Core Foundations (Weeks 1–3)

*Goal: Move past the "ChatBox" and understand how the weights actually turn into words.*

- **Week 1: The Architecture of Deployment.** Don't get stuck in the math of backpropagation. Focus on: Tokenization (BPE), Context Windows, and KV Caching. Understand why longer prompts cost more and get slower.
- **Week 2: The Modern Stack.** Master the Hugging Face `transformers` library and the OpenAI API. Learn to switch between local models (Llama 3, Mistral) and API models.
- **Week 3: Structured Outputs.** This is the #1 skill for Applied Engineers. Learn Pydantic, JSON mode, and Function Calling. If your LLM can't reliably trigger a Python function, it’s useless in a production pipeline.

------

## Phase 2: RAG & Data Engineering (Weeks 4–6)

*Goal: Connect the LLM to private data. This is where 90% of LLM jobs live.*

- **Week 4: The Vector Basics.** Learn embeddings and Vector Databases (Pinecone, Milvus, or Qdrant). Build a "Naive RAG" pipeline.
- **Week 5: The "Hiring" Differentiator (Advanced RAG).** Basic RAG fails in production. You must learn:
  - **Query Transformation:** HyDE (Hypothetical Document Embeddings).
  - **Reranking:** Using Cohere or BGE-reranker to refine results.
  - **Hybrid Search:** Combining Keyword (BM25) with Semantic search.
- **Week 6: Evaluation (The "Gold" Skill).** Companies are desperate for people who can prove their LLM works. Learn **RAGAS** or **DeepEval**. Learn to build "Golden Datasets" to benchmark your system.

------

## Phase 3: Choose Your Path (Weeks 7–10)

Choose **Path A** if you want to build products/features. Choose **Path B** if you want to manage GPUs and scale systems.

### Path A: Applied LLM Engineer

- **Week 7-8: Agentic Workflows.** Move beyond LangChain tutorials. Build an agentic system using **LangGraph** or **CrewAI** that can loop, self-correct, and use tools.
- **Week 9-10: Fine-Tuning.** Don't fine-tune for knowledge; fine-tune for **format and style**. Use **LoRA/QLoRA** on a local GPU (or Unsloth/Predibase) to make a small model (Llama 3 8B) perform like a big one for a specific task.

### Path B: LLM Infra Engineer

- **Week 7-8: Inference Optimization.** Master **vLLM** and **TGI (Text Generation Inference)**. Understand PagedAttention. Learn how to host a model so it handles 100 concurrent users without crashing.
- **Week 9-10: Quantization & Hardware.** Learn the difference between AWQ, GPTQ, and GGUF. Understand GPU memory (VRAM) requirements. Set up a multi-GPU cluster using Docker and Kubernetes (KServe).

------

## Phase 4: The "Hire Me" Portfolio (Weeks 11–12)

*Goal: Stop learning. Start shipping.*

Do not build another "PDF Chatbot." Instead, build one of these:

1. **Applied:** An automated Customer Support agent that accesses a real SQL database, uses a reranker, and has a 95%+ score on an Eval benchmark you created.
2. **Infra:** A benchmarking dashboard that compares the tokens-per-second and latency of 3 different quantization methods on the same model across different hardware.

### Comparison of Roles

| **Feature**        | **Applied Engineer**          | **Infra Engineer**           |
| ------------------ | ----------------------------- | ---------------------------- |
| **Primary Tool**   | LangGraph, LlamaIndex, Python | Kubernetes, vLLM, CUDA, C++  |
| **Success Metric** | Accuracy & User Satisfaction  | Latency & Cost-per-1k-tokens |
| **Focus**          | Prompting, Agents, RAG        | Scaling, GPU Orchestration   |

------

## How to avoid "Tutorial Hell"

- **The 20/80 Rule:** Spend 20% of your time watching/reading and 80% coding. If you watch a 10-minute video on Reranking, spend 40 minutes implementing it.
- **Break things on purpose:** Change a parameter in your vector search until the results become garbage. Understand *why* it broke.
- **Read the Logs:** Look at the raw JSON exchanged between your app and the LLM. Most "magic" happens in the hidden prompts of libraries like LangChain.

**Would you like me to create a specific 1-week deep dive into one of these topics, such as Advanced RAG or vLLM deployment?**

# Q2

To get hired as an LLM Applied or Infra Engineer in 2025, you need a mix of deep theory and production-grade implementation skills. This guide provides the highest-ROI resources for each stage of your journey.

------

## Stage 1: The Transformer Foundation (Week 1–2)

*Goal: Understand the "why" and "how" of the architecture powering every modern model.*

- **The "Gold Standard" Video:** [Let’s build GPT: from scratch, in code](https://www.youtube.com/watch?v=kCc8FmEb1nY) by Andrej Karpathy. The best way to understand the architecture is to code it using PyTorch.
- **The Visual Guide:** [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) by Jay Alammar. Essential for visualizing Multi-Head Attention.
- **The "Must-Read" Paper:** [Attention Is All You Need](https://arxiv.org/abs/1706.03762). Focus on the encoder-decoder interaction.
- **Interactive Learning:** [Transformer Explainer](https://poloclub.github.io/transformer-explainer/). A 3D interactive visualization of how tokens flow through the model.

## Stage 2: RAG & Data Engineering (Week 3–4)

*Goal: Build systems that connect LLMs to private data with high accuracy.*

- **Comprehensive Course:** [DeepLearning.AI: Retrieval Augmented Generation for Production](https://www.deeplearning.ai/short-courses/). Quick, high-impact tutorials on vector databases.
- **Advanced Techniques:** [Pinecone Learning Center: RAG Strategies](https://www.pinecone.io/learn/). Learn about hybrid search, reranking, and self-querying.
- **Evaluation Frameworks:** [Ragas Documentation](https://docs.ragas.io/) and [DeepEval (GitHub)](https://github.com/confident-ai/deepeval). Learn how to measure "Faithfulness" and "Relevancy" rather than just "vibes."
- **Vector DB Deep Dive:** [Vector Database 101](https://www.google.com/search?q=https://zilliz.com/learn/vector-database-fundamentals) by Zilliz/Milvus.

## Stage 3: Agents & Orchestration (Week 5–6)

*Goal: Move from "one-off chats" to autonomous workflows that use tools (APIs/Code).*

- **The Frameworks:** * **LangGraph:** [LangGraph Academy](https://academy.langchain.com/). Focus on cyclic graphs and state management (essential for production).
  - **CrewAI:** [Multi-AI Agent Systems Course](https://www.deeplearning.ai/short-courses/multi-ai-agent-systems-with-crewai/). Great for role-playing agent architectures.
- **Small & Efficient Agents:** [Hugging Face `smolagents`](https://www.google.com/search?q=[https://github.com/huggingface/smolagents](https://github.com/huggingface/smolagents)). A 2025 favorite for building lightweight, code-writing agents.
- **Conceptual Deep Dive:** [LLM Powered Assistants](https://lilianweng.github.io/posts/2023-06-23-agent/) by Lilian Weng (OpenAI). The "Bible" of agentic reasoning.

## Stage 4: Fine-Tuning & Alignment (Week 7–8)

*Goal: Adapt models for specific domains or styles while managing costs.*

- **Hands-on Library:** [Unsloth AI](https://github.com/unslothai/unsloth). The fastest way to fine-tune Llama-3/Qwen models on a single GPU.
- **PEFT Mastery:** [Hugging Face PEFT Guide](https://huggingface.co/docs/peft/index). Learn LoRA and QLoRA—the industry standards for parameter-efficient tuning.
- **Instruction Tuning:** [Axolotl GitHub Repo](https://www.google.com/search?q=https://github.com/OpenAccess-AI-Collective/axolotl). The most popular configuration-based tool for professional fine-tuning.
- **Alignment:** [RLHF to DPO](https://huggingface.co/blog/dpo-trl). Learn Direct Preference Optimization (DPO), which has largely replaced complex RLHF in 2025.

## Stage 5: Inference & LLM Infra (Week 9–10)

*Goal: Deploy models that are fast, cheap, and scalable.*

- **Inference Engines:** * **vLLM:** [vLLM Documentation](https://docs.vllm.ai/). Learn PagedAttention and continuous batching.
  - **TGI:** [Text Generation Inference](https://huggingface.co/docs/text-generation-inference/index) by Hugging Face.
- **Quantization:** [Maarten Grootendorst’s Guide to Quantization](https://www.maartengrootendorst.com/blog/quantization/). Understand GGUF, AWQ, and EXL2.
- **The "Open Book":** [Stas Bekman’s ML Engineering Open Book](https://github.com/stas00/ml-engineering). A raw, high-level look at GPU memory, networking, and scaling.

## Stage 6: Interview Prep & Portfolio (Week 11–12)

*Goal: Prove your skills to recruiters.*

- **Question Bank:** [63 Must-Know LLM Interview Questions](https://github.com/Devinterview-io/llms-interview-questions). Covers everything from KV Caching to Beam Search.
- **System Design:** [Applied LLMs in Production](https://www.google.com/search?q=https://github.com/ml-engineering-guide/applied-llm). Real-world case studies from Uber, Netflix, and OpenAI.
- **Project Idea:** Build a **"Personal Research Agent"** that:
  1. Scrapes Arxiv for papers.
  2. Uses RAG to store them in a Vector DB.
  3. Uses LangGraph to summarize and find contradictions between papers.
  4. Deployed via vLLM in a Docker container.

------

**Would you like me to create a customized 12-week study calendar based on these resources for a specific job track (Applied vs. Infra)?**

# APPENDIX

链接：https://gemini.google.com/share/9d488fad2209