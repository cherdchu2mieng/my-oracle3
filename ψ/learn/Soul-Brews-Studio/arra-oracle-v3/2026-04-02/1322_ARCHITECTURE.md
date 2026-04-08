# Arra Oracle v3 — Architecture

> MCP Memory Layer with hybrid search, knowledge management, and trace/forum capabilities

## Overview

**arra-oracle-v3** คือ MCP server ที่ทำหน้าที่เป็น persistent memory layer สำหรับ Claude/AI agents — index markdown files จาก `ψ/` brain vault แล้ว expose เป็น tools ผ่าน Model Context Protocol

```
┌─────────────────────────────────────────────────────────────┐
│                   ARRA ORACLE v3 SYSTEM                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  Claude MCP │    │  HTTP API   │    │  Frontend   │     │
│  │  (stdio)    │    │  (Hono)     │    │  (React)    │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘     │
│         └──────────────────┼──────────────────┘            │
│                            │                               │
│                    ┌───────▼───────┐                       │
│                    │  OracleMCP    │                       │
│                    │   index.ts    │                       │
│                    └───────┬───────┘                       │
│              ┌─────────────┼──────────────┐                │
│              │             │              │                 │
│  ┌───────────▼──┐  ┌───────▼────┐  ┌──────▼──────┐         │
│  │  SQLite/FTS5 │  │  LanceDB   │  │  Markdown   │         │
│  │  (keyword)   │  │  (vectors) │  │  ψ/ vault   │         │
│  └──────────────┘  └────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
arra-oracle-v3/
├── src/
│   ├── index.ts              # MCP server entry point (OracleMCPServer class)
│   ├── server.ts             # HTTP API server (Hono)
│   ├── config.ts             # Paths & env vars
│   ├── const.ts              # Constants (MCP_SERVER_NAME, COLLECTION_NAME)
│   ├── types.ts              # Shared types
│   ├── db/
│   │   ├── index.ts          # createDatabase() — Drizzle ORM setup
│   │   ├── schema.ts         # All table definitions
│   │   └── migrations/       # SQL migration files (0000–0006)
│   ├── tools/                # MCP tool handlers
│   │   ├── index.ts          # Barrel export of all tools
│   │   ├── search.ts         # arra_search (hybrid FTS5 + vector)
│   │   ├── learn.ts          # arra_learn (add to knowledge base)
│   │   ├── list.ts           # arra_list
│   │   ├── read.ts           # arra_read
│   │   ├── stats.ts          # arra_stats
│   │   ├── concepts.ts       # arra_concepts
│   │   ├── handoff.ts        # arra_handoff (session handoff)
│   │   ├── inbox.ts          # arra_inbox (list pending handoffs)
│   │   ├── supersede.ts      # arra_supersede ("Nothing is Deleted")
│   │   ├── forum.ts          # arra_thread, arra_threads, arra_thread_read/update
│   │   ├── trace.ts          # arra_trace, arra_trace_list/get/link/unlink/chain
│   │   └── types.ts          # ToolContext, ToolResponse interfaces
│   ├── indexer/              # File scanner & DB indexer
│   │   ├── index.ts          # Main indexer
│   │   ├── collectors.ts     # File collectors by type
│   │   ├── parser.ts         # Frontmatter parser
│   │   ├── storage.ts        # DB write logic
│   │   └── discovery.ts      # File discovery
│   ├── vector/               # Vector DB abstraction layer
│   │   ├── factory.ts        # createVectorStore() factory
│   │   ├── embeddings.ts     # Embedding provider abstraction
│   │   ├── types.ts          # VectorStoreAdapter interface
│   │   └── adapters/         # Implementations
│   │       ├── lancedb.ts    # LanceDB (default)
│   │       ├── sqlite-vec.ts # sqlite-vec
│   │       ├── qdrant.ts     # Qdrant
│   │       ├── chroma-mcp.ts # ChromaDB via MCP
│   │       └── cloudflare-vectorize.ts
│   ├── vault/                # ψ/ vault management
│   │   ├── handler.ts        # getVaultPsiRoot()
│   │   ├── discovery.ts      # Vault file discovery
│   │   └── git.ts            # Git operations
│   ├── forum/                # Thread system types/handlers
│   ├── trace/                # Trace log types/handlers
│   ├── routes/               # HTTP route handlers (Hono)
│   ├── server/               # Server utilities
│   │   ├── context.ts        # Request context
│   │   ├── handlers.ts       # Core business logic
│   │   ├── logging.ts        # DB logging helpers
│   │   └── project-detect.ts # Auto-detect project from cwd
│   └── config/
│       └── tool-groups.ts    # Tool enable/disable config
├── frontend/                 # React dashboard (Vite + TypeScript)
├── oracle-v2/                # (legacy/reference code)
├── docs/                     # Architecture & API docs
└── e2e/                      # Playwright E2E tests
```

## Entry Points

