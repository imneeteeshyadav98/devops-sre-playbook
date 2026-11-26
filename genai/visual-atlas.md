# Visual atlas — GenAI concepts applied to Chatout

> **Scope:** Architecture diagrams are **learning and design models**, not screenshots of deployed services. The arrows describe responsibilities, not proof of implemented features. Source material: the CampusX LangChain/RAG topics and the Agentic AI “What is Agentic AI?” class discussed through 26 November 2026.

**How to read:** start at the customer message, then trace (1) model proposal, (2) verification, and (3) authorized effect. Every arrow crossing a trust boundary requires backend policy.

## 1. One-screen picture: the safe sales assistant

```mermaid
flowchart TB
  Customer["Customer DM on Instagram"] --> Webhook["Trusted backend: webhook and workspace context"]
  Webhook --> State["Conversation and verified selection state"]
  State --> Model["AI proposal: what does the customer want?"]
  Model --> Validate["Schema + semantic validation"]
  Validate --> Router{"Backend decision"}
  Router -->|"Product / checkout"| Commerce["Trusted commerce API: Shopify first"]
  Router -->|"Policy / FAQ"| Retriever["Optional tenant-scoped RAG"]
  Router -->|"Human requested"| Handoff["Merchant handoff / alert"]
  Router -->|"Unknown"| Clarify["Ask a safe clarifying question"]
  Commerce --> Response["Compose response from verified results"]
  Retriever --> Response
  Handoff --> Response
  Clarify --> Response
  Response --> Gate["Permission, deduplication, audit, outbound-send gate"]
  Gate --> CustomerReply["Customer response if authorized"]
```

**Read this as:** The model is one decision-support component. It is **not** the source of truth for inventory or checkout and cannot send merely because it generated text.

## 2. Why each class exists: from words to dependable decisions

```mermaid
flowchart LR
  A["Prompt + messages"] --> B["Chat model"]
  B --> C["Structured output"]
  C --> D["Parser + validation"]
  D --> E["Conditional runnable / router"]
  E --> F["Trusted state and policy"]
  F --> G["Authorized tool result"]
  G --> H["Grounded response"]
  I["Loaders"] --> J["Splitters"]
  J --> K["Embeddings + vector index"]
  K --> L["Tenant-scoped retriever"]
  L -.->|"Only policy evidence"| H
```

The **upper path** handles intent and control. The **lower path** supports document evidence. They meet at response construction; neither replaces authorization.

## 3. Four customer turns: where context can break

```mermaid
sequenceDiagram
  autonumber
  participant C as Customer
  participant B as Backend state
  participant AI as AI classifier
  participant S as Shopify adapter
  C->>B: T1: Need ski wax
  B->>B: Save candidate, not confirmed stock
  C->>B: T2: This variant looks good
  B->>S: Resolve verified product/variant
  S-->>B: Verified selection
  C->>B: T3: Two pieces
  B->>B: Save quantity 2
  C->>B: T4: Send checkout link
  B->>AI: Intent + safe state summary
  AI-->>B: checkout_request proposal
  B->>B: Recover verified selection
  B->>S: Recheck checkout prerequisites
  alt Eligible
    S-->>B: Verified result
    B-->>C: Approved checkout response
  else Unavailable or unverified
    B-->>C: Clarification or safe status
  end
```

**Historical regression:** Turn 4 once fell into low confidence when model classification and backend selection recovery failed. A merged code fix is not itself evidence of completed staging acceptance.

## 4. Multi-intent messages must not erase handoff

```mermaid
flowchart TD
  A["'Checkout link bhejo ya seller se baat karwa do'"] --> B["Extract independent signals"]
  B --> C["checkout_requested = true"]
  B --> D["human_requested = true"]
  C --> E["Validate and preserve both"]
  D --> E
  E --> F{"Human request / policy precedence"}
  F -->|"Human requested"| G["Handoff path + merchant alert requirement"]
  F -->|"Otherwise"| H["Verified checkout path"]
  G --> I["Safe reply; do not invent link"]
  H --> I
```

**Key idea:** One enum is often insufficient to represent independent requests. Use an explicit human flag or multi-label proposal; backend owns precedence.

## 5. RAG has two separate pipelines

