---
title: maw fleet config path resolution: ghqRoot + "/" + repo
tags: [maw, fleet, ghq, multi-agent, path-resolution]
created: 2026-04-08
source: rrr: my-oracle3
project: github.com/cherdchu2mieng/my-oracle3
---

# maw fleet config path resolution: ghqRoot + "/" + repo

maw fleet config path resolution: ghqRoot + "/" + repo

maw ใช้ `ghqRoot/repo` ในการ resolve path ดังนั้น fleet config ต้องใส่ `github.com/owner/repo` ไม่ใช่แค่ `owner/repo`

ตัวอย่าง:
- ghqRoot = /home/cherdchu/ghq
- repo field = "github.com/cherdchu2mieng/my-oracle3"
- resolved path = /home/cherdchu/ghq/github.com/cherdchu2mieng/my-oracle3

ก่อนสร้าง fleet config ต้อง verify ก่อน:
```bash
ghq list | grep <repo>  # → github.com/owner/repo
```

---
*Added via Oracle Learn*
