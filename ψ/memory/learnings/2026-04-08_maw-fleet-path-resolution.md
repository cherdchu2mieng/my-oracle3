---
title: maw fleet config — path resolution pattern
tags: [maw, fleet, ghq, multi-agent]
created: 2026-04-08
source: rrr: my-oracle3
project: github.com/cherdchu2mieng/my-oracle3
---

# maw fleet config — path resolution

maw resolves fleet window paths เป็น: `ghqRoot + "/" + repo`

ถ้า `ghqRoot = /home/cherdchu/ghq` และ `repo = cherdchu2mieng/my-oracle3`:
- maw ใช้: `/home/cherdchu/ghq/cherdchu2mieng/my-oracle3`
- ghq เก็บที่: `/home/cherdchu/ghq/github.com/cherdchu2mieng/my-oracle3`

**ก่อนสร้าง fleet config ต้อง verify:**
```bash
ghq list | grep <repo>
# → github.com/owner/repo
```

**fleet config repo field ต้องใส่ `github.com/owner/repo`** ไม่ใช่แค่ `owner/repo`

*Added via Oracle Learn*
