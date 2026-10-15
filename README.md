# Groww Review Intelligence — approval-gated prototype

A local prototype that analyzes the prepared Groww sample or a CSV you upload, then prepares a weekly product pulse and fee explanation tied to the detected feedback. Uploaded CSV analysis offers a local keyword mode and an optional OpenAI structured-output mode.

## Run

Requires Python 3.9+; no third-party packages are needed. On managed hosts such as Railway, the app reads the injected `PORT` variable and binds on `0.0.0.0`.

```bash
cd outputs
python3 server.py
```

Open <http://127.0.0.1:8000>. To use your own data, upload a CSV containing a review text column (such as `review`, `text`, `content`, `body`, or `feedback`) and optionally a date and source URL column, then select an analysis mode and click **Analyze CSV**. If a date column exists, rows without a valid date and rows older than 12 weeks are excluded; without a date column, the prototype cannot enforce a review window. Review the generated result, tick the approval checkbox, then click **Approve & create outputs**.

Keyword mode parses and analyzes the CSV in the browser. OpenAI mode becomes available when `OPENAI_API_KEY` is configured for the server. It sends review text and dates from the filtered batch to OpenAI only after the user checks the explicit data-sharing consent box. Source URLs are not sent to OpenAI; they are retained locally to link selected quotes. The server keeps the API key and sends requests with response storage disabled. If OpenAI is not configured or returns an error, the interface reports it and keyword mode remains available.

To enable OpenAI mode, configure the key in the shell that starts the server, then restart it:

```bash
export OPENAI_API_KEY="your-api-key"
export OPENAI_MODEL="gpt-6-astra"  # optional; this is the default
python3 server.py
```

The LLM response uses a strict JSON Schema for review labels, issue, quote IDs, actions, and fee bullets. The server verifies label coverage, limits clustering to five themes, selects quotes from the original review text, requires a repeated fee category before presenting it as recurring, and composes a pulse capped at 250 words. A real OpenAI call cannot be made until you configure a valid API key with model access.

## Approval and simulated MCP actions

The checkbox and button are the human approval gate. The browser sends `approved: true` and the current analysis to the local `/api/approve` endpoint only after that action. The endpoint rejects requests without approval, then performs two local, MCP-style simulated write actions:

1. Appends the required structured fields (`date`, `top_themes`, `weekly_pulse`, `identified_fee_issue`, `explanation_bullets`, `source_links`) plus approval metadata to `notes.jsonl`.
2. Creates `email_drafts/weekly-product-pulse.eml` with subject **Weekly Product Pulse + Customer Clarification — Groww** and the pulse plus reusable fee snippet in the body.

This simulates Notes/Docs and Gmail draft actions for the prototype. It is not an MCP server or a live connection to those providers, and it never sends email. Output links appear only after approval. The analysis runs on demand after CSV upload; it is not a real-time review feed.

## Fee issue identified

Recent public feedback repeatedly asks why the net amount after selling shares is lower than the displayed sale value and what “external charges” contain. The linked official Groww pages support a facts-only explanation of brokerage, DP charges, taxes and other possible line items. Pricing can change, so the fee snippet includes its check date and official sources.

## Deliverables

- `index.html` — interactive prototype with CSV upload, keyword mode, and opt-in OpenAI mode
- `server.py` — local analysis endpoint, approval gate, and simulated MCP-style write actions
- `reviews.csv` — eight public Reddit feedback excerpts from 23 Jul–25 Sep 2026
- `weekly-pulse.md` — structured internal note (235 words, including quotes and headings)
- `fee-explainer.md` — customer snippet (six bullets) with two official links and last-checked date
- `source-list.md` — six selected review and official source URLs; all seven unique Reddit URLs remain in the CSV

## Data and method notes

The CSV is a manually curated, convenience sample of public Reddit posts/comments, not a Google Play/App Store export and not a representative sample. Theme counts refer only to these eight excerpts. Quotes are linked to their source posts. See `source-list.md` for limitations and source links.

## Deploy

The app is a small Python standard-library web server. The repository includes a `Procfile` and `railway.json` for Railway deployment:

1. Push this directory to a GitHub repository.
2. In Railway, create a project from that GitHub repository and deploy the `outputs` project root (or make this repository root the app directory).
3. Generate a public domain in Railway. The health check is `/api/status`.
4. Keep `OPENAI_API_KEY` unset unless you intentionally want to enable the optional OpenAI mode; never commit API keys.

The browser analyzes uploaded CSVs locally in keyword mode. If you approve a result, its selected quotes and summary are sent to the app server to create the Notes entry and `.eml` draft. Do not upload confidential or personal customer data to a public demo. The sample feedback is a small public Reddit convenience sample. Hosting uses local file writes for this prototype; generated records are not a durable production database.

## Submission status and limitations

The approval gate simulates internal Notes/Docs and Gmail draft actions; it is not a live MCP server or a connection to those providers, and it never sends email. OpenAI analysis is implemented but remains unavailable until a valid API key with model access is configured. There is no live review feed. The prepared Notes entry and email draft contain the approved sample output; uploaded analyses are appended only after approval.
