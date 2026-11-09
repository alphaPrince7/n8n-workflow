# Task 2 — n8n API Integration Workflow

**Workflow:** `Task2_Workflow_PrinceSharma.json` — "GitHub Trending Morning Brief"

A daily-brief style workflow: it pulls trending GitHub repos for a topic, keeps the top 5, enriches each with its README size, routes them by a star threshold, and posts a formatted digest to Discord — with error handling so a failed API call never crashes the run.

## APIs used (and why)
- **GitHub REST API** (first call: `GET /search/repositories?q=topic:<topic>&sort=stars`).
  *Why:* free, no key needed for basic rate limits, returns rich structured JSON, and is easy to reason about in the follow-up call. Stars give a natural, defensible threshold.
- **GitHub README endpoint** (enrichment: `GET /repos/{owner}/{repo}/readme`).
  *Why:* a second, real enrichment call on the same provider — fetches each repo's README metadata (size), which I surface as "README KB" in the digest. Shows I can chain a per-item follow-up request.
- **Discord webhook** (output).
  *Why:* the fastest free notification channel to set up — a webhook URL created in any server, no OAuth/app-review.

## What the transformation does
A **Code node** (`Code: Top 5 Repos`) takes the search response, slices to the **top 5** by stars, and reshapes each into just the fields the digest needs (`name, stars, description, url, owner, repo`). A second small Code node attaches the README size from the enrichment call. This keeps the payload small and the digest clean.

## Conditional branch
An **IF node** compares each repo's `stars` to a **config-driven threshold** (`starThreshold`, set in the `Set Config` node, default 20000). Repos above it are tagged **🔥 HOT**, the rest **⭐ STANDARD**, then merged back for one combined digest. The threshold lives in config, not hard-coded in the node, so it's easy to change and easy to explain.

## How the error path behaves
- The two GitHub HTTP nodes and the Discord node use **`onError: continueErrorOutput`** (Continue-On-Fail) with **retries** (3× / 2× with a wait), so transient failures self-heal.
- On a real failure, the node's **error output** routes to a dedicated **`Error Handler (fallback)`** Code node that logs a **structured, non-sensitive summary** (`status, node, message, timestamp`) — so a failed API call is **logged, never silent**. In production this node would post to an ops channel.

## Credentials (no hard-coded secrets)
The Discord webhook URL is **not** in the workflow. It's stored in n8n's **Credentials** store and referenced from the node.

**Setup (one-time):**
1. In Discord: Server Settings → Integrations → Webhooks → New Webhook → copy the URL.
2. In n8n: **Credentials → New → Generic Credential type** (or use the dedicated Discord node's credential). Store the webhook URL there.
3. The HTTP node references the credential rather than containing the URL.
   *(If you prefer, swap the HTTP Request output node for n8n's built-in **Discord node**, which has first-class credential support — same result.)*

## How to run / test
- **Manual:** open the workflow, click **Execute Workflow**. The Schedule trigger fires hourly in production; for testing use the **Webhook** trigger:
  ```bash
  curl "http://localhost:5678/webhook-test/github-brief"
  ```
- A successful run posts the digest to your Discord channel and shows green nodes with output data.

## Design notes
- Working over pretty, per the brief: the canvas is laid out left-to-right in execution order so it reads top-down.
- The topic and threshold are the two "knobs" — both in the `Set Config` node — so the behaviour is easy to change without touching logic.
