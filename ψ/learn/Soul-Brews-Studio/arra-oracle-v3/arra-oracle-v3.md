# arra-oracle-v3 Learning Index

## Source
- **Origin**: ./origin/
- **GitHub**: https://github.com/Soul-Brews-Studio/arra-oracle-v3

## Explorations

### 2026-04-02 13:22 (default)
- [[2026-04-02/1322_ARCHITECTURE|Architecture]]
- [[2026-04-02/1322_CODE-SNIPPETS|Code Snippets]]
- [[2026-04-02/1322_QUICK-REFERENCE|Quick Reference]]

**Key insights**:
1. **arra-oracle-v3 คือ v3 repo แต่ package.json ยังชื่อ `arra-oracle-v2`** — rebrand ยังไม่เสร็จ
2. **Vector store abstraction รองรับ 5 backends**: LanceDB (default), sqlite-vec, Qdrant, ChromaDB, Cloudflare Vectorize — เลือกผ่าน env var
3. **"Nothing is Deleted" philosophy** บังคับในระดับ DB — supersede_log เก็บ audit trail แม้ file จะถูกลบ
4. **Trace system** มี linked list (prev/next) + tree (parent/child) — ทำให้ chain traces ได้ทั้งแนวนอนและแนวตั้ง
5. **Tool groups config** ให้ disable tools เป็น group ได้ผ่าน `ψ/data/config.json`
