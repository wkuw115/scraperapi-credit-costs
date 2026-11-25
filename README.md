# Headless Browser API: What a Rendered Page Really Costs, How Far the Free Tier Goes, and When ScraperAPI Is the Wrong Buy (Credit Costs and Full Plan Table)

A headless browser API turns one HTTP request into what a real browser would have produced: rendered HTML, a screenshot, structured JSON. You don't install Chrome, and you don't keep a browser fleet alive. The provider starts a browser, loads your page, hands back the output, and closes the session.

ScraperAPI sells that capability as a flag on a scraping request rather than as a separate browser endpoint. Adding `render=true` routes the request through a Chromium instance before the HTML comes back, and it costs **10 credits per request** on every plan, free plan included. That one number answers most of the buying questions people actually have: how far the free tier stretches, what a monthly bill really buys, and whether a dedicated browser API would be cheaper for your workload.

Below: the full plan ladder with the prices ScraperAPI publishes, the credit costs that determine your real bill, the concurrency limit that isn't in the plan table, and the cases where a browser-first product fits better.

> **Quick pick:** start on the free plan (1,000 credits a month, 5 concurrent threads, no card) and send ten rendered requests to see what your target sites actually cost. If you then need roughly 100,000 rendered pages a month, that's Startup: **$149/month, or $134.10/month billed annually ($1,609.20/year, saving $178.80)**. If you need to drive a page yourself with Playwright for more than a minute, or you need PDFs from arbitrary HTML, read the comparison section first.
> 👉 [Compare ScraperAPI plans and start with the free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)

## ScraperAPI plans and prices

| Plan | Credits per month | Concurrency | Price | Billing | Extras |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 5 threads | $0 | Monthly | No credit card required |
| Hobby | Not published | Not published | Not published | Monthly | Auto-upgrade eligible |
| Startup | 1,000,000 | 50 threads | $149/mo, or $134.10/mo annual ($1,609.20/yr) | Monthly or annual | US & EU geotargeting; auto-upgrade |
| Business | Not published | Not published | Not published | Monthly | Auto-upgrade eligible |
| Scaling | Not published | Not published | Not published | Monthly | Auto-upgrade; pay-as-you-go |
| Professional | Not published | Not published | Not published | Monthly | Auto-upgrade; pay-as-you-go |
| Advanced | Not published | Not published | Not published | Monthly | Pay-as-you-go |
| High-volume / Enterprise | 22,000,000+ | 500+ threads | Custom | Contract | Dedicated support, Slack channel, volume discounts |

