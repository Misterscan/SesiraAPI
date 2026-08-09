# Memory and retrieval

Short-term memory is a bounded array of recent turns. Long-term memory is a Sesi document database collection separated by namespace. `memory_bank` composes both.

Durable records contain `id`, `content`, `metadata`, `created_at`, and `updated_at`. Calling `remember` with an existing ID updates it while retaining its creation time.

Retrieval performs local normalization, unique-term coverage scoring, and an exact-phrase bonus. This makes behavior deterministic, offline, inexpensive, and easy to test. It is intentionally not presented as semantic equivalence. Applications that need embedding retrieval can place an embedding/vector service behind a normal Sesira tool and store its references in metadata.

Pass a `scorer(doc, query)` function in retrieval options to replace the built-in ranking. This provides a dependency-free extension point for embeddings, hybrid search, or domain-specific scoring.

Metadata filters require exact key/value matches:

```sesi
let hits = "refund" | Sesira.recall(store, {
  "limit": 5,
  "min_score": 0.1,
  "filter": {"tenant": "acme"}
})
```

For sensitive memory, pass a password to `long_memory`. Sesi encrypts that database at rest. Namespace separation is organizational, not a security boundary; use separate encrypted databases when isolation matters.
