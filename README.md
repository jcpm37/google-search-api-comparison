# Google Search API: Is Google's Own JSON API Still Available, What Does It Cost, and What Should You Use Instead? (Full Plan Comparison)

Google's Custom Search JSON API is closed to new customers and will be discontinued on January 1, 2027. Existing customers keep 100 free queries per day, then pay $5 per 1,000 queries, with a ceiling of 10,000 queries per day. If you are starting a project now, the practical question is no longer "how do I call Google's API" but "which third-party SERP provider returns Google results as JSON, and what will it cost me per thousand lookups."

This article covers what the official google search api still does and where it hard-stops, how ScraperAPI's structured Google endpoint works and what a single Google request actually costs in credits, a full plan comparison, and two other technical routes (Gemini's search grounding, plus SerpApi and Apify) so you can tell which shape of output you actually need.

> **Quick pick:** ScraperAPI's free tier includes 1,000 credits and 5 concurrent connections. At 25 credits per Google request, that is roughly 40 Google searches, enough to confirm the JSON fields, geo-targeting parameters and latency before you pay anything. 👉 [Start on the free tier and see the full plan list](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What Google's own search API still does

The Custom Search JSON API is one endpoint, called over HTTP GET, that returns web or image results from a Programmable Search Engine as JSON. The response format follows the OpenSearch 1.1 specification.

Three parameters are mandatory on every call: `key` for your API key, `cx` for the Programmable Search Engine ID, and `q` for the search expression. Everything else is optional, and all values have to be URL encoded. Setup is two separate steps that both have to happen before your first request works. You create and configure a Programmable Search Engine in the control panel to obtain the `cx` value (formats vary, so do not assume a fixed pattern), then generate an API key in the console.

A successful call returns `200 OK` with a JSON object carrying three things: metadata about the request, metadata about the engine, and the results themselves. The `items` array holds the actual results with URL, title and text snippet, plus rich snippet data where the page supports it. Paging metadata lives in `queries`, which names each object after an OpenSearch query role. `request` is always present as a one-element array. `nextPage` disappears on the last page, `previousPage` on the first.

There is also a `callback` parameter that makes the endpoint JSONP-friendly, so you can drop a script tag into an HTML page and render results with no server-side code at all. Useful for a prototype, less useful once you need to store anything.

## The two limits that decide whether it fits your project

The first is depth. The API returns up to the first 100 results only. That is a total cap, not a per-page cap. If your use case is rank tracking across the top 100, you are exactly at the boundary. If it is competitive research that wants 300 results per query, or anything that paginates deep into a result set, this endpoint cannot do it at any price.

The second is volume plus lifetime. Google's own documentation now states that the published pricing applies only to existing Custom Search JSON API customers until the service discontinuation on January 1, 2027, and that the API is not available for new customers. Read the ceiling in that light: 10,000 queries per day works out to a maximum of about 300,000 queries per month, and that maximum expires.

At $5 per 1,000 queries, the per-query price is genuinely low. It is also a price you probably cannot buy at, and it comes attached to a shutdown date. That combination makes the official API a migration problem for existing users rather than a foundation for new work.

## How ScraperAPI returns Google results

ScraperAPI's structured Google endpoint takes a POST with a JSON body rather than query-string parameters. You send your `apiKey` and either a single `query` or a `queries` array for a batch, and it returns a parsed JSON object rather than raw HTML you have to write selectors against.

The parameters that matter for search work:

- `country_code` and `tld` for geography. The TLD list covers 20 Google domains including `com`, `co.uk`, `ca`, `de`, `es`, `fr`, `it`, `co.jp`, `in`, `com.au`, `com.br` and `com.mx`. Scraping google.com from another country requires both `tld` and `country_code` to be set.
- `hl` for host language and `gl` to boost results from a specific country.
- `start` for the starting result position, which is how you get past page one.
- `output_format` for JSON (the default) or CSV.
- `callback` with a webhook URL, so long batches post results to your endpoint instead of making you poll.

Google-specific parameters carry through as well: `UULE` for a named region, and `TBS` for a time window, with `tbs=h`, `d`, `w`, `m` or `y` limiting results to the past hour, day, week, month or year. There is no `cx` to configure, because there is no Programmable Search Engine in the loop.

