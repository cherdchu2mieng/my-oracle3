# maw + oracle-v2 setup patterns

## maw (multi-agent-workflow-kit)

ติดตั้ง: `uvx --no-cache --from git+https://github.com/Soul-Brews-Studio/multi-agent-workflow-kit.git@v0.5.1 multi-agent-kit init`

- สร้าง git worktrees สำหรับ agents (agents/1, agents/2, agents/3)
- จัดการผ่าน tmux session ชื่อ `ai-{repo-name}`
- ส่งงาน: `maw hey <agent> "<message>"`
- Broadcast: `maw send "<command>"`
- ต้อง `source .envrc` ก่อนใช้ทุกครั้ง

## oracle-v2 dashboard

- API server: `bun src/server.ts` → port 47778
- Frontend: `bunx vite --port 3000` → port 3000
- MCP server (stdio): spawn อัตโนมัติโดย Claude Code
- Database: `~/.arra-oracle-v2/oracle.db` (SQLite)
- Start script: `~/start-oracle.sh`

## tmux attach from Claude Code

`maw attach` / `tmux attach` ไม่ทำงานใน Claude Code terminal
→ ต้องเปิด terminal อื่นแล้วรัน: `tmux attach-session -t ai-my-oracle3`
