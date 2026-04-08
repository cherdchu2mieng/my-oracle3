---
title: Oracle v2 ใช้ hybrid search architecture: FTS5 (keyword matching) + ChromaDB (ve
tags: [oracle, search, hybrid, fts5, chromadb, vector, embedding]
created: 2026-04-02
source: Session observation 2026-04-02
project: github.com/cherdchu2mieng/my-oracle3
---

# Oracle v2 ใช้ hybrid search architecture: FTS5 (keyword matching) + ChromaDB (ve

Oracle v2 ใช้ hybrid search architecture: FTS5 (keyword matching) + ChromaDB (vector/semantic search) ทำงานร่วมกัน

- FTS5: full-text search ใน SQLite, เร็ว, ทำงานได้เสมอ
- ChromaDB: embedding-based semantic search, ต้องมี model รัน (bge-m3 default, nomic, qwen3)
- Hybrid mode (default): รวมผลจากทั้งสอง, fallback เป็น FTS-only ถ้า ChromaDB ไม่พร้อม
- ถ้า vector search ไม่ return ผล → ChromaDB อาจยังไม่ index หรือ embedding model ยังไม่รัน

---
*Added via Oracle Learn*
