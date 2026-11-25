# Tripadvisor scraper API: how much does a full review pull cost, which fields do you actually get, and which plan covers it? (credit math + official API limits)

Tripadvisor's official Content API returns roughly five reviews and five photos per location, according to Apify's February 2026 guide to the program, which is nowhere near a full review history for a city. A scraping API closes that gap, and with ScraperAPI the cost question has a concrete answer: a standard page request costs 1 credit, so the $49/month Hobby plan's 100,000 credits buy 100,000 page fetches before any add-ons.

What follows is the full plan lineup with the prices published on the pricing page, the specific limits of the official Content API, the field-level differences between hotel, restaurant and attraction records, and a credit estimate you can run against your own list size.

> **Most people should start on Hobby:** 100,000 credits a month for $49 ($44.10/month billed annually at $529.20/year), 20 concurrent threads, US and EU geotargeting. That covers up to 100,000 standard page fetches, so a 20-city pull with 600 properties each uses about 12% of the allowance, as long as you aren't paying the rendering and premium-proxy multipliers on every request.
> 👉 [See current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans side by side

| Plan | Credits per month | Concurrent threads | Price | Get it |
| --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | $49/mo, or $44.10/mo annually ($529.20/yr, save $58.80) | [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | Quoted on pricing page | [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Quoted on pricing page | [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Quoted on pricing page | [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Quoted on pricing page | [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 (+500K bonus credits, limited offer) | 500 | $1,975/mo, or $1,777.50/mo annually ($21,330/yr, save $2,370) | [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom | [Talk to ScraperAPI about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Hobby and Advanced are the two list prices published on the pricing page; Startup through Professional sit between them and are quoted on that same page.

Two things change as you move up beyond the credit number. Geotargeting is US and EU only on Hobby and Startup, and becomes country-level from Business upward. Analytics history is capped at the last 30 days on Hobby and Startup and unlimited from Business up, which matters if you're tracking rating drift across quarters rather than pulling a one-off dataset.

## What the official Content API won't give you

According to Apify's February 2026 write-up, the Content API is built for partners who display Tripadvisor content inside their own product, not for exporting whole city lists. Its published constraints:

- Only a small subset of website content is exposed through the API at all.
- Review and photo endpoints typically return up to 5 reviews and 5 photos per location.
- New developers get around 5,000 free API calls per month, but billing information is required upfront.
- Usage is capped by a daily budget you configure in advance; once you hit it, requests are blocked.
- The search endpoint is limited to roughly 10,000 requests per day.
- One API key per account, which gets awkward the moment two teams or two projects need separate quotas.

Those figures come from a third-party summary rather than Tripadvisor's own documentation, so treat them as directional. The shape of the problem holds either way: a competitor analysis of 500 Boston restaurants needs 500 review sets, and five reviews per property is not a dataset.

A scraping API takes the other route. You send the same public URLs a browser would open, and you get the HTML or parsed JSON back. ScraperAPI charges per request rather than per record, which is the number you need to model before picking a tier.

## The fields are not uniform across property types

This is the part most comparison posts skip, and it decides whether a scraping setup is even usable for your project. Details below are drawn from the published field documentation of three Tripadvisor-specific tools.

**Review fields that only exist on some pages.** Helpful votes on a reviewer's contributions appear on hotel review pages and not on restaurant ones. If your scoring model weights community endorsement, it will work for hotels and silently return nothing for restaurants.

**Subratings differ by category.** Hotel reviews carry value, rooms, location, cleanliness, service and sleep quality. Restaurant reviews carry value, service, food and atmosphere. Any category the reviewer skipped comes back empty, so a five-star restaurant review can still have zero subratings attached.

**Attraction list pages are a known gap.** One Tripadvisor actor explicitly returns an `unsupported_surface` error for attraction list pages, meaning you can't page through "things to do in Lisbon" the way you can restaurants or hotels. You either feed it detail URLs directly or discover attractions another way. Budget for that before you promise a client an attractions dataset.

**Tripadvisor lists one phone number and one website per property.** That's the stated reason one open-source scraper added a separate crawl of each property's own site to pull emails, social profiles and the tech stack behind the site, tagging each result with the source page and an `is_likely_official` flag. If your outreach list needs a contact name or an email, Tripadvisor alone won't produce it.

**Review text, dates and owner replies are all available.** Full review body, headline, star rating out of five, author name and location, contribution count, stay date, trip type, publication date, photos, and the property's owner response with its signatory and date. That's enough to build sentiment and response-rate analysis without touching the official API.

**Room rates are date-dependent.** One tool exposes `check_in_date`, `check_out_date`, `rooms` and `adults` alongside hotel listings, and returns a `price` plus a `price_range` in the currency you specify. Those numbers change with the stay dates you send, so store the inputs with the output or your price history is unreadable six months later.

## Estimating your real credit cost

ScraperAPI's unit is the request, not the row. The base rate is 1 credit per standard page, with multipliers stacked on top:

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX, Akamai) | 10 |
| JS rendering (`render=true`) | 10 |
| Premium proxies (`premium=true`) | 10 |
| Premium proxies + JS rendering | 25 |
| Ultra Premium | 30 |
| Ultra Premium + JS rendering | 75 |

Geotargeting, sessions, device type, output format and auto parsing add nothing.

Now run your own numbers. Tripadvisor list pages hold 30 places each, so 600 restaurants is 20 list requests, which is noise. The detail pages are what cost money.

Take 20 cities at 600 properties each: 12,000 detail pages.

- Plain HTML at 1 credit: 12,000 credits, roughly 12% of Hobby.
- With JS rendering on every page at 11 credits: 132,000 credits, which busts Hobby's 100,000 and pushes you to Startup's 1,000,000.
- With premium proxies plus rendering at 25 credits: 300,000 credits, still inside Startup.

Add review pagination on top. If each property's reviews need several requests to page through, multiply the detail-page count by pages per property before you pick a tier. A 12,000-property pull at three review pages each is 36,000 base requests, not 12,000.

You don't have to guess the multiplier for a specific URL. The dashboard's Domain Cost Estimator quotes the cost before you scrape, every response returns its charge in the `sa-credit-cost` header, and `max_cost` caps any single request.

👉 [Check what a Tripadvisor URL costs before you commit credits](https://www.scraperapi.com/?fp_ref=coupons)

## Where a general scraping API stops and a dedicated actor starts

ScraperAPI fetches pages. It does not know what a Tripadvisor review is, so you write and maintain the parser, the pagination logic and the retry rules. That's the trade: you own the code, you get every field the page exposes, and you aren't limited to the fields a vendor chose to model.

Dedicated Tripadvisor tools come at it from the other end. Apify's `s-r/tripadvisor-scraper` returns places and reviews in one dataset with a `row_type` column to separate them, handles up to 600 places or 200 detail pages per run, defaults to 60 places, and lets you set `max_reviews_per_url` to 0 for every review on the page or a cap between 1 and 50. It bills per row with no unit price published.

Apify's `blackfalcondata/tripadvisor-scraper` adds keyword and radius search (`lat`, `lon`, `radiusKm`), `maxResults` where 0 means unlimited, `maxReviewsPerPlace` defaulting to 20, and an incremental mode the listing claims saves 80–95% on weekly syncs. That mode only watches the top-N ranked results, so a new restaurant that enters the city at rank 400 won't show up until it climbs. It also ships notifications to Slack, Telegram, Discord, WhatsApp and generic webhooks. No published price either.

The open-source option is `omkarcloud/tripadvisor-scraper`, MIT licensed, Python, self-hosted, 214 stars and 37 forks on GitHub, first published June 2023 with no tagged releases. It splits search, list, detail and review endpoints, pages 30 results at a time, filters reviews by rating, date, traveler type and keyword, and adds a cruise line covering 19 destinations with per-cabin pricing across vendors. It also has the website-contact crawl described earlier.

None of the three publishes a per-unit rate, so there's no honest way to rank them against ScraperAPI on price alone. Compare them on work: a dedicated actor saves you a parser, a general API saves you a vendor's field choices, and self-hosting saves you the subscription at the cost of running the infrastructure.

## Who shouldn't pay for this

If you need 50 reviews from one hotel for a single piece of analysis, skip the paid tiers. ScraperAPI's help content has referenced 1,000 free credits on signup (5 concurrent connections), which covers a small pull; check the dashboard for what's currently on offer, since the pricing page now leads with Hobby. The MIT scraper is also free to self-host if you're comfortable with Python and don't mind maintaining it.

If your output has to be a clean structured record with no parsing work, a Tripadvisor-specific actor gets you there faster than any general API will.

If you're collecting contacts for outreach rather than reviews, the constraint is Tripadvisor's one-phone-one-website limit, not your scraping budget. No tier of any API fixes that.

👉 [Start on Hobby at $49/month if your pull is under 100,000 standard pages](https://www.scraperapi.com/?fp_ref=coupons)

## Running long jobs without burning the allowance

Three mechanics are worth knowing before you schedule anything recurring.

Concurrency limits return a 429 rather than billing you extra. The request is turned away, not queued, so a burst of parallel jobs doesn't quietly cost more; it just fails and needs retrying.

When credits run out, scraping pauses until they renew. On Hobby, Startup, Business, Scaling and Professional you can switch on auto-upgrade to move to the next tier automatically. From Scaling up through Enterprise, pay-as-you-go keeps requests flowing at a fixed per-credit rate up to a monthly cap you set. Annual plans can also renew early for a fresh batch of credits.

Set `max_cost` on every scrape where the page type varies. A single request that silently lands on Ultra Premium with rendering costs 75 credits, and 1,300 of those empties Hobby on their own.

## Which plan fits your case

Pulling reviews for a few hundred properties once, with pages that come back as plain HTML: Hobby at $49/month, or the annual rate at $44.10/month if you know you'll keep it.

Running the same pull with rendering or premium proxies on most requests: multiply your page count by 11 or 25 first. Under 1,000,000 credits you're on Startup; above that, Business or Scaling.

Weekly competitor monitoring across dozens of cities, where credits are consumed continuously rather than in bursts: the pay-as-you-go option starting at Scaling is what keeps a spike from stalling the pipeline mid-month.

Before you check out, confirm two numbers. First, run one real Tripadvisor URL through the Domain Cost Estimator, because that tells you whether you're on the 1-credit path or a 10x path, and that single answer decides your tier. Second, decide what happens when credits run dry: auto-upgrade means your bill moves up a step, pay-as-you-go means it moves up per credit until your cap. Both are opt-in, and neither is reversible mid-cycle.

👉 [Compare the full ScraperAPI lineup and pick your credit tier](https://www.scraperapi.com/pricing?fp_ref=coupons)