👉 [See the current plan ladder and pick a tier](https://www.scraperapi.com/pricing?fp_ref=coupons)

Two things about that table. The plan names from Hobby through Advanced are ScraperAPI's own, taken from its billing FAQ, where they appear in the auto-upgrade and pay-as-you-go rules. Their individual allowances and prices aren't published in the pricing material available, so the cells stay empty rather than filled with a guess. Only the Free and Startup figures are stated outright.

The second thing: the free plan and the trial are different offers. New accounts get a **7-day trial with 5,000 credits**, no credit card. When it ends you drop to the free plan's **1,000 credits a month and 5 concurrent threads**. If you're benchmarking a rendering workload, do it inside the trial week, when you have five times the credits.

## The credit table that decides your bill

ScraperAPI doesn't price per browser-second or per gigabyte. Every request has a credit cost, and the features you switch on stack.

| What you send | Credits per request | Notes |
| --- | --- | --- |
| Standard page | 1 |  |
| E-commerce (Amazon, Walmart, eBay) | 5 |  |
| SERP (Google, Bing, including subdomains) | 25 |  |
| Social (LinkedIn) | 30 |  |
| Bot-protection bypass | 10 | Cloudflare, Turnstile, DataDome, PerimeterX/HUMAN, Akamai |
| JS rendering (`render=true`) | 10 | Available on all plans |
| Premium proxies (`premium=true`) | 10 |  |
| Screenshot (`screenshot=true`) | 10 |  |
| Premium proxies + JS rendering | 25 |  |
| Ultra premium (`ultra_premium=true`) | 30 | Paid plans only |
| Ultra premium + JS rendering | 75 | Paid plans only |
| Geotargeting, sessions, device type, output format, auto parsing | 0 | No extra cost |

Run the arithmetic on your own pipeline before you pick a tier:

- 1,000 free credits = **100 rendered requests**, or 1,000 static pages.
- 1,000,000 Startup credits = **100,000 rendered requests** at 10 credits each, or 40,000 requests if every one of them also needs premium proxies and rendering (25 credits).
- Screenshots cost the same 10 credits as rendering, so 1,000 credits buys 100 screenshots.
- The 75-credit ultra-premium-plus-rendering combination is the one to avoid unless a target site genuinely refuses everything else. At that price, 1,000,000 credits is about 13,300 requests.

The pattern worth internalising: rendering multiplies your effective cost by ten, and proxy tier multiplies it again. ScraperAPI's own documentation recommends using rendering only when you need it, because it adds latency and can lower success rates, which reduces how many requests you get through. Turning it on globally "just in case" is how people end up buying two tiers above what they need.

Two tools keep this under control. Every response carries an `sa-credit-cost` header showing what that request consumed, and you can set `max_cost` per scrape so one expensive URL can't quietly drain the balance. The dashboard's Domain Cost Estimator prices any URL before you send traffic at it.

👉 [Start on the free plan and price your own URLs](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How you actually turn the browser on

Three ways, all documented, all costing the same 10 credits:

1. Add `render=true` to the request as a query parameter.
2. Send an `x-sapi-render: true` header.
3. Go through the proxy port and put the instruction in the username, as `scraperapi.render=true`, with your API key as the password.

If the page paints its content late, add `wait_for_selector` to hold the response until a specific element appears. It only works alongside `render=true`; on its own it's ignored. ScraperAPI's docs also note that rendering doesn't load everything on a page by default, deliberately skipping images and tracking scripts to keep requests fast, which is exactly why `wait_for_selector` earns its place when a value you need arrives after the first paint.

There's also a rendering instruction set for interaction: click, type, scroll, and wait for events before the response comes back. That covers form-driven flows and "click Load More twice" pages without you writing a Puppeteer script.

## The limit that isn't in the plan table

Rendered requests have their own throughput ceiling, separate from your plan's thread count. ScraperAPI sets rendering concurrency at a **default burst limit of 10 requests per second**. The docs spell out the implication: if each rendered request takes 25 seconds, sustained sending at 10 requests/second means up to 250 rendered requests running at once. Raising that burst limit is an Enterprise conversation.

So a 50-thread Startup plan and a 500-thread Enterprise plan don't differ only in raw parallelism. If your workload is render-heavy, the 10 req/sec ceiling shapes your throughput more than the thread count does, and you should test against it during the trial rather than discovering it in production.

## The anti-bot fork in the road

This is where the headless browser API market splits, and the split is a policy choice, not a technical one.

ScraperAPI charges **10 credits per request** to bypass Cloudflare, Cloudflare Turnstile, DataDome, PerimeterX/HUMAN, or Akamai. Bypassing is a product feature with a price tag.

Cloudflare, which sells its own browser product, takes the opposite position. Its "Well-behaved Bot Mode" identifies the browser as a bot using cryptographic signatures and advertises that it does not bypass bot protections, positioning that as an ethical stance against competitors who do. Browserless sits on ScraperAPI's side of the line: its `/unblock` endpoint exists to get past CAPTCHAs and bot detection, and its own docs recommend residential proxies alongside it.

What each choice costs you:

- **Bypass-capable route (ScraperAPI, Browserless):** you get content from sites that actively resist scraping. You pay 10 extra credits per request on ScraperAPI, and you carry whatever risk the target site's terms attach to that.
- **Signed-bot route (Cloudflare):** the request announces itself, which is cleaner for sites that allow declared crawlers. Against a site with hard anti-bot protection, you may simply not get the page.

Neither position is wrong for every reader. If you're scraping your own properties, price feeds you're licensed to use, or sites that tolerate crawlers, the cheap path is the standard page at 1 credit. The 10-credit bypass premium only makes sense against sites you've decided you need.

## Where a dedicated browser API beats ScraperAPI

ScraperAPI's unit is the request. Two competing products sell a different unit: browser time and browser sessions.

**Browserless** runs a REST endpoint per task: content, CSS-selector scraping, unblock, PDF, screenshot, export, download, server-side Puppeteer functions, and Lighthouse performance audits. Region-specific hostnames in San Francisco, London, and Amsterdam let you open the browser near your targets. Its most interesting detail for developers is `/unblock` with `browserWSEndpoint: true` and a `ttl` of up to 60 seconds: you get a live WebSocket endpoint with the page's cookies, connect your own Puppeteer or Playwright script, and continue from there. That 60-second window is the bridge between a REST call and full scripted control. There's a free plan with no credit card, and no published numbers on the free allowance.

**Cloudflare Browser Run** (renamed from Browser Rendering; older docs and API paths still use the `browser-rendering` name) splits into stateless Quick Actions and stateful Browser Sessions. Quick Actions cover Markdown, screenshots, PDFs, snapshots, links, element scraping, structured JSON, and asynchronous multi-page crawling, each in a single HTTP request with no code deployed. Browser Sessions give you Puppeteer, Playwright, CDP, or Stagehand inside Workers or from anywhere via CDP, with session reuse and Durable Objects to cut cold starts. Billing is by browser time used, with a free tier on both Free and Paid plans.

Pick against your job, honestly:

- **You need rendered HTML or a screenshot of a known URL, at volume, behind rotating proxies.** ScraperAPI's per-request model is easy to forecast, and proxy, geotargeting, sessions, and parsing don't add cost.
- **You need pixel-accurate PDFs from arbitrary HTML, headers and footers included.** Browserless and Cloudflare both list PDF endpoints with page-size and margin options. PDF generation isn't part of the ScraperAPI feature set verified here.
- **You need your own Playwright script to stay on a page, log in, paginate, and download files.** You want a CDP or WebSocket endpoint and a session that outlives one request. Browserless gives you 60 seconds of takeover; Cloudflare gives you reusable sessions.
- **You want to avoid starting a browser at all.** Browserless Smart Scrape starts with a fast HTTP fetch and escalates only when a page demands it. ScraperAPI's equivalent lever is leaving `render=true` off until a site proves it needs it.

## Two names that cause expensive confusion

"Browserless" refers to two unrelated things. The commercial hosted API has years of production history, large Docker adoption, and SOC 2 Type II. Separately, `microlinkhq/browserless` is an MIT-licensed open-source Puppeteer driver with a few thousand GitHub stars, a CLI, and an optional cloud service pointing at a different vendor's API. Nothing in the material reviewed shows any relationship between them. Don't credit the commercial product's compliance credentials to the open-source library, or vice versa.

The Cloudflare rename is the second trap. Documentation pages now dated 2026 use the Browser Run name, while the product page and example request paths still say `browser-rendering`. Same product, two strings; searching for the old one still finds current documentation.

## How to decide

Work backwards from what your pipeline actually does with the response.

- **Static pages, mostly.** Skip the browser entirely. Standard requests at 1 credit, 1,000 free credits a month to validate, and ScraperAPI's per-request pricing is the cheapest shape available.
- **JavaScript-heavy SPAs where you parse the DOM yourself.** `render=true` at 10 credits. Budget a 10x multiplier against your static-page estimate, and use `wait_for_selector` instead of cranking up global rendering.
- **Sites behind Cloudflare, DataDome, or PerimeterX.** Add 10 credits per request and decide whether your use case justifies it. If it doesn't, run the same workload against a signed-bot product and accept that some pages won't come back.
- **You need to script a browser session.** Choose on session length and control surface, not on endpoint count. A 60-second takeover, reusable cloud sessions, and an instruction set of clicks and scrolls solve different problems.

Two things to confirm before you enter a card number. First, what happens when credits run out: by default scraping pauses until they renew, auto-upgrade is opt-in and available on Hobby, Startup, Business, Scaling, and Professional, and pay-as-you-go is limited to Scaling, Professional, Advanced, and Enterprise, capped at a monthly maximum you set. Second, whether annual billing fits: annual plans can renew early for a fresh credit batch, and Startup's annual rate saves $178.80 over twelve monthly payments, but that's only a saving if you're still running the same volume in month twelve.

Test in the trial week, price your ten hardest URLs, then buy the tier the arithmetic points to.

👉 [Start the 7-day trial with 5,000 credits, no card required](https://www.scraperapi.com/?fp_ref=coupons)
