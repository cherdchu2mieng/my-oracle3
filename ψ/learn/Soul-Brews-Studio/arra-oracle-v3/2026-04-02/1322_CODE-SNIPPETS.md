# Arra Oracle v3 — Code Snippets

## MCP Server Bootstrap (src/index.ts)

```typescript
class OracleMCPServer {
  constructor(options: { readOnly?: boolean; toolGroups?: ToolGroupConfig } = {}) {
    this.readOnly = options.readOnly ?? false;
    this.repoRoot = process.env.ORACLE_REPO_ROOT || process.cwd();
    
    // Load tool group config (disable/enable tool groups)
    const groupConfig = options.toolGroups ?? loadToolGroupConfig(this.repoRoot);
    this.disabledTools = getDisabledTools(groupConfig);
    
    // Create vector store (LanceDB by default)
    this.vectorStore = createVectorStore({ dataPath: CHROMADB_DIR });
    
    // Create SQLite DB via Drizzle ORM
    const { sqlite, db } = createDatabase(DB_PATH);
    this.db = db;
    
    this.setupHandlers();
  }
}

// Entry point
const readOnly = process.env.ORACLE_READ_ONLY === 'true' || process.argv.includes('--read-only');
const server = new OracleMCPServer({ readOnly });
await server.run();
```

---

## ToolContext Pattern

```typescript
// ToolContext passed to every handler
interface ToolContext {
  db: BunSQLiteDatabase<typeof schema>;
  sqlite: Database;
  repoRoot: string;
  vectorStore: VectorStoreAdapter;
  vectorStatus: 'unknown' | 'connected' | 'unavailable';
  version: string;
}

// ใช้ใน handler
export async function handleSearch(ctx: ToolContext, input: OracleSearchInput): Promise<ToolResponse> {
  const { db, sqlite, vectorStore, vectorStatus } = ctx;
  // ...
}
```

---

## Vector Store Factory (src/vector/factory.ts)

```typescript
// Default: LanceDB + Ollama + bge-m3
export function createVectorStore(config: VectorStoreConfig = {}): VectorStoreAdapter {
  const type = config.type || process.env.ORACLE_VECTOR_DB || 'lancedb';
  
  switch (type) {
    case 'lancedb': {
      const embedder = createEmbeddingProvider('ollama', process.env.ORACLE_EMBEDDING_MODEL);
      return new LanceDBAdapter(collectionName, dbPath, embedder);
    }
    case 'sqlite-vec': { ... }
    case 'qdrant': { ... }
    case 'cloudflare-vectorize': { ... }
    default: // chroma
      return new ChromaMcpAdapter(collectionName, dataPath, pythonVersion);
  }
}

// Multi-model registry (bge-m3 default, nomic, qwen3)
export function getVectorStoreByModel(model?: string): VectorStoreAdapter {
  const key = model && models[model] ? model : 'bge-m3';
  // Cache instances by model key
  if (!modelStoreCache.has(key)) {
    const store = createVectorStore({ type: 'lancedb', embeddingModel: preset.model });
    modelStoreCache.set(key, store);
  }
  return modelStoreCache.get(key)!;
}
```

---

## Hybrid Search Logic (src/tools/search.ts)

```typescript
// FTS5 query sanitization
export function sanitizeFtsQuery(query: string): string {
  return query
    .replace(/[?*+\-()^~"':.\/]/g, ' ')
    .replace(/\s+/g, ' ')
    .trim();
}

// FTS5 score normalization: exponential decay
export function normalizeFtsScore(rank: number): number {
  return Math.exp(-0.3 * Math.abs(rank)); // → [0, 1]
}

// Hybrid combine: 50% FTS + 50% vector + 10% boost if in both
export function combineResults(ftsResults: SearchResult[], vectorResults: SearchResult[]): SearchResult[] {
  const combined = new Map<string, SearchResult>();
  
  for (const r of ftsResults) {
    combined.set(r.id, { ...r, score: r.score * 0.5 });
  }
  for (const r of vectorResults) {
    if (combined.has(r.id)) {
      combined.get(r.id)!.score += r.score * 0.5 + 0.1; // boost
    } else {
      combined.set(r.id, { ...r, score: r.score * 0.5 });
    }
  }
  
  return [...combined.values()].sort((a, b) => b.score - a.score);
}
```

