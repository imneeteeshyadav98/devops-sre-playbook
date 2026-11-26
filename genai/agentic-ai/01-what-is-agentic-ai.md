# Agentic AI — 01: What is Agentic AI?

**Learning status:** CampusX *Agentic AI using LangGraph*, Video 2 ("What is Agentic AI?") discussed; knowledge-check answer **B** was correct (**1/1**). **Practical lab not completed.** Next topic **LangChain vs LangGraph** is still pending confirmation.

## Concept in Hinglish

**Normal LLM call:** input diya, response mila. **Fixed chain:** predefined steps follow hoti hain. **Agentic AI:** system goal ke basis par bounded choices karta hai — reasoning/planning, tools, intermediate observations, state and possible retries — but **only within developer-approved limits**. Being "agentic" does not automatically make a system reliable, and not every task benefits from autonomy.

| Pattern | Example | When to prefer |
|---|---|---|
| Simple model invocation | FAQ wording / intent suggestion | One bounded transformation |
| Fixed sequential/conditional chain | Validate intent → trusted lookup → respond | Deterministic sales workflow |
| Stateful graph | Clarify missing SKU, preserve selection and resume | Multi-turn branching with recovery |
| Agent with tool selection | Choose among approved read-only knowledge tools | When dynamic choice is truly needed |

## Chatout mapping

Customer sequence:

```text
T1: I want ski wax.
T2: Which one is available?
T3: Two pieces.
T4: Great, send the checkout link.
```

A stateful workflow should track `workspace_id`, `conversation_id`, `verified_product_id`, `quantity`, `handoff_requested`, `checkout_readiness`, `tool_results` and action status. The **model may suggest next action**; the backend validates and decides whether that action is authorized.

### Decision boundary

- **Agent may propose:** `lookup_product`, `ask_clarification`, `retrieve_policy`, `request_handoff`.
- **Agent must not independently:** invent prices, declare stock, charge cards, create/claim a checkout without backend verification, send live Instagram messages without required gates.
- **A graph doesn't fix bugs by itself:** missing selection recovery and mixed handoff must be addressed in state design and tests.

## Lab experiment: bounded checkout state machine

Build a tiny fake graph with states `NEED_PRODUCT → VERIFY_PRODUCT → READY_FOR_CHECKOUT / HANDOFF / CLARIFY`. Use mock merchant API responses and no external sends.

## Acceptance checks

1. Turn-4 checkout without product text recovers **trusted** earlier selection.
2. "checkout link bhejo **ya seller se baat karwa do**" preserves handoff information, with appropriate priority.
3. Invalid product/stock returns clarification; no fabricated URL.
4. Retry/replay never duplicates a side-effect (idempotency key and event log).
5. Workspace A's state/data cannot be used to answer workspace B.

## Implementation decision

**Learn and prototype stateful orchestration in an isolated lab only.** Do not replace current Chatout AI-service merely because LangGraph can model a state machine. Require measured improvement against the real regression suite and explicit approval before integration.

**Interview answer:** "An agent uses tools and maintains state to pursue a goal, but in production its autonomy must be scoped by deterministic authorization, typed contracts, timeouts, idempotency, human handoff and observability."
