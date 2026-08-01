# wademillerbombers.com — Site Ops

Autonomous CFL parody/news site by **NLK Automation**. All content is AI-synthesized ("AI-synthesized analysis" disclaimed on every page).

Full operational reference (services, cron, wadebot internals, section regeneration, pitfalls): see the `bombers-site-ops` Hermes skill.

## Stack

- **Serving**: nginx → static `/var/www/wademillerbombers` + 3 local Python services (systemd)
- **AI**: DeepSeek **`deepseek-v4-flash`** for all generation and the chatbot (explicit model name — never `deepseek-chat` alias / `v4-pro`). Reasoning budget `max(max_tok*4, 8000)`.
- **Images**: FAL/FLUX `fal-ai/flux-2/klein/9b` (key from 1Password), converted to webp.

## Services

| Service | Port | Script | Purpose |
|---|---|---|---|
| wadebot | 8766 | `chatbot-server.py` | Wade Bot chat API + chat-log admin API |
| form | 8765 | `form-server.py` | contact form, leads, newsletter |
| analytics | 8768 | `analytics-server.py` | visitor tracking |

Scripts live in `/root/.hermes/scripts` (git). `systemctl restart wadebot` after editing the bot.

## Content pipeline (4x daily: 6/10/14/20h)

`bombers-daily-update.py` (research → DeepSeek v4-flash → parse) → `bombers-daily-update-safe.py` (BeautifulSoup, AUTO-marker sections only) → sitemap → archive interlinks → validator → FAL image → X post → mailer (6AM only).

**Top Wade Bot Conversations** (`AUTO:CHATREACT`): regenerated near-real-time after every chat by `bombers-chat-react-refresh.py`, and every pipeline run. Sources: live `chat-logs.json` + durable `chat-bank/`. Max 6 cards, newest first, full replies.

**Around the CFL**: drama/controversy content, refreshed ~weekly (`cfl-refresh-state.json`).

## Admin

- Leads: `/leads` · Chat log (view/edit/delete): `/chats` — both basic-auth (`.htpasswd-bombers-leads`)
- Chat admin API: `GET /chats-data`, `POST /chats-admin/delete|edit` (keys = `sha1(ts|ip)[:20]`)
- Homepage serves `Cache-Control: no-cache` — always fresh after pipeline runs.

## Repos

- Site: this repo (auto-commit 4:15AM via `site-backup.sh`)
- Scripts: `/root/.hermes/scripts` (cron-referenced scripts must be committed)
