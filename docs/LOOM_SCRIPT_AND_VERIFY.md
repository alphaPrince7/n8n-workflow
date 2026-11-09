# Loom Script + Run & Verify Checklist

## PART A — Run & verify BEFORE you record (do this first)

These are the things only you can do — the submission must reflect YOUR real runs.

### Task 1 (QA)
- [ ] Open https://demo.realworld.io and reproduce each of the 7 bugs in the report.
- [ ] Delete any bug that no longer reproduces; keep at least 5 you personally confirmed.
- [ ] Take a screenshot of each confirmed bug (encouraged, not required).

### Task 2 (n8n workflow)
- [ ] Install n8n: `npx n8n`  (or Docker). Open http://localhost:5678.
- [ ] Create a Discord webhook (Server Settings → Integrations → Webhooks) and store it in n8n Credentials.
- [ ] Import `Task2_Workflow_PrinceSharma.json`.
- [ ] Open the Discord node → point it at your credential.
- [ ] Click **Execute Workflow** → confirm the digest lands in Discord.
- [ ] Screenshot (a) the canvas, (b) a successful execution showing output data.

### Bonus (uptime)
- [ ] Import `Bonus_UptimeMonitor_PrinceSharma.json`.
- [ ] Execute once → confirm it reads status 200 (no alert).
- [ ] Temporarily change the target URL to a bad one (e.g. add `/nonexistent-xyz` or use https://httpstat.us/503) → execute → confirm an alert posts to Discord.
- [ ] Screenshot the workflow.

### Push to GitHub (public)
```bash
cd n8n-qa-automation
git init
git add .
git commit -m "Task 1 QA report"
git commit --allow-empty -m "Task 2 n8n workflow + README"   # or stage separately
git branch -M main
git remote add origin https://github.com/alphaPrince7/automation-qa-assessment.git
git push -u origin main
```
(Commit in a few logical chunks — Task 1, then Task 2, then Bonus — so it reads like real work.)

---

## PART B — Loom script (aim for 4–6 minutes)

**0. Intro (20s)**
"Hi, I'm Prince. This is my Automation & QA take-home. Three parts: a QA report on a
vibe-coded app, an n8n API workflow, and the bonus uptime monitor. I'll walk through
each and explain my choices."

**1. Task 1 — QA (90s)**
- Show the RealWorld demo. Reproduce 2 of your strongest bugs live (e.g. stale session,
  no validation).
- "I found 7 issues total; they're in the report with repro steps and severity."
- Open the root-cause analysis: "I picked the stale-auth bug. The app trusts a token in
  local storage without re-validating it, and there's no global 401 handler — so expired
  sessions silently fail. Fix: a 401 interceptor that logs the user out, plus validating
  the token on load."

**2. Task 2 — n8n workflow (120s)**
- Show the canvas left to right. Narrate each node:
  - "Trigger — schedule hourly, or a webhook for manual testing."
  - "GitHub search for a topic, sorted by stars. **Why GitHub?** Free, no key, clean JSON."
  - "Code node keeps the top 5 and reshapes them."
  - "Second call enriches each with its README size — that's the required second API call."
  - "IF node routes by a star threshold — **why 20k?** It cleanly separates the standout
     repos; it's config-driven so it's easy to change."
  - "Digest posts to Discord. **Why Discord?** Fastest free channel to set up."
  - "**Error handling:** every HTTP node is Continue-On-Fail with retries, and failures
     route to this error node that logs a safe summary — nothing fails silently."
  - "The webhook URL is in Credentials, not hard-coded."
- Click Execute → show the message land in Discord.

**3. Bonus — uptime monitor (60s)**
- "Pings the app every 5 minutes, checks the status code, alerts on anything but 200."
- Show the healthy run (no alert), then the forced-failure run (alert in Discord).
- "Extras: 3 retries so one blip doesn't page anyone, plus response-time tracking."

**4. Wrap (20s)**
"Everything's in the public repo with a README. Happy to go deeper on any choice. Thanks!"

---

## Defense-call prep (they WILL ask these)
- **Why GitHub API?** Free, no auth needed for basic rate limits, structured JSON, defensible threshold (stars).
- **Why the README enrichment?** A real second call on real data; shows chaining a per-item request.
- **Why threshold 20,000 stars?** Separates genuinely standout repos from the long tail; it's in config so it's trivially tunable — I'd tune it to the topic in production.
- **Why Discord?** Fastest free notification setup; no OAuth. Swappable for Slack/Telegram.
- **How does error handling work?** Continue-On-Fail + retries on HTTP nodes; a dedicated error node logs a non-sensitive summary. Nothing crashes or fails silently.
- **Why not hard-code the webhook?** Secrets belong in the Credentials store — rotatable, not committed to Git, not visible in exports.
