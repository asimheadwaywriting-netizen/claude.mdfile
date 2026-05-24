# HVAC Intelligence Digest — Lessons Learned

## Format
Each entry: [Date] — [Lesson] — [Why it matters]

---

## Lessons

<!-- Add as discovered, newest at top -->

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
