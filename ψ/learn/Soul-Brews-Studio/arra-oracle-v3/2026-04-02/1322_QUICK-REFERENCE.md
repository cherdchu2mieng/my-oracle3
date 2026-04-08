# Arra Oracle v3 — Quick Reference

## What Is This?

**arra-oracle-v3** คือ MCP server ที่ทำหน้าที่เป็น persistent memory layer สำหรับ AI agents (Claude ฯลฯ)

ทำงานยังไง:
1. Index markdown files จาก `ψ/` brain vault ลง SQLite (FTS5) + vector DB (LanceDB)
2. Expose เป็น MCP tools (`arra_*`) ให้ Claude ใช้ค้นหา, เรียนรู้, จำ context ข้าม session
3. มี HTTP API server สำหรับ dashboard/frontend

**Package**: `arra-oracle-v2` (package name ยังเก่า) | **Version**: 0.5.0

---

## Installation

```bash
# Clone
git clone https://github.com/Soul-Brews-Studio/arra-oracle-v3

# Install deps (requires Bun >= 1.2.0)
bun install

# Setup DB
bun run db:migrate

# Run MCP server (stdio)
bun src/index.ts

# Run HTTP server
bun src/server.ts  # port 47778
```

---

## MCP Configuration

```json
{
  "mcpServers": {
    "oracle-v2": {
      "command": "bun",
      "args": ["/path/to/arra-oracle-v3/src/index.ts"],
      "env": {
        "ORACLE_REPO_ROOT": "/path/to/your/psi-vault",
        "ORACLE_VECTOR_DB": "lancedb",
        "ORACLE_EMBEDDING_PROVIDER": "ollama",
        "ORACLE_EMBEDDING_MODEL": "bge-m3"
      }
    }
  }
}
```

**Read-only mode** (สำหรับ share oracle ข้ามหลาย agents):
```json
"env": { "ORACLE_READ_ONLY": "true" }
```

---

## Key Tools

### arra_search — ค้นหา
```
arra_search(query: string, type?: 'all'|'learning'|'principle'|'pattern'|'retro', mode?: 'hybrid'|'fts'|'vector', limit?: 5, project?: string)
```
- Hybrid = FTS5 keyword + vector semantic (50/50 + 10% boost ถ้า match ทั้งคู่)
- Graceful fallback: ถ้า vector unavailable → FTS5-only

### arra_learn — บันทึก learning
```
arra_learn(pattern: string, source?: string, concepts?: string[], project?: string)
```
- Creates markdown ใน `ψ/memory/learnings/`
- Indexes ลง DB ทันที

### arra_read — อ่าน document
```
arra_read(id: string)
```

### arra_handoff — ส่ง context ข้าม session
```
arra_handoff(content: string)
```

### arra_inbox — รับ handoff
```
arra_inbox()  → list pending handoffs
```

### arra_supersede — "Nothing is Deleted"
```
arra_supersede(oldId: string, newId: string, reason?: string)
```
- ไม่ลบ document เก่า แค่ mark เป็น superseded
- มี `supersede_log` เก็บ audit trail

### arra_thread / arra_threads — threaded discussions
```
arra_thread(title: string, message: string)  → สร้าง thread ใหม่
arra_threads()  → list all threads
arra_thread_read(threadId: number)  → อ่าน messages
arra_thread_update(threadId, message, status?)  → reply
```

### arra_trace — log discovery session
```
arra_trace(query: string, foundFiles?: [], foundCommits?: [], ...)
arra_trace_list()  → หา past traces
arra_trace_get(id)  → explore dig points
arra_trace_link(prevId, nextId)  → chain traces
arra_trace_chain(id)  → view full linked chain
```

---

## HTTP API Endpoints (port 47778)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/health` | GET | Health check |
| `/search` | GET | Keyword search |
| `/list` | GET | Browse documents |
| `/stats` | GET | DB statistics |
| `/learn` | POST | Add pattern |
| `/file` | GET | Read file content |
| `/api/graph` | GET | Knowledge graph |
| `/api/dashboard` | GET | Dashboard data |
| `/api/schedule` | GET/POST | Calendar events |
| `/api/forum/threads` | GET/POST | Forum threads |

---

## Tool Groups Config

ปิด/เปิด tools เฉพาะกลุ่มได้ผ่าน `ψ/data/config.json` หรือ `arra.config.json`:

```json
{
  "toolGroups": {
    "forum": true,
    "trace": true,
    "vault": false,
    "schedule": true
  }
}
```

---

## Vault Structure ที่ index

```
ψ/memory/
├── resonance/*.md     → principles
├── learnings/*.md     → learnings (created by arra_learn)
├── retrospectives/    → retrospectives
└── traces/            → trace logs
```

---

## Write Tools (disabled in read-only mode)

- `arra_learn`
- `arra_thread` / `arra_thread_update`
- `arra_trace`
- `arra_supersede`
- `arra_handoff`

---

## Philosophy: "Nothing is Deleted"

ทุก document ที่ถูก supersede จะยังอยู่ใน DB แค่ mark `superseded_by` → เก็บ audit trail ไว้ตลอด
