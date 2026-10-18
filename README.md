<p align="center"><a href="https://apify.com/DevilScrapes/ai-overview-citations"><img src="assets/banner.svg" alt="Google AI Overview Citation Scraper — JavaScript example" width="100%"></a></p>

# Google AI Overview Citations Scraper for JavaScript and Node.js: citation checks in your backend

<p align="center">
<a href="https://apify.com/DevilScrapes/ai-overview-citations"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/google-ai-overview-citations-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/google-ai-overview-citations-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Here is a Node.js starting point for the **[Google AI Overview Citation Scraper](https://apify.com/DevilScrapes/ai-overview-citations)**. One ES module, built on the official `apify-client`, starts a run with your queries and returns plain objects that say which sites Google's AI Overview linked to and in what order. Apify drives the browser, waits for the overview and deals with Google's interruptions, so your service only starts a run and reads the records. Use it as a CLI that saves `results.json` and `results.csv`, or lift the core call into an API route, a queue worker or a scheduled serverless function.

> **Actor:** [Google AI Overview Citation Scraper](https://apify.com/DevilScrapes/ai-overview-citations) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [google-ai-overview-citations-scraper-python](https://github.com/devilscrapes/google-ai-overview-citations-scraper-python)

## 🔥 What you get

- **Long-format rows**: every source the AI Overview links to becomes its own record holding the query, its slot in the citation carousel, the registrable domain, the full link and the anchor title Google showed.
- **Absence is recorded too**: a query that produced no AI Overview still writes one row with `ai_overview_appeared` set to false, so your denominator is never guesswork.
- **CAPTCHA pages are flagged, not hidden**: when Google keeps serving its interstitial, the Actor retries on a new proxy session and, if that still fails, writes a row with `blocked_by_captcha` set to true instead of pretending the overview was missing.
- **Markup drift you can see**: `selector_used` names which detection rule matched the overview container, so a change in Google's page shows up as a shift in that column.
- **Locale control**: `country` and `language` map to Google's `gl` and `hl` parameters, so the same query list can be compared across markets.
- **A spend ceiling built into the input**: `maxQueries` caps how many queries one run processes, whatever length the list has.

## 🎯 Use cases

### Add an AI-answer visibility check to a SEO product

Expose an endpoint that takes a customer's keywords, starts a run and returns, per keyword, whether their domain was cited and at what position. Each record is flat JSON, so it maps onto a database row or a dashboard card without reshaping.

### Post a webhook when a citation is won or lost

Keep the last report in storage, compare it with the new one after each scheduled run and send only the changes to Slack, Teams or any webhook URL. A domain that drops out of the carousel for a money keyword is worth a ping.

### Run checks on a serverless timer

A scheduled function on AWS Lambda, Cloud Run or Vercel can start the Actor with a fixed query list and store the rows. The heavy browser work stays on Apify, so the function needs no browser binaries and stays small.

### Back a client report with raw evidence

Agencies can save each record's `source_url` and `scraped_at` with the report they send, so a claim like "you are cited for this query" comes with the exact link and time it was observed.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/google-ai-overview-citations-scraper-javascript.git
cd google-ai-overview-citations-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [Google AI Overview Citation Scraper](https://apify.com/DevilScrapes/ai-overview-citations), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Four statements carry the whole integration. `call()` resolves once the run has finished, and `listItems()` then reads the records from the run's default dataset.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/ai-overview-citations').call({"queries": ["what causes inflation", "how do heat pumps work"], "country": "us", "language": "en", "maxQueries": 2, "proxyConfiguration": {"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"]}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` reads `input.json` and passes it on untouched. The bundled example checks two topics with `maxQueries` at 2, so trying it costs little. Keep the residential proxy group if your plan has it; otherwise the Actor switches to a small datacenter pool by itself and you should expect more CAPTCHA marker rows.

```json
{
  "queries": [
    "what causes inflation",
    "how do heat pumps work"
  ],
  "country": "us",
  "language": "en",
  "maxQueries": 2,
  "proxyConfiguration": {
    "useApifyProxy": true,
    "apifyProxyGroups": [
      "RESIDENTIAL"
    ]
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `queries` | array | yes | `["best running shoes 2026", "what causes inflation"]` | — |
| `country` | string | no | `"us"` | — |
| `language` | string | no | `"en"` | — |
| `maxQueries` | integer | no | `25` | — |
| `waitMsAfterLoad` | integer | no | `8000` | — |
| `proxyConfiguration` | object | yes | `{"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"]}` | — |

## 📤 Output

Expect two record shapes in the same array: citation records with a carousel position, domain, link and title, and single marker records for queries without an overview or stuck behind a CAPTCHA. Branch on `ai_overview_appeared` and `blocked_by_captcha` in your handler before treating a record as a citation.

| Field | Type | Always present |
|---|---|---|
| `query` | string | yes |
| `country` | string | yes |
| `language` | string | yes |
| `ai_overview_appeared` | boolean | yes |
| `ai_overview_text_excerpt` | string or null | no |
| `citation_position` | integer or null | no |
| `source_domain` | string or null | no |
| `source_url` | string or null | no |
| `source_title` | string or null | no |
| `selector_used` | string or null | no |
| `blocked_by_captcha` | boolean | no |
| `query_error` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Citation row | $0.005 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/ai-overview-citations) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This script groups the saved records by query, checks whether a domain you pass on the command line (for example `node watch.js example.com`) was cited, and writes `watch-report.json`. If `WEBHOOK_URL` is set it also POSTs the report there. Node.js built-ins only, including the global `fetch`.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const rows = JSON.parse(await readFile('results.json', 'utf8'));
const watched = (process.argv[2] ?? 'wikipedia.org').toLowerCase();

const byQuery = new Map();
for (const row of rows) {
    const entry = byQuery.get(row.query) ?? { query: row.query, overview: false, blocked: false, sources: [] };
    entry.overview ||= row.ai_overview_appeared === true;
    entry.blocked ||= row.blocked_by_captcha === true;
    if (row.source_domain) entry.sources.push({ domain: row.source_domain, position: row.citation_position });
    byQuery.set(row.query, entry);
}

const report = [...byQuery.values()].map(({ query, overview, blocked, sources }) => {
    const hits = sources.filter((s) => s.domain === watched || s.domain.endsWith(`.${watched}`));
    const positions = hits.map((s) => s.position).filter((p) => p !== null && p !== undefined);
    return { query, overview, blocked, cited: hits.length > 0, bestPosition: positions.length ? Math.min(...positions) : null, sources: sources.length };
});

console.table(report);
const payload = JSON.stringify({ watched, checkedAt: new Date().toISOString(), report }, null, 2);
await writeFile('watch-report.json', payload);
if (process.env.WEBHOOK_URL) {
    const res = await fetch(process.env.WEBHOOK_URL, { method: 'POST', headers: { 'content-type': 'application/json' }, body: payload });
    console.log(`Webhook answered ${res.status}`);
}
```

## ❓ FAQ

### Can I call it from TypeScript?

Yes. `apify-client` includes type definitions, so moving the core call into a `.ts` file gives you typed run and dataset objects. Describe the record fields with your own interface; nullable ones such as `citation_position` should be typed as possibly null.

### How long does a run take, and should my HTTP request wait for it?

Every query loads a full results page and then waits several seconds (`waitMsAfterLoad`, 8 seconds by default) for the overview to render, so a long list takes minutes. In a web service, start the run with `start()`, return the run ID and collect results from a webhook or a follow-up job instead of holding the request open.

### How do I keep test runs cheap?

Use one or two queries, set `maxQueries` to match, and pass `--max-charge` with a small dollar figure. That flag becomes the run's spending ceiling on Apify's side, so a typo in the query list cannot turn into a surprise bill.

### What does `query_error` mean?

It is set only when every attempt to load a query's page crashed, for example on repeated navigation timeouts. Log those records and retry the query later; the other queries in the run are not affected.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/ai-overview-citations) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Google AI Overview Citation Scraper on Apify Store](https://apify.com/DevilScrapes/ai-overview-citations)
- [Python version of this example](https://github.com/devilscrapes/google-ai-overview-citations-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Bing Search API Replacement Scraper](https://apify.com/DevilScrapes/bing-search-scraper) — classic organic Bing results for the same query list, to compare against what the AI answer cites.
- [Google News Scraper — Search & Topic RSS Feeds](https://apify.com/DevilScrapes/google-news-scraper) — news coverage for a topic, useful for spotting which publishers are earning fresh mentions.
- [XML Sitemap URL Extractor](https://apify.com/DevilScrapes/xml-sitemap-url-extractor) — pull the URLs a cited competitor lists in its sitemap to see what content it publishes.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/ai-overview-citations) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Google.
