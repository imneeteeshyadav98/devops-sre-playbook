# 09 — Embeddings, vector stores and Chroma

## Concept

An **embedding** maps text to a numerical vector such that related meaning is often near in vector space. A **vector store/index** supports nearest-neighbor search. A general relational database (e.g., PostgreSQL) and a vector store serve different tasks; one does not make the other obsolete. `pgvector` can support vector retrieval in Postgres, while **Chroma** is a useful local lab prototype.

```text
"Delivery kab hogi?" -> query embedding
Merchant policy chunks -> precomputed embeddings
Similarity + tenant/document filters -> retrieved source passages
```

Similarity ≠ truth, business authorization, current stock or guaranteed answer.

## Chatout boundary

Store stable merchant FAQ/policy content (if approved), but retrieve real-time **price / inventory / checkout / order details** from a trusted commerce integration. Distinguish semantic similarity from exact SKU matching.

## Minimal lab record

```json
{
  "id": "w1-doc7-chunk2",
  "text": "Returns accepted within 7 days ...",
  "metadata": {"workspace_id":"w1","document_id":"doc7","version":2,"section":"returns"}
}
```

## Safe experiment

Create two fake merchant workspaces with intentionally similar policies but opposite return windows. Index into local Chroma (or test doubles). Search from W1 and insist W2 results are never returned.

## Acceptance checks

- `workspace_id` is applied as a mandatory server-derived filter.
- Re-index updates old chunks; delete removes all obsolete vectors.
- Retrieval hit-rate and wrong-tenant result count are logged.
- No LLM answer can override live Shopify authority.

## Decision

**Prototype** embeddings/Chroma locally. Choose production storage only after measuring scale, isolation, deletion, costs and operations.
