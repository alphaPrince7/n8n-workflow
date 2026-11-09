# Automation & QA Developer — Take-Home Assessment

**Prince Sharma** · Automation & QA Developer assessment · Node.js / n8n

This repo contains my submission for the three tasks. Keep it **public** for review.

## What's here
```
workflows/
  Task2_Workflow_PrinceSharma.json        # Task 2 — GitHub → enrich → IF → Discord (importable)
  Task2_README.md                         # Task 2 explanation (APIs, transform, error path, credentials)
  Bonus_UptimeMonitor_PrinceSharma.json   # Bonus — pings the Task-1 app every 5 min, alerts on non-200
docs/
  Task1_QA_Report_PrinceSharma.pdf        # Task 1 — bug table (7 issues) + root-cause analysis
  (screenshots you add: workflow canvas, a successful execution, the uptime workflow)
```

## Task 1 — Web App QA & Debug Report
- App tested: **RealWorld "Conduit" demo** (https://demo.realworld.io).
- Exercised sign-up, login, create/edit/delete, comment, favorite, follow, log out, plus edge cases (long/special input, Slow-3G throttling, expired/corrupted session, double-click, a11y audit).
- Found **7 candidate issues** (documented in the required template) and wrote a root-cause analysis of the stale-authentication issue.
- See `docs/Task1_QA_Report_PrinceSharma.pdf`.

## Task 2 — n8n API Integration Workflow
- **GitHub trending repos** (search) → **Code node** keeps top 5 → **README enrichment** (2nd API call) → **IF** star-threshold branch (HOT/STANDARD) → **Discord** digest.
- **Error handling:** Continue-On-Fail + retries on every HTTP node; failures route to a dedicated error-handler node that logs a safe summary (never silent).
- **No hard-coded secrets:** the Discord webhook lives in n8n's Credentials store.
- Full write-up in `workflows/Task2_README.md`.

## Bonus — Uptime Monitor
- Pings the Task-1 app **every 5 minutes**, reads the HTTP status, and posts a Discord **alert on any non-200**.
- Extras: **3× retry** (so a single blip doesn't alert), **response-time tracking**, and `neverError` so even a 5xx is inspected rather than thrown.

## How to run
1. Install n8n (`npx n8n` or Docker) and open `http://localhost:5678`.
2. **Credentials → New** → store your Discord webhook URL.
3. **Import** the two workflow JSONs (Workflows → Import from File).
4. Open each, confirm the Discord node points at your credential, click **Execute Workflow**.
5. For Task 2's webhook test: `curl "http://localhost:5678/webhook-test/github-brief"`.

## Notes & assumptions
- Discord chosen as the notification channel for fastest free setup; swappable for Slack/Telegram with one node change.
- GitHub chosen as the data source: free, no key for basic use, clean JSON, and a defensible star threshold.
- Working over pretty, per the brief — canvases are laid out in execution order for readability.
