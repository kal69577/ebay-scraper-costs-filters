# eBay Scraper: Why Do Your Filters Fail Silently, How Many Requests Does 1,000 Listings Take, and What Will It Cost? (Full Plan Table)

eBay's markup is the easy part of an eBay scraper. The two things that actually sink projects are the anti-bot layer, and the search filters eBay throws away without returning an error, so a mistyped parameter hands you unfiltered results while charging you for every page. If you'd rather rent that plumbing than maintain it, ScraperAPI handles the fetching and prices eBay in its e-commerce bucket at **5 credits per request**, with 1,000 free credits and 5 concurrent connections to test the math before you pay anything.

Below: what those credits buy at realistic volumes, the eBay parameters that quietly do nothing, and how the plan tiers differ once your allowance runs out.

> **Quick verdict:** Start on the free tier to measure what one of your own eBay URLs actually costs, then buy the smallest paid tier whose allowance covers your monthly request count. A daily detail sweep across 1,000 listings burns roughly 150,000 credits a month, so price-monitoring teams usually land well above the entry tiers. 👉 [Check current tier allowances and per-credit pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Every ScraperAPI plan, and what actually changes between them

Three things separate the tiers: how many credits you get, the price per credit (larger allowances usually come at a better rate), and what happens when you hit the ceiling. ScraperAPI revises allowances and prices periodically, so the table links to live numbers rather than quoting figures that go stale. The Free tier is fixed and verifiable: 1,000 credits, maximum 5 concurrent connections.

| Plan | What happens when you run out | Price | Sign up |
| --- | --- | --- | --- |
| **Free** | Trial allowance only; 5 concurrent connections is the hard cap | $0 | [Claim the 1,000 free credits](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Hobby** | Scraping pauses until credits renew; auto-upgrade can bump you to the next tier | On the pricing page | [See Hobby pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Startup** | Pause until renewal, or opt into auto-upgrade | On the pricing page | [See Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Business** | Pause until renewal, or opt into auto-upgrade | On the pricing page | [See Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Scaling** | Auto-upgrade, or pay-as-you-go at a fixed per-credit rate | On the pricing page | [See Scaling pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Professional** | Auto-upgrade, or pay-as-you-go up to a monthly cap you set | On the pricing page | [See Professional pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Advanced** | Pay-as-you-go, capped at a monthly limit | On the pricing page | [See Advanced pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Enterprise** | Pay-as-you-go, capped; custom terms | Custom | [Talk to sales or compare tiers](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two mechanics matter more than the plan names. Going over your concurrency limit returns a 429 and the request is turned away rather than queued, with no extra charge for the attempt. And running out of credits is not a surprise bill by default: scraping pauses until renewal unless you have deliberately switched on auto-upgrade (Hobby through Professional) or pay-as-you-go (Scaling and above). Annual plans can renew early for a fresh batch of credits if you need to keep a job running mid-cycle.

The pay-as-you-go distinction is the one people miss. On Hobby, Startup and Business, the only way past your ceiling is jumping to the next plan. If your eBay workload is spiky, a monthly auction-season surge on top of a steady baseline, that jump is exactly the wrong shape.

## 1,000 eBay listings: requests, credits, and the two cost tiers

eBay search returns 60 results per page by default unless you set `_ipg`, and you get the total result count from the `.srp-controls__count-heading` element, which is what lets you compute page count instead of guessing. So 1,000 listings is about 17 search requests. Item pages are one request each. Product descriptions are a separate document, because eBay renders them inside an iframe (`iframe#desc_ifr`), so if you want the seller's text rather than a URL, that's another request per item.

At 5 credits per eBay request, the spread looks like this:

| What you collect | Requests | Credits |
| --- | --- | --- |
| 1,000 listings from search pages only | 17 | 85 |
| 1,000 listings plus each item page | 1,017 | 5,085 |
| Plus seller descriptions from the iframe | 2,017 | 10,085 |
| Item pages behind a bot-protection bypass (+10 credits each) | 2,017 | 20,085 |

The last row is the one that breaks budgets. ScraperAPI's documentation adds 10 credits per request when it has to bypass Cloudflare, Datadome, PerimeterX or Akamai. eBay sits behind anti-scraping protection, so 1,000 item pages can cost three times the base rate on a bad day. The same math applies to eBay's regional sites, and you can steer the localized view (currency, shipping quotes) with `country_code`, which is not on the extra-cost list.

Search-only scraping is cheap enough to do casually: 85 credits for a thousand rows. The moment you want item specifics, gallery images or stock levels per listing, you are in the 5,000 to 20,000 credit range for the same thousand products.

Daily monitoring multiplies it. A nightly detail refresh of 1,000 listings is 5,000 credits a day, about 150,000 a month, before any bypass surcharge. That number should decide your tier, not the headline credit count on the pricing page.

Before you commit to a figure, price the exact URL. The dashboard's Domain Cost Estimator, the `/account/urlcost` API endpoint, and the `sa-credit-cost` response header all report the real cost of a request, and the `max_cost` parameter lets you cap any single scrape so a pathological URL can't drain the account. 👉 [Run your eBay URL against the 1,000 free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Five ways eBay scraping goes wrong without telling you

These are eBay behaviours, not tool behaviours, and they hit API users and self-hosted spiders equally.

**1. eBay silently drops filters it doesn't recognise.** A misspelled parameter does not throw an error; it returns the unfiltered result set and you pay for it. The language trap is nastier than the typo trap. On ebay.com, `Brand=Nike` put "Nike" in 212 of 213 titles, against 9 of 213 without the filter. On ebay.de, `Brand=Nike` does nothing; you need `Marke=Nike`, which hit 210 of 211. Same concept, different key, and no warning either way.

**2. A top-level category without a keyword returns zero results, every time.** `_sacat=1` (Collectibles) on ebay.com answers with no listings no matter how often you retry. Add `_nkw`, or use a subcategory. An unknown category ID fails the same way, which is easy to misread as "no inventory".

**3. `LH_PrefLoc` is ignored on mobile searches.** If your location filter looks like it's working but results keep coming from abroad, switch to `LH_LocatedIn=1&_salic=<code>`. The code 45 is China; 77 is the site's own country, which on ebay.de produced 212 of 212 domestic results against 21 of 212 foreign ones without it. Seller exclusion has a similar trap: `_saslop=2` alongside `_fss=1&LH_SpecificSeller=1&_sasl=...` is what removes sellers, and without it three sellers you meant to exclude occupied 37 of 213 rows.

**4. The description isn't on the item page.** You get a `description_url` pointing at the iframe document. One request per item, or no description text at all. This is also why "1,000 products" means two different request counts depending on whether anyone reads the copy.

**5. Variant data hides inside a script tag, and prices come in two flavours.** Colour and size options, per-variant stock and per-variant images live in a JavaScript variable containing `MSKU`, which normal DOM parsing won't touch. Worse for comparison sites, the price exists twice: an original value (`convertedFromValue` / `convertedFromCurrency`) and a converted value (`value` / `currency`). Mix the two fields across marketplaces and your price comparison is wrong in a way no exception will flag. Stock has the same shape: `outOfStock` plus a `NumberValidation.maxValue` purchase cap.

## Sold comps expire in roughly 90 days

Completed-sale data is the reason most people scrape eBay at all, and it comes with an expiry date. Sold records usually cover the last ~90 days, with an explicit window parameter available to tighten it. Sold and active listings are two separate scrapes with two separate bills; if you want sell-through rate, you need both counts, because the ratio is sold divided by sold plus active. Pricing your inventory off stale comps is a quiet way to lose money, so write the window into the request rather than filtering after the fact.

## Writing your own Scrapy spider versus paying per request

A public Scrapy-based eBay spider reports its own numbers: about 100 search listings per minute, roughly 30 fully detailed products per minute, over 95% field extraction success, and more than 90% success against anti-bot measures. It covers 80+ product fields, runs three pipelines (validation, cleaning, price normalization), rotates user agents, and depends on a third-party proxy service whose free tier covers around 1,000 requests, enough for development and nothing else.

Run the arithmetic on that 30-per-minute detail rate and 1,000 products is about 33 minutes of solid work on one machine, assuming nothing blocks you. Your alternative is buying concurrency and paying 5 credits per page request. What you don't escape by self-hosting is maintenance: when eBay changes a selector or tightens a challenge, the spider breaks and you fix it. When a bypass fails on the API side, someone else's on-call engineer fixes it.

The honest split: if your monthly need fits in a few thousand credits, search pages only, a weekly sweep, the entry paid tier is the whole answer and building anything is a waste of a weekend. Somewhere past 100,000 credits a month, self-hosting starts to look rational, provided you actually want to own a scraper. In between, it comes down to whether your time or the credit bill is the scarcer resource.

## Who shouldn't buy this

If you need a few dozen listings once a quarter, manual exports and eBay's own research tools will do, and a scraping subscription is overhead. If you plan to redistribute eBay data commercially or resell it as a dataset, sort out licensing and eBay's terms first; an API handles the fetching, not the rights. Even the open-source spiders carry an explicit "educational and research purposes only" disclaimer for this reason.

## Choosing a tier without guessing

Match the tier to your monthly credit burn, not to the plan names:

- **Occasional price checks, search pages only** → free tier until you exceed 1,000 credits, then the lowest paid plan. A thousand listings from search costs 85 credits; you have room to experiment.
- **Daily monitoring of a few hundred listings with item details** → the middle tiers. At 5 credits per item page plus potential 10-credit bypass surcharges, a few hundred items a night lands in the tens of thousands of credits monthly.
- **Multi-marketplace, high concurrency, spiky volume** → Scaling or above, because pay-as-you-go with a spending cap is what keeps a big month from forcing a plan jump you didn't want.

Two things to confirm before you enter a card: what the trial allowance actually covers for your specific eBay URL (use the cost estimator, not the average), and whether your scrape needs bypass or rendering, since either one changes the per-request cost by 10 credits or more.

👉 [Start on the 1,000 free credits and size your plan from the real request count](https://www.scraperapi.com/pricing?fp_ref=coupons)
