# HVAC Intelligence Digest — Lessons Learned

## Format
Each entry: [Date] — [Lesson] — [Why it matters]

---

## Lessons

<!-- Add as discovered, newest at top -->

### 2026-05-25 — 3 Source Fixes + Rules to Remember

**What we fixed:**

**HouseCall Pro — 403 error**
- Problem: n8n's HTTP Request node was being blocked because it didn't identify itself as a browser
- Fix: Added a User-Agent header mimicking Chrome
- Then: Switched to HTTP Request + XML node + Code node to parse the RSS into clean fields

**Jobber Academy — "contentType is required" error**
- Problem: Content Type field was set to text/html instead of application/json
- Fix: Changed Content Type to application/json

**HVAC Talk via Apify — same error**
- Problem: Same Content Type mismatch
- Fix: Same fix — change to application/json

**Rules to remember:**
- **Whenever you send JSON data to an API, always set Content Type to `application/json`** — not text/html, not raw
- **Whenever n8n gets a 403 from a website, first suspect a missing User-Agent header** before assuming Cloudflare
- **RSS feeds return XML** — always need an XML node before a Code node can read them

### 2026-05-25 — Firecrawl as Cloudflare Bypass for n8n (fixed)
- **Use Firecrawl (`https://api.firecrawl.dev/v1/scrape`) for any Cloudflare-protected site in n8n.** It runs a real headless browser, solves Cloudflare JS challenges automatically, and returns clean markdown — no HTML parsing needed.
- **Request pattern:** POST with `Authorization: Bearer YOUR_KEY` header and body `{"url":"https://target.com","formats":["markdown"]}`. Use `contentType: "raw"` (not `"json"`) to avoid double-encoding.
- **Response shape:** `{success: true, data: {markdown: "...", metadata: {title, sourceURL}}}` — markdown goes straight to Claude, no further cleaning.
- **Free tier:** 500 credits/month. Two sites every other day = ~30 scrapes/month, well within limit.
- **Why not Apify Smart Article Extractor (`lukaskrivka~article-extractor-smart`):** The actor's input schema changed and now rejects calls with `contentType is required` — a schema-validation 400 error that can't be worked around without knowing the new required field name. Don't use this actor.

### 2026-05-25 — Cloudflare vs Basic UA Filter — Different Problems, Different Fixes
- **Diagnose before you fix.** "Returns Cloudflare page" can mean two completely different things: (a) basic User-Agent filter — the site checks the UA string and blocks non-browser requests; or (b) genuine Cloudflare JS challenge — requires a real browser that executes JavaScript to solve the challenge.
- **Test to tell them apart:** `Invoke-WebRequest -Uri $url -UserAgent "Mozilla/5.0 ..."` — if that returns real content, it's just a UA filter. If it still returns the Cloudflare challenge page, it's genuine Cloudflare.
- **HouseCall Pro was a UA filter:** Adding `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36` header to the httpRequest node fixes it. No Apify needed.
- **Jobber Academy + HVAC Informed are genuine Cloudflare:** No header tricks work. Need a real browser. Solution: Apify Smart Article Extractor (see next lesson).

### 2026-05-25 — Apify Smart Article Extractor for Cloudflare-Protected Sites (fixed)
- **Use Apify actor `lukaskrivka~article-extractor-smart` for any site with genuine Cloudflare protection.** It runs a real Puppeteer browser, solves Cloudflare JS challenges automatically, and returns structured article data — no HTML parsing needed.
- **Endpoint:** `POST https://api.apify.com/v2/acts/lukaskrivka~article-extractor-smart/run-sync-get-dataset-items?token=YOUR_TOKEN&outputRecordFormat=json`
- **Body:** `{"startUrls":[{"url":"https://target-site.com/"}],"maxArticlesPerCrawl":8}`
- **Response fields:** `title`, `text`, `url`, `date` — map to `contentSnippet`, `link`, `pubDate` in your parse node.
- **Cost:** Well within Apify free tier (250 compute units/month) for 2 sites running every other day (~30 runs/month).
- **Parse node pattern:**
  ```javascript
  const items = $input.all();
  const results = [];
  for (const item of items) {
    if (item.json.error !== undefined) continue;
    if (!item.json.text && !item.json.title) continue;
    results.push({ json: {
      title: item.json.title || '',
      contentSnippet: (item.json.text || '').replace(/\s+/g, ' ').trim().substring(0, 2000),
      link: item.json.url || '',
      pubDate: item.json.date || new Date().toISOString(),
      feedSource: 'source-name.com'
    }});
  }
  return results.length > 0 ? results : [{ json: { error: 'No articles from Apify' } }];
  ```