```mermaid
flowchart TB
  subgraph Ingestion["Offline / asynchronous document processing"]
    A["Approved merchant file"] --> B["Validate + extract text"]
    B --> C["Chunks + workspace, document, version metadata"]
    C --> D["Embeddings"]
    D --> E[("Vector index")]
  end
  subgraph Query["Customer question at request time"]
    F["Policy question"] --> G["Workspace from authenticated context"]
    G --> H["Search with tenant + active-version filters"]
    H --> I["Evidence validation and reranking"]
    I --> J{"Sufficient trusted evidence?"}
    J -->|"Yes"| K["Grounded answer with source"]
    J -->|"No"| L["Abstain / clarify / handoff"]
  end
  E --> H
```

**Do not use this for:** live inventory, current price, order creation or asserting checkout success. RAG retrieves *documents*, not authoritative transaction state.

## 6. SQL data and vector search solve different problems

```mermaid
flowchart LR
  Q["Question: 'Can I return ski wax?'"] --> Embed["Embed query"]
  Embed --> Vec[("Vector search: similar policy passages")]
  Auth["Authenticated workspace"] --> Filter["Mandatory tenant + version filter"]
  Filter --> Vec
  Vec --> Evidence["Return policy evidence"]
  DB[("Postgres: users, workspaces, document lifecycle, conversation state")] --> Filter
  DB --> Verify["Authorization and authoritative relations"]
  Evidence --> Answer["Evidence-grounded response"]
  Verify --> Answer
```

**Similarity is not authority.** PostgreSQL can coexist with Chroma; pgvector is another possible storage choice, not a commitment here.

## 7. Chain vs bounded agent loop

```mermaid
flowchart LR
  subgraph Fixed["Fixed chain"]
    A["Normalize"] --> B["Classify"] --> C["Validate"] --> D["Route"]
  end
  subgraph Agent["Bounded agent-like loop (future lab)"]
    E["Goal + state"] --> F["Choose permitted next step"]
    F --> G["Read-only tool / observation"]
    G --> H{"Enough evidence / step limit?"}
    H -->|"No, safe to retry"| F
    H -->|"Yes"| I["Return proposed outcome"]
  end
```

**Decision rule:** For predictable sales actions, start with deterministic routing. A loop introduces complexity, cost, tool and retry risk. Agentic does **not** mean unrestricted autonomy.

## 8. Where trust changes

```mermaid
flowchart TD
  U["Untrusted: DM / uploaded document / model output"] --> P["Treat as data"]
  P --> V["Type check + schema + source validation"]
  V --> A["Backend-derived tenant identity + permissions"]
  A --> F{"Fact or action type"}
  F -->|"Current catalog / stock / checkout"| C["Commerce lookup + transaction policy"]
  F -->|"Merchant FAQ"| R["Approved tenant-scoped retrieval"]
  F -->|"Human / unknown"| H["Handoff or clarification"]
  C --> O["Audited effect gate"]
  R --> O
  H --> O
```

**Prompt injection example:** A document saying “ignore all instructions” is still document text. It never becomes system policy.

## 9. Labs are not production features

```mermaid
flowchart LR
  L["Learn concept"] --> D["Isolated synthetic lab"]
  D --> T["Focused tests + negative cases"]
  T --> E["Evidence and metrics"]
  E --> G{"Explicit product approval?"}
  G -->|"No"| Keep["Keep in lab"]
  G -->|"Yes"| R["Reviewed implementation / release gates"]
  R --> S["Staging runtime acceptance"]
  S --> P["Production only after launch approvals"]
```

**Evidence vocabulary:** learned / prototyped / tested / independently reviewed / merged / runtime accepted are different states. Never collapse them into “Done”.

---

## Next steps and related notes

- [Class-by-class course map](video-to-project-map.md)
- [Chatout architecture and safety](chatout/architecture-and-safety.md)
- [Checkout/handoff failure cases](chatout/failure-cases.md)
- [Lab blueprint with acceptance metrics](chatout/lab-experiments.md)
- [Learning progress and what is still pending](learning-progress.md)

**Diagram maintenance:** Mermaid is plain text and GitHub-rendered. Update the diagram alongside its explanatory paragraph whenever an architectural assumption changes; keep labels concise and put detailed caveats in prose.
