# Generative AI learning notes — applied to Chatout

> **Learning repository, not production code.** Notes reflect what I studied and discussed through **26 Nov 2026**. Examples are simplified; they do **not** prove a capability is implemented or deployed in Chatout.

## Why these notes exist

I am learning Generative AI, LangChain, RAG and Agentic AI with the goal of designing a **reliable Instagram-to-commerce assistant (Chatout)**. A merchant may sell through Shopify first, then WooCommerce or a custom store. The most important principle: **LLM suggests intent and language; authoritative systems own product facts, inventory, checkout and sends.**

Each lesson follows: **Concept → Chatout mapping → real failure/use case → isolated experiment → acceptance checks → implementation decision**. Notes are Hinglish-friendly but preserve the standard engineering terminology used in interviews.

## Learning index

| Track | Lesson | Status |
|---|---|---|
| LangChain | [01 Introduction / LLMs](langchain/01-introduction-llms.md) | Discussed |
| LangChain | [02 Components and models](langchain/02-components-and-models.md) | Discussed |
| LangChain | [03 Prompts and messages](langchain/03-prompts-and-messages.md) | Discussed |
| LangChain | [04 Structured outputs](langchain/04-structured-outputs.md) | Discussed |
| LangChain | [05 Output parsers](langchain/05-output-parsers.md) | Discussed |
| LangChain | [06 Runnables and chains](langchain/06-runnables-and-chains.md) | Discussed |
| LangChain | [07 Document loaders](langchain/07-document-loaders.md) | Discussed |
| LangChain | [08 Text splitters](langchain/08-text-splitters.md) | Discussed |
| LangChain | [09 Embeddings and vector stores](langchain/09-embeddings-and-vector-stores.md) | Discussed |
| LangChain | [10 Retrievers](langchain/10-retrievers.md) | Discussed |
| LangChain | [11 RAG and fine-tuning distinction](langchain/11-rag.md) | Discussed |
| Agentic AI | [01 What is Agentic AI?](agentic-ai/01-what-is-agentic-ai.md) | Video 2 knowledge check passed; lab pending |

**Next planned lesson:** LangChain vs LangGraph. This is **not marked completed**. LangGraph introductory concepts were discussed, not deployed.

## Apply it to the product

- [Chatout architecture and safety boundaries](chatout/architecture-and-safety.md)
- [Real failure cases mapped to GenAI topics](chatout/failure-cases.md)
- [Isolated GenAI lab experiments and test matrix](chatout/lab-experiments.md)
- [GenAI/SRE interview preparation](chatout/interview-preparation.md)
- [Learning progress / quiz evidence](learning-progress.md)

## Hard engineering rules

1. **No GenAI learning code directly into production AI-service.** Start in a separate lab/microservice workspace; promotion requires explicit approval.
2. Never let RAG or LLM memory decide **live product price, stock, delivery promise, order state or checkout success**. Verify through a trusted commerce API/backend.
3. Any tool/action must respect **workspace identity, authorization, idempotency, validation, audit and outbound-send approval**.
4. Redact customer PII and secrets in learning examples. Use synthetic merchant documents for retrieval experiments.
5. Do not count a lecture as a completed engineering feature. **Learned ≠ implemented ≠ tested ≠ merged ≠ runtime accepted**.
6. Chatout master Excel tracker remains the product source of truth; this learning repo is not a second product backlog.

## Reusable class note checklist

- Explain the concept in simple English/Hinglish.
- Draw where it would sit in Chatout.
- Link one actual regression or business use case.
- Define a safe toy experiment with mock data.
- Add positive, negative, cross-workspace and security cases.
- State **use now / prototype later / not needed** and why.