- **contentType must be `"raw"`** — same rule as Claude and HVAC Talk nodes. Do NOT use `contentType: "json"` or the body will be double-encoded.

### 2026-05-23 — API Patches Silently Lost on Re-Fetch (fixed)
- **Never patch a workflow in multiple separate PUT calls.** Each PUT replaces the entire workflow. If you fetch → patch → push, then fetch again → patch → push, the second fetch gets the saved state from the first push — but if the first push missed nodes (e.g. parse nodes added in a prior session), the second push permanently drops them.
- **Always do ONE comprehensive fetch → ALL mutations → ONE push.** Verify node count and key field values from the PUT response before trusting success.
- **`Add-Member -Force` on nested PSCustomObjects does not always persist through `ConvertTo-Json`.** Use direct property assignment (`$node.parameters.jsCode = $code`) for existing properties. For new properties on nested objects, rebuild the entire sub-object as a fresh `[PSCustomObject]@{}`.

### 2026-05-23 — Error Items Leak Through Filter Node (fixed)
- **Always add `if (item.json.error !== undefined) continue;` as the FIRST guard in any filter/dedup Code node.** When upstream nodes have `continueOnFail: true`, failed items flow downstream as `{error: "..."}` objects. These have no `pubDate` (so they pass the age check) and no `link` (so they get a random dedup key and pass that check too) — meaning every error item survives the filter and pollutes Claude's input.
- **Fallback also needs the guard:** `items.filter(i => !i.json.error).slice(0, 12)` — otherwise the fallback can return only error items when all real content is old.

### 2026-05-23 — RSS Nodes Fail on Sites Without RSS Feeds (fixed)
- **ALWAYS verify that a source URL is an actual RSS/Atom feed before using `rssFeedRead` node.** If the URL serves HTML (not XML with `<rss>` or `<feed>` tags), the node throws XML parsing errors (`Attribute without value`, `Invalid character in tag name`, `Unexpected close tag`).
- **How to test:** `Invoke-WebRequest -Uri $url | Select-Object -Expand Content` and check for `<rss` or `<feed`. If not present, the URL is not an RSS feed.
- **Fix pattern:** Replace `rssFeedRead` with `httpRequest` (GET, responseFormat=text) + a Code node that strips HTML and returns a single structured item `{title, link, pubDate, contentSnippet, feedSource}`.
- **Affected sources:** housecallpro.com/resources, getjobber.com/academy, hvacinformed.com — none had public RSS feeds.

### 2026-05-23 — Apify Body Not Sent (fixed)
- **When patching an httpRequest node's credentials via API, always re-check `contentType` is set.** Patching credentials can leave `contentType` as empty string, causing n8n to send no `Content-Type` header — so Apify receives the POST body but can't parse it, returning `startUrls is required`.
- **Fix:** Set `contentType: "raw"` and add `content-type: application/json` manual header on any httpRequest node sending a JSON body.

### 2026-05-23 — Claude Body Double-Encoding Bug (fixed)
- **NEVER use `contentType: "json"` + `specifyBody: "string"` + `JSON.stringify()` together in an HTTP Request node.** n8n JSON-encodes the already-stringified string a second time, so Anthropic receives a JSON string literal instead of an object — causing `model: Field required`.
- **Correct pattern:** Use `contentType: "raw"` + `specifyBody: "string"` + add `content-type: application/json` as a manual header. n8n sends the string body exactly as-is with no additional encoding.
- **Symptom:** Anthropic returns 400 `invalid_request_error: model: Field required` even though the body expression looks correct in n8n's debug view.

### 2026-05-23 — Claude Auth Bug (fixed)
- **NEVER use `genericCredentialType` + `httpHeaderAuth` for Anthropic HTTP Request nodes** — the existing httpHeaderAuth credential may inject `Authorization: Bearer` instead of `x-api-key`, causing a 400 Bad Request from Anthropic.
- **Correct pattern:** Use `authentication: "predefinedCredentialType"` + `nodeCredentialType: "anthropicApi"` + attach the `anthropicApi` credential type. n8n knows Anthropic's exact header format and injects `x-api-key` correctly.
- **Credential to use:** "Anthropic account 3" (`Ew8le0xtB1KvwHFJ`, type: `anthropicApi`)

### 2026-05-23 — Setup
- RSS feed URLs follow WordPress /feed/ pattern for most HVAC blog sources
- hvac-talk.com (vBulletin forum) requires Apify scraping — no public RSS confirmed
- Apify website-content-crawler works for forums but needs generous timeout (120s)
- Claude prompt is built in the Code node (not inline in HTTP Request) to avoid expression escaping hell
- All fetch nodes must have continueOnFail: true so one bad feed doesn't kill the digest
- Filter code falls back to latest 12 items if pubDate is missing or old — prevents empty digest
