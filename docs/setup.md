# Setup guide

Estimated setup time: **30–45 min**. Estimated weekly run cost: **~€3–€8** for 500 leads.

## 1. Prerequisites

- An n8n instance (self-hosted or cloud) — v1.50 or later
- Accounts at: Apify, Anthropic, Dropcontact, PhantomBuster, Instantly.ai
- A Google Cloud project with Sheets API enabled
- A Google Sheet with **5 tabs** named exactly:
  `Wedding Planners` · `Festivals` · `Event Managers` · `Théâtres` · `Jeunes Artistes`
  Each tab must have this header row:
  ```
  name | city | website | email | phone | rating | reviewsCount | address | googleMapsUrl | email_subject | email_body
  ```

## 2. Import the workflow

1. Download `workflows/weekly-prospecting-event-industry-fr.json`
2. In n8n: **Workflows → Import from file**
3. Pick the JSON, click **Import**

## 3. Wire up credentials

Replace the following placeholders inside the imported workflow. A `.env.example` is provided as a checklist — n8n itself reads credentials from its own credential store, not from `.env`.

| Placeholder                    | Where to get it                                     |
|--------------------------------|-----------------------------------------------------|
| `YOUR_APIFY_TOKEN`             | apify.com → Settings → Integrations                 |
| `YOUR_ANTHROPIC_API_KEY`       | console.anthropic.com → API Keys                    |
| `YOUR_PHANTOMBUSTER_API_KEY`   | phantombuster.com → Account → API key               |
| `YOUR_PHANTOM_ID_EVENT_MANAGERS` | PhantomBuster phantom URL (numeric ID)            |
| `YOUR_PHANTOM_ID_THEATRES`     | PhantomBuster phantom URL (numeric ID)              |
| `YOUR_SPREADSHEET_ID_HERE`     | the `/d/<ID>/` segment of your Google Sheets URL    |
| Instantly.ai API key           | app.instantly.ai → Settings → Integrations          |
| Dropcontact                    | replace the "Dropcontact — Email Enrichment" Code node with an official Dropcontact node and bind your API key |

> **Security**: never commit real tokens. Use the n8n credentials UI, and keep this repo's `.env.example` as your reference.

## 4. First run

1. **Manually execute** the workflow first (▶ button).
2. Check that each branch produces non-empty output.
3. Expect the first full run to take **8–12 minutes** (Apify actor cold starts).
4. Inspect the Google Sheet — rows should appear under each tab.

## 5. Schedule

The `Every Week – Monday 9AM` trigger handles scheduling. To change cadence, edit that node.

## 6. Monitoring

- Enable **Error workflow** in n8n settings to catch failures.
- Dropcontact and Instantly both have soft rate limits; the workflow already waits 3 minutes between PhantomBuster launch and fetch.

## 7. Cost model (500 leads / week)

| Service        | Unit cost          | Estimate / week |
|----------------|--------------------|-----------------|
| Apify Google Places | ~€0.005 / place | ~€2.50         |
| PhantomBuster  | flat subscription  | €59 / month     |
| Dropcontact    | €0.10 / enrichment | ~€50 (max)      |
| Claude 3.5 Sonnet | ~€0.003 / lead  | ~€1.50          |
| Instantly      | flat subscription  | €37 / month     |
| **Total variable** |                | **~€54 / week** |

Numbers assume worst case (every lead enriched). In practice dedup + contact-page scraping reduces Dropcontact usage by 30–50 %.
