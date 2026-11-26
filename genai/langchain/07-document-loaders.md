# 07 — Document loaders and ingestion

## Concept

A document loader reads content from a **source** (PDF, TXT, HTML, CSV, etc.) and represents it as text plus metadata. Parsing is only the first step; it does not imply the content is correct, safe, or current.

```text
Merchant upload -> file validation -> loader -> extracted text + metadata
                -> review/sanitization -> splitter -> embeddings -> index
```

## Chatout use case

Merchant uploads return policy, sizing chart, product care instructions or FAQ. The assistant should be able to answer *"return kitne din mein hota hai?"* from the authorized merchant policy, not global model memory.

## Metadata worth keeping

`workspace_id`, `document_id`, `file_name`, `source_type`, `version`, `uploaded_at`, `page_number/section`, `language`, `policy_effective_date`, `approval_status`, `checksum`, `visibility`. Generate IDs server-side. **Never accept workspace ID from document text as authority.**

## Scope and security

- Reject unsupported file types / oversized uploads, and scan untrusted files.
- Do not store plaintext secrets, personal contact details or customer chats as vector data in this lab.
- Track document deletion and index invalidation; a removed policy must not remain retrievable.
- Preserve origin/page metadata to support evidence and citations.

## Safe experiment / acceptance

Upload two synthetic merchants' distinct policies. Extract text and page references, then assert every chunk retains correct tenant and document version. Verify corrupt PDF handling and deletion invalidation without affecting other merchants.

## Decision

**Optional future merchant knowledge module** in separate GenAI lab. It is not necessary for authoritative Shopify facts and must not be treated as already built.
