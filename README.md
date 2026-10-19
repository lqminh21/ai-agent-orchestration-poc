# AI Agent Orchestration PoC

Dashboard giám sát và điều phối agent Claude Code: tạo agent/prompt/pipeline bằng file markdown, kéo thả prompt vào agent, xem SDLC pipeline chạy theo thời gian thực.

## Chạy

Yêu cầu: Node 22+, đã đăng nhập Claude Code (`claude` chạy được) hoặc có `ANTHROPIC_API_KEY`.

```bash
npm install
npm run dev        # server ws://localhost:3001 + UI http://localhost:5173
```

Biến môi trường (tuỳ chọn):

| Biến | Mặc định | Ý nghĩa |
|---|---|---|
| `WORKSPACE_DIR` | `./workspace` | Thư mục chứa các file md |
| `AGENT_CWD` | `demo-app` | Thư mục làm việc của agent tạo từ bảng (tương đối với `WORKSPACE_DIR`) |
| `PORT` | `3001` | Cổng server |

## Cách dùng

- **Agents tab:** kéo một agent từ thư viện (trái) vào bảng → tạo agent mới. Kéo một prompt vào thẻ agent → giao việc (agent đang bận thì vào hàng đợi). Click thẻ để xem timeline, todo, duyệt quyền tool, nhắn thêm.
- **Pipeline tab:** click một pipeline trong thư viện → graph hiện ra. Kéo agent/prompt vào node để đổi cho lần chạy tiếp theo. Bấm **Chạy**, điền input. Stage có `gate: manual` sẽ chờ bạn **Duyệt** hoặc **Yêu cầu sửa** ở panel phải.
- Sửa file md → thư viện tự cập nhật; file lỗi hiện ⚠ kèm lý do.

## File markdown

```
workspace/
├─ .claude/agents/*.md            # agent: frontmatter (name, description, tools, model) + system prompt
├─ .orchestrator/prompts/*.md     # prompt/task, có biến {{...}}
├─ .orchestrator/pipelines/*.md   # pipeline: stages + needs + gate
└─ demo-app/                      # project mẫu cho agent làm việc
```

Biến trong prompt: `{{inputs.<key>}}` (input khi bấm Chạy), `{{stages.<id>.output}}` (output stage trước), `{{output}}` (đường dẫn output của stage hiện tại). Prompt kéo thả trực tiếp vào agent: mọi biến sẽ được hỏi trong dialog.

Định dạng agent giống subagent của Claude Code nên dùng lại được trong CLI.

## Cấu trúc code

```
server/
  index.ts            WebSocket server, state, lệnh từ UI, hot reload file md
  library.ts          đọc + validate file md (gray-matter + zod)
  agent-session.ts    bọc Agent SDK query() ở chế độ streaming input
  pipeline-runner.ts  chạy stage theo needs, gate, truyền output
shared/               types + template dùng chung server/UI
src/                  React + shadcn + React Flow
```

## Giới hạn của PoC

- State lưu trong bộ nhớ: restart server là mất session/run.
- Chưa có git worktree: các agent cùng sửa một thư mục.
- Chỉ có Claude; chưa có CodexProvider.
- Chi phí hiển thị là ước tính phía client (`total_cost_usd` của SDK), không phải hoá đơn.
- Agent chạy với `settingSources: []` (không nạp settings/hooks/plugins trong `~/.claude`). Read/Grep/Glob/TodoWrite tự chạy; các tool khác hỏi duyệt trừ khi bật "Tự duyệt" hoặc `autoApprove: true` trong pipeline.
# ai-agent-orchestration-poc
