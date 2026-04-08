---
title: ghq vs ~/repos Oracle folder pattern:
tags: [ghq, folder-structure, symlink, oracle-setup, maw]
created: 2026-04-08
source: rrr: cherdchu2mieng/my-oracle3
project: github.com/cherdchu2mieng/my-oracle3
---

# ghq vs ~/repos Oracle folder pattern:

ghq vs ~/repos Oracle folder pattern:

~/repos/ = Oracle repos ของ Mieng เท่านั้น (my-oracle3, writer-oracle, etc.)
ghq/github.com/Soul-Brews-Studio/ = tool repos ของคนอื่น (maw-js, arra-oracle-v3, oracle-studio)

Rules:
1. `ghq get` ไม่ใช่ symlink — clone fresh จาก GitHub เสมอ
2. หลัง awaken ใหม่ → ln -sf ~/repos/oracle ghq/.../oracle (symlink เข้า ghq)
3. tool repos ไม่ควรอยู่ใน ~/repos

Pattern นี้คือ "ghq + symlink BIG WIN" ที่ Nat ค้นพบ Dec 2025

---
*Added via Oracle Learn*
