# Learning: ghq vs ~/repos — Oracle Folder Pattern

**Date**: 2026-04-08
**Source**: Session cleanup + ancestor repo research

---

## Pattern

```
~/repos/                    ← Oracle repos ของ Mieng เท่านั้น
├── my-oracle3/             ← Pegasus
├── writer-oracle/          ← Oracle ใหม่ที่กำลังสร้าง
└── (oracle ต่อไป)/

ghq/github.com/
├── cherdchu2mieng/
│   └── my-oracle3 → ~/repos/my-oracle3   ← symlink!
└── Soul-Brews-Studio/
    ├── maw-js/             ← tool repos ของคนอื่น
    ├── arra-oracle-v3/
    ├── oracle-studio/
    └── oracle-v2/
```

## Rules

1. **`ghq get` ≠ symlink** — clone fresh จาก GitHub เสมอ ถ้า maw fleet ต้องชี้ไป existing Oracle repo ใช้ `ln -sf ~/repos/oracle ghq/.../oracle` แทน
2. **tool repos** (Soul-Brews-Studio) → ghq เท่านั้น ไม่ใส่ใน ~/repos
3. **หลัง `/awaken` ใหม่** → สร้าง symlink เข้า ghq ให้ maw fleet ใช้งานได้

## Why

Nat ค้นพบ "ghq + symlink pattern" (BIG WIN, Dec 2025) — symlink repos เข้า ψ/ และ ghq เพื่อให้ Oracle access ได้โดยไม่ต้องมี duplicate copies pattern นี้คือ DNA ของ Oracle architecture
