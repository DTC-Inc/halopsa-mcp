# HaloPSA MCP Server

Rust-based MCP server for HaloPSA. Enables AI assistants to interact with HaloPSA tickets, actions, and workflows through the Model Context Protocol.

## Architecture

Cargo workspace with 4 crates:

| Crate | Type | Purpose |
|-------|------|---------|
| `hmcp-common` | library | Shared types, HaloPSA API client, DB traits, ticket chunking |
| `hmcp-db-postgres` | library | Postgres + pgvector backend (auth tokens + semantic embeddings) |
| `hmcp-server` | binary | MCP server (SSE + Streamable HTTP transport, OAuth 2.1 relay, tools) |
| `hmcp-embedder` | binary | Embedding sidecar (fastembed/OpenAI/Ollama, job queue worker) |

## Key Design Decisions

- **OAuth relay**: MCP server relays OAuth to HaloPSA's own auth. Users authenticate directly with HaloPSA, so all API calls carry their real permissions.
- **Permission-respecting semantic search**: Embeddings indexed via service account, but results filtered per-user by checking each ticket against the user's HaloPSA token.
- **Workflow-aware actions**: Available ticket actions determined by walking the ticket's workflow definition and filtering to the current step.
- **Modeled after bookstack-mcp**: Same patterns for MCP protocol, SSE transport, DB traits, embedding pipeline.

## Building

```bash
cargo build --release
```

## Environment

See `.env.example` for all configuration options.

## Ownership and rules

Moved from `bees-roadhouse` to `DTC-Inc` on 2026-10-22 (Nate: "halopsa-mcp should go to apis and dtc"). Apis owns it. The DTC KB governs it: the DevOps book's *Approved Languages & Stack*, *Branching Strategy* and PR process, never the Roadhouse DevOps book.

- **Rust**, on this workspace. It's DTC's primary backend language (KB page 1263). Changing it needs a case put to the CTO, per that page.
- **Single `main`.** No staging tier exists yet, and the KB's rule is "if you are not sure, use single `main`." Move to `development` + `release` when a staging deployment does.
- **Tools** follow the `halopsa-api` and `halopsa-workflows` skills, one recipe per tool. Writes to Halo go draft, approve, then send.
- Decision record: D-055 in the Nate and Apis working journal (kb.dtctoday.com, book 4137).