---

## Database Schema Highlights (src/db/schema.ts)

```typescript
// Main document index
export const oracleDocuments = sqliteTable('oracle_documents', {
  id: text('id').primaryKey(),
  type: text('type').notNull(),           // principle | learning | pattern | retro
  sourceFile: text('source_file').notNull(),
  concepts: text('concepts').notNull(),    // JSON array
  supersededBy: text('superseded_by'),     // "Nothing is Deleted"
  supersededAt: integer('superseded_at'),
  origin: text('origin'),                  // 'mother' | 'arthur' | 'human' | null
  project: text('project'),                // 'github.com/owner/repo'
  createdBy: text('created_by'),           // 'indexer' | 'arra_learn' | 'manual'
});

// Trace log — discovery sessions with dig points
export const traceLog = sqliteTable('trace_log', {
  traceId: text('trace_id').unique().notNull(),
  query: text('query').notNull(),
  foundFiles: text('found_files'),          // JSON: [{path, type, matchReason}]
  foundCommits: text('found_commits'),      // JSON: [{hash, date, message}]
  foundIssues: text('found_issues'),        // JSON: [{number, title, url}]
  prevTraceId: text('prev_trace_id'),       // ← linked list
  nextTraceId: text('next_trace_id'),       // → linked list
  parentTraceId: text('parent_trace_id'),  // ↑ tree hierarchy
  depth: integer('depth').default(0),
  status: text('status').default('raw'),    // raw → reviewed → distilled
  awakening: text('awakening'),             // extracted insight
  distilledToId: text('distilled_to_id'),  // learning ID if promoted
});

// Schedule — shared calendar per human
export const schedule = sqliteTable('schedule', {
  date: text('date').notNull(),         // YYYY-MM-DD
  time: text('time'),                   // HH:MM or "TBD"
  event: text('event').notNull(),
  recurring: text('recurring'),         // null | 'daily' | 'weekly' | 'monthly'
  status: text('status').default('pending'), // pending | done | cancelled
});
```

---

## arra_learn Handler Pattern (src/tools/learn.ts)

```typescript
// Normalize project input to canonical form
export function normalizeProject(input?: string): string | null {
  if (!input) return null;
  // Accepts: "github.com/owner/repo", "owner/repo", GitHub URLs, local ghq paths
  // Returns: "github.com/owner/repo"
}

export async function handleLearn(ctx: ToolContext, input: OracleLearnInput): Promise<ToolResponse> {
  const { db, repoRoot } = ctx;
  const vaultRoot = getVaultPsiRoot(repoRoot);
  
  // 1. Create markdown file in ψ/memory/learnings/
  const fileName = `${Date.now()}_${slugify(input.pattern.slice(0, 40))}.md`;
  const filePath = path.join(vaultRoot, 'memory/learnings', fileName);
  fs.writeFileSync(filePath, `# Learning\n\n${input.pattern}`);
  
  // 2. Index into oracle_documents + oracle_fts (FTS5)
  const docId = generateId();
  await db.insert(oracleDocuments).values({
    id: docId,
    type: 'learning',
    sourceFile: filePath,
    concepts: JSON.stringify(coerceConcepts(input.concepts)),
    project: normalizeProject(input.project),
    createdBy: 'arra_learn',
    // ...timestamps
  });
  
  return { content: [{ type: 'text', text: `Learned: ${docId}` }] };
}
```

---

## Project Auto-Detection (src/server/project-detect.ts)

```typescript
// Auto-detect project from cwd — used by arra_search({ cwd }) and arra_learn({ project })
export function detectProject(cwd: string): string | null {
  // Follows symlinks → resolves to ghq path
  // Returns "github.com/owner/repo" or null
}
```

---

## Read-Only Mode Filter

```typescript
// Write tools disabled in read-only mode
const WRITE_TOOLS = ['arra_learn', 'arra_thread', 'arra_thread_update', 'arra_trace', 'arra_supersede', 'arra_handoff'];

// In ListTools handler:
let tools = allTools.filter(t => !this.disabledTools.has(t.name));
if (this.readOnly) {
  tools = tools.filter(t => !WRITE_TOOLS.includes(t.name));
}
```