## What one Google request costs you in credits

This is the number that determines your bill, and it is easy to miss. ScraperAPI prices a standard page at 1 credit. Amazon costs 5. Google and Bing, including their subdomains, cost 25. LinkedIn costs 30. Requests that go through bot protection such as Cloudflare, Datadome or PerimeterX add 10 credits on top when the bypass is used.

So one Google SERP request is 25 standard pages. The free tier's 1,000 credits buys about 40 Google searches, not 1,000. A keyword list of 10,000 terms checked once a month consumes 250,000 credits before you add retries, deeper `start` positions or a second locale. If you want to estimate a specific URL before committing, the dashboard has a Domain Cost Estimator, and you can set `max_cost` on a request to cap what any single scrape is allowed to burn.

Two behaviours are worth knowing before you scale. Exceeding your plan limits returns a 429 asking you to bring concurrency back within your allowance; the request is refused rather than queued, and there is no overage charge. Annual plans can also renew early to pull a fresh batch of credits.

## Full plan comparison

ScraperAPI publishes eight tiers. Prices and per-plan credit allowances are shown live on the pricing page and change with billing cycle, so the table below sticks to what is fixed: the plan names, what happens when you hit your limit, and the type of workload each tier suits.

| Plan | At the credit limit | Who it fits | Start |
| --- | --- | --- | --- |
| Free: 1,000 credits, 5 concurrent connections | Nothing; requests are refused. Extra test credits available on request | Confirming the Google endpoint returns the fields you need before spending | [Open the free tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Auto-upgrade to the next plan when credits run out | A single keyword list on a weekly or monthly schedule | [See the Hobby plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Auto-upgrade | Several lists running on a recurring schedule | [See the Startup plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | Auto-upgrade | Multiple projects or people sharing one credit pool | [See the Business plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Auto-upgrade or pay-as-you-go up to a monthly cap you set | Spiky SERP volume you do not want to re-plan for | [See the Scaling plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Auto-upgrade or pay-as-you-go | Sustained volume with occasional campaign bursts | [See the Professional plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | Pay-as-you-go only | Continuously running collection, budgeted per credit | [See the Advanced plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Pay-as-you-go only | Committed monthly volume, procurement and invoicing | [Talk to ScraperAPI about Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Pay-as-you-go keeps you scraping past your limit at a fixed per-credit rate without changing plans, up to a monthly spending cap you control. It is available on Scaling, Professional, Advanced and Enterprise. Auto-upgrade is opt-in and covers Hobby, Startup, Business, Scaling and Professional. The difference matters for budgeting: auto-upgrade changes your monthly commitment, pay-as-you-go does not.

## Picking a tier from your query count

Work backwards from Google queries per month, then multiply by 25 to get the credit figure you need. That single conversion handles most of the decision.

If you are validating the endpoint or running a one-off export of a few hundred keywords, the free tier covers it and you will learn whether the returned JSON has the fields your pipeline needs. Move off it when the 5 concurrent connections become the bottleneck rather than the credits.

For a recurring keyword set in the low thousands per month, Hobby or Startup is the right shape, with auto-upgrade on so a bigger-than-expected month rolls into the next tier instead of returning 429s mid-run. Teams running several properties, or an agency billing clients for rank data, usually land on Business or Scaling because the credit pool is shared and nobody has to arbitrate whose job gets the last credits.

Once collection runs continuously, the pay-as-you-go tiers do more work than a bigger allowance. Scaling, Professional, Advanced and Enterprise let you set a monthly spend cap and keep going at a fixed per-credit rate. If your monthly Google volume swings by a factor of three, that flexibility is worth more than a marginally better per-credit price on a fixed plan. 👉 [Compare the plans against your monthly Google query count](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The other routes, and what they actually return

Gemini's Grounding with Google Search is the option people ask about most, and it solves a different problem. You pass `tools: [{"type": "google_search"}]` on an Interactions request and the model runs the searches itself, then answers with inline `url_citation` annotations containing the URL, title and character offsets. The response also includes a `google_search_call` step listing the queries actually executed and a `google_search_result` step with an HTML snippet for rendering suggestions.

The billing is the part that catches people out. On Gemini 3 and newer models you are billed for each search query the model decides to execute, not per API call. A single prompt that triggers two searches counts as two billable uses. Empty queries are ignored. There are 5,000 free search requests per month shared across all Gemini 3+ models, then $14 per 1,000 search requests. The Paid Tier default is 1,500 requests per day per project, raisable on request. Gemini 2.5 and earlier models still bill per prompt.

Compare that $14 per 1,000 against Google's $5 per 1,000 and the trade is clear: you are paying for synthesis and citations, not for a result list. If your downstream code needs `items[7].link`, Gemini gives you prose with sources attached. If your downstream reader is a person who wants an answer with links they can check, it is the better fit.

SerpApi occupies the structured-list side. One GET endpoint, `q` and `api_key` required, `engine` defaulting to `google`, returning structured JSON or Markdown. Its engine coverage is the widest of the three, spanning Maps, Maps Reviews, Shopping, Images, Videos, News, Autocomplete, Local, Jobs, Scholar, Trends, Flights, Lens, Events, Hotels, Reverse Image, Finance, Patents, plus AI Overview and AI Mode. Client libraries exist for Python and JavaScript, along with no-code integrations for automation platforms.

Apify's Google Search Scraper runs as an Actor. Inputs include multiple `queries`, `resultsPerPage`, `maxPagesPerQuery`, `countryCode`, `languageCode`, `mobileResults`, `maxConcurrency`, `saveHtml`, `includeUnfilteredResults` and a `customDataFunction` hook. Output covers organic results, paid ads, AI overviews, People Also Ask, related queries, product ads with prices, ratings and review counts, and business lead fields such as name, work email, phone number, job title and LinkedIn profile. Exports run to JSON, CSV, Excel, XML, HTML, JSONL and RSS, and Apify Proxy is included on every plan including the free one. Neither vendor's pricing is published in a form we could verify here, so compare those directly before deciding.

## FAQ

**Can I still sign up for Google's Custom Search JSON API?**
No. Google's documentation states the API is not available for new customers, and the published pricing applies only to existing customers until the January 1, 2027 discontinuation.

**How do I get more than 100 results from the official API?**
You cannot. The API returns up to the first 100 results only, regardless of how you page. Deeper coverage requires a different data source.

**Is $5 per 1,000 queries expensive?**
Per query it is cheap. The constraint is the 10,000 queries-per-day ceiling and the fact that the service ends. Budgeting around a price you cannot buy at is the actual risk.

**Can Gemini's search grounding replace a SERP API?**
Only if you wanted an answer rather than a result list. It returns text with citation annotations, billed per executed query at $14 per 1,000 after 5,000 free requests per month, and it will not give you structured organic results with positions.

**Do I need a Programmable Search Engine to use ScraperAPI's Google endpoint?**
No. The endpoint takes your API key and a query, with `country_code`, `tld`, `hl`, `gl` and `start` for targeting. There is no `cx` value to create or maintain.

**What happens if I run out of credits mid-month?**
On Hobby, Startup, Business, Scaling and Professional, auto-upgrade moves you to the next plan. On Scaling, Professional, Advanced and Enterprise, pay-as-you-go continues at a fixed per-credit rate up to your monthly cap. Otherwise requests pause until credits renew, and annual plans can renew early for a fresh batch.

## What to do next

If you already have Custom Search JSON API access and a working integration, keep it running and start moving the data layer now rather than in late 2027. The 100-result cap and the 10,000-per-day ceiling mean the migration is really about depth and volume, not about the endpoint URL.

If you are starting fresh, the choice comes down to output shape. Structured Google results with position data: pick a SERP provider and budget at the credit rate, which for ScraperAPI means 25 credits per Google request. Prose answers with citations: Gemini's grounding is the cheaper design for that job at $14 per 1,000 executed queries.

Before you enter a card number, confirm two things. First, your real monthly Google query count multiplied by 25, so the plan you pick is sized against credits rather than page views. Second, whether you want auto-upgrade (a higher monthly commitment, no failed requests) or pay-as-you-go (a fixed rate and a cap you set), because the two are available on different tiers.

👉 [Start with the 1,000 free credits and size your plan from there](https://www.scraperapi.com/pricing?fp_ref=coupons)
