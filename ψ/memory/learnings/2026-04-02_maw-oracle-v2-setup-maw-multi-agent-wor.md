---
title: ## maw + oracle-v2 setup
tags: [maw, multi-agent, oracle-v2, tmux, worktree, dashboard, setup]
created: 2026-04-02
source: rrr: my-oracle3
project: github.com/cherdchu2mieng/my-oracle3
---

# ## maw + oracle-v2 setup

## maw + oracle-v2 setup

**maw** (multi-agent-workflow-kit) ติดตั้งด้วย:
`uvx --no-cache --from git+https://github.com/Soul-Brews-Studio/multi-agent-workflow-kit.git@v0.5.1 multi-agent-kit init`

- สร้าง git worktrees: agents/1, agents/2, agents/3
- tmux session: `ai-{repo-name}`
- ส่งงาน: `maw hey <agent> "<message>"`
- ต้อง `source .envrc` ก่อนใช้ทุกครั้ง

**oracle-v2 dashboard:**
- API: `bun src/server.ts` → port 47778
- Frontend: `bunx vite --port 3000` → port 3000
- Database: `~/.arra-oracle-v2/oracle.db`
- Start script: `~/start-oracle.sh`

**Note:** `tmux attach` ไม่ทำงานใน Claude Code terminal → ต้องเปิด terminal อื่นแล้วรัน `tmux attach-session -t ai-my-oracle3`

---
*Added via Oracle Learn*
