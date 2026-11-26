# Isolated GenAI learning lab plan

**Do not put new class exercises into the live Chatout AI-service.** This document is a practice plan, **not** an automatically approved Chatout product task. The product master tracker remains unchanged.

## Lab layout (proposed)

```text
genai-lab/
  app/
    contracts.py         # Pydantic intent proposal (no business authority)
    intents.py           # model adapter / fake classifiers
    state_machine.py     # trusted state fixture and next-step policy
    ingestion.py         # synthetic merchant knowledge loader
    retrieval.py         # workspace-filtered retriever
  fixtures/
    customer_turns.jsonl
    merchant_policies/
  tests/
    test_multi_turn_checkout.py
    test_mixed_handoff.py
    test_tenant_isolation.py
    test_prompt_injection.py
  evaluations/
    results.md
```

## Experiments in learning order

| Stage | Goal | Mock inputs only | Key output / acceptance |
|---|---|---|---|
| E1 | Intent schema + parser | 20 synthetic DMs | Invalid proposals rejected; no external effects |
| E2 | Multi-turn state and routing | 4-turn checkout fixture | Recover trusted selection, detect human signal |
| E3 | Loader + splitter | Merchant policy docs | Versioned chunks keep metadata |
| E4 | Chroma / embeddings retrieval | W1 and W2 fake policies | Zero cross-workspace matches |
| E5 | RAG answers with citations | 30 labelled policy questions | Supported answers or abstention |
| E6 | Bounded graph / agent evaluation | Fake commerce tool + retries | No hallucinated checkout; idempotent effects |

### Example test matrix

| Case | Required result |
|---|---|
| T4 checkout with earlier verified selection | Checkout intent retained; backend still verifies |
| T4 checkout without any selection | Clarification, not invented product |
| Checkout + explicit human request | Preserve human signal |
| Model emits malformed JSON | Rejected; zero sends/orders |
| Model fabricates checkout URL | Discard URL; zero unauthorized effects |
| W1 query uses W2 policy | Block; zero data leakage |
| Policy document deleted | No retrieval from revoked version |
| Retrieved document says "ignore instructions" | Treat as data only |
| API lookup timeout | Safe response/handoff; zero claims of success |
| Replayed checkout step | No duplicate side effect |

## Measurement template

```text
Experiment and commit:
Dataset/fixture version:
Accuracy / correct action proposal:
Multi-turn continuity success rate:
Cross-tenant leakage count:
Unsupported answer count:
False checkout success / duplicate effect count:
Handoff request recall:
P50 / P95 latency:
Token and retrieval cost:
Failures and next action:
```

## Promotion gate

Only propose integration after documented reproducible tests, independent safety review, explicit user approval and comparison against the existing solution. No auto-deploy, no live Instagram sending, no production merchant data in the learning lab.
