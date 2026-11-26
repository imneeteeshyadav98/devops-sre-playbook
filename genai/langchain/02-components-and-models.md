# 02 — LangChain components and models

## Concept

Think in replaceable blocks: **chat model** (understand/generate), **embeddings model** (text → vector), **prompt template**, **output parser/schema**, **retriever**, **tools**, and **orchestration**. A chat model and an embeddings model have different outputs and jobs. A framework is an integration library, **not** a guarantee of safer AI decisions.

```text
Customer DM
   | -> message/state handling -> chat model -> validated intent proposal
   | -> (optional) retriever + merchant FAQ -> grounded answer
   | -> authoritative product/checkout API -> verified business result
```

## Chatout mapping

- Chat model: understand "same item, 2 pieces" in conversation context.
- Embedding model: find relevant merchant shipping-policy text even when words differ.
- Retriever: enforce workspace scope and pick relevant document snippets.
- Typed schema/parser: prevent malformed action data.
- Deterministic backend: own verified Shopify facts and action permissions.

This is a **conceptual design map**; it does not establish that each LangChain component exists inside the current service.

## Mistake to avoid

Embedding similarity is **not** inventory availability. A stored chunk saying "size M available" can be obsolete even if it is the closest semantic match.

## Safe lab

Build two minimal adapters: `fake_chat_model(prompt) -> IntentProposal` and `fake_embedder(text) -> vector`. Mock commerce lookup separately. Swap the fake model implementation without changing the authorization or checkout layer.

## Acceptance checks

- Model failure yields a controlled error or handoff, never an action.
- Embeddings/retrieval cannot bypass commerce verification.
- Tenant/workspace ID is carried through all reads.
- Unit tests mock the model and verify request/response shapes.

## Decision

**Prototype behind interfaces** in the separate learning lab. Avoid installing a framework merely to wrap one static prompt.