| Entry | Purpose |
|-------|---------|
| `src/index.ts` | MCP server (stdio transport) |
| `src/server.ts` | HTTP API server (port 47778 default) |
| `src/indexer/cli.ts` | CLI indexer (scan vault → DB) |
| `src/cli/index.ts` | CLI client tools |

## Core Abstractions

### OracleMCPServer (src/index.ts)
- ทำ `Database` (bun:sqlite) + `BunSQLiteDatabase` (drizzle-orm)
- ทำ `VectorStoreAdapter` ผ่าน factory
- Register tools ทั้งหมดผ่าน `setupHandlers()`
- Support `--read-only` mode (disable write tools)
- Support `tool-groups` config (disable specific groups)

### ToolContext (src/tools/types.ts)
```typescript
interface ToolContext {
  db: BunSQLiteDatabase<typeof schema>;
  sqlite: Database;
  repoRoot: string;
  vectorStore: VectorStoreAdapter;
  vectorStatus: 'unknown' | 'connected' | 'unavailable';
  version: string;
}
```

### VectorStoreAdapter (src/vector/types.ts)
Abstract interface รองรับหลาย backends:
- **LanceDB** (default) — local, fast
- **sqlite-vec** — embedded SQLite extension
- **Qdrant** — remote vector DB
- **ChromaDB via MCP** — ChromaDB via MCP bridge
- **Cloudflare Vectorize** — cloud

### Hybrid Search Algorithm
1. Sanitize query (remove FTS5 special chars)
2. Run FTS5 keyword search (SQLite)
3. Run vector search (LanceDB/ollama embeddings)
4. Normalize scores: FTS5 = `e^(-0.3 * |rank|)`, Vector = `1 - distance`
5. Merge + deduplicate by document ID
6. Score = 50% FTS + 50% vector + 10% boost if in both
7. Graceful fallback: ถ้า vector unavailable → FTS5-only

## MCP Tools (arra_*)

| Tool | Type | Purpose |
|------|------|---------|
| `____IMPORTANT` | meta | Workflow guide embedded in tool list |
| `arra_search` | read | Hybrid FTS5 + vector search |
| `arra_read` | read | Read full document content |
| `arra_list` | read | Browse all documents |
| `arra_stats` | read | DB statistics |
| `arra_concepts` | read | Topic coverage |
| `arra_learn` | write | Add new pattern/learning |
| `arra_supersede` | write | Mark doc as outdated (Nothing is Deleted) |
| `arra_handoff` | write | Save session context |
| `arra_inbox` | read | List pending handoffs |
| `arra_thread` | write | Start threaded discussion |
| `arra_threads` | read | List threads |
| `arra_thread_read` | read | Read thread messages |
| `arra_thread_update` | write | Update thread |
| `arra_trace` | write | Log discovery session |
| `arra_trace_list` | read | Find past traces |
| `arra_trace_get` | read | Get trace + dig points |
| `arra_trace_link` | write | Chain traces together |
| `arra_trace_unlink` | write | Unlink traces |
| `arra_trace_chain` | read | View full linked chain |

## Database Tables

| Table | Purpose |
|-------|---------|
| `oracle_documents` | Document metadata index |
| `oracle_fts` | FTS5 virtual table (keyword search) |
| `indexing_status` | Background indexing progress |
| `search_log` | Search query history |
| `learn_log` | What was learned & when |
| `document_access` | Access tracking |
| `forum_threads` | Threaded discussions |
| `forum_messages` | Individual messages in threads |
| `trace_log` | Discovery traces with dig points |
| `supersede_log` | Audit trail for "Nothing is Deleted" |
| `activity_log` | User activity tracking |
| `schedule` | Calendar events (per-human) |
| `settings` | Key-value config store |

## Dependencies

| Package | Purpose |
|---------|---------|
| `@modelcontextprotocol/sdk` | MCP server/transport |
| `drizzle-orm` | Type-safe ORM for SQLite |
| `hono` | Lightweight HTTP framework |
| `@lancedb/lancedb` | Default vector store |
| `sqlite-vec` | SQLite vector extension |
| `@qdrant/js-client-rest` | Qdrant client |
| `bun:sqlite` | Native SQLite (Bun runtime) |

**Runtime**: Bun >= 1.2.0

## Configuration

### Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `ORACLE_REPO_ROOT` | `process.cwd()` | ψ/ vault location |
| `ORACLE_READ_ONLY` | `false` | Disable write tools |
| `ORACLE_VECTOR_DB` | `lancedb` | Vector backend |
| `ORACLE_EMBEDDING_PROVIDER` | `ollama` | Embedding provider |
| `ORACLE_EMBEDDING_MODEL` | `bge-m3` | Embedding model |
| `PORT` | `47778` | HTTP server port |

### Embedding Models

| Key | Model | Dim | Note |
|-----|-------|-----|------|
| `bge-m3` | bge-m3 (default) | 1024 | Thai+EN multilingual |
| `nomic` | nomic-embed-text | 768 | Fast |
| `qwen3` | qwen3-embedding | 4096 | Cross-language |
