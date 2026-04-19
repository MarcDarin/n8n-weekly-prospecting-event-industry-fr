# Architecture — Weekly Prospecting (Event Industry FR)

## High-level flow

```mermaid
flowchart TD
    A[Schedule Trigger<br/>Every Monday 9AM] --> B1[Apify Google Places<br/>5 queries in parallel]
    A --> B2[Apify Instagram<br/>Wedding Planners + Artistes]
    A --> B3[PhantomBuster LinkedIn<br/>Event Managers + Theatres]

    B1 --> C1[Process & normalize<br/>5 Code nodes]
    B2 --> C2[Process Instagram]
    B3 --> C3[Process LinkedIn]

    C1 --> D[Merge All Results]
    C2 --> D
    C3 --> D

    D --> E[Deduplicate<br/>on email + domain]
    E --> F[Scrape contact page<br/>fallback for missing emails]
    F --> G[Dropcontact<br/>email enrichment]

    G --> H[Build AI prompt<br/>per lead]
    H --> I[Claude 3.5 Sonnet<br/>subject + body]
    I --> J[Parse & merge<br/>AI response]

    J --> K{Switch<br/>by category}
    K -->|Wedding Planners| L1[Google Sheets tab 1]
    K -->|Festivals|        L2[Google Sheets tab 2]
    K -->|Event Managers|   L3[Google Sheets tab 3]
    K -->|Theatres|         L4[Google Sheets tab 4]
    K -->|Jeunes Artistes|  L5[Google Sheets tab 5]

    L1 --> M[Instantly.ai<br/>Cold email campaign]
    L2 --> M
    L3 --> M
    L4 --> M
    L5 --> M
```

## Node breakdown

| Stage            | Nodes | Purpose                                                                   |
|------------------|-------|---------------------------------------------------------------------------|
| Trigger          | 1     | Weekly schedule (Monday 9AM)                                              |
| Apify (Places)   | 5     | Google Maps scraping per category                                         |
| Apify (Instagram)| 3     | Public profile scraping for visual categories                             |
| PhantomBuster    | 4     | LinkedIn Sales Navigator scraping (2 x launch + fetch)                    |
| Code (process)   | 8     | Normalize raw API responses into a unified lead schema                    |
| Merge            | 1     | Unify all channels                                                        |
| Deduplicate      | 1     | Drop leads already seen (by email or domain)                              |
| Contact scraper  | 1     | Fallback: parse contact page HTML to extract emails                       |
| Dropcontact      | 1     | Waterfall email enrichment + validation (GDPR compliant)                  |
| AI personalisation | 3   | Build prompt -> Claude API -> parse JSON                                  |
| Routing          | 1     | Switch node fans out by `category` field                                  |
| Google Sheets    | 5     | One tab per category, appends rows                                        |
| Instantly.ai     | 5     | Pushes personalised sequences into cold email campaigns                   |

Total: **58 nodes** (including 5 sticky notes used as inline documentation).

## Data schema (per lead)

```json
{
  "name": "string",
  "city": "string",
  "website": "string",
  "email": "string",
  "phone": "string",
  "rating": "number",
  "reviewsCount": "number",
  "address": "string",
  "googleMapsUrl": "string",
  "category": "wedding_planner|festival|event_manager|theatre|jeune_artiste",
  "email_subject": "string (AI generated)",
  "email_body":    "string (AI generated, uses pain points)"
}
```

## Why this architecture

- **Parallel scraping** reduces total runtime to the slowest category (~6 min) instead of summing all of them.
- **Deduplication before enrichment** saves Dropcontact credits (€0.10/lead).
- **AI personalisation *after* enrichment** means only verified leads consume Claude tokens.
- **Category routing at the end** keeps the scraping graph simple and lets you pause any segment without rewiring the flow.
