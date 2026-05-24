# HVAC Intelligence Digest — Lessons Learned

## Format
Each entry: [Date] — [Lesson] — [Why it matters]

---

## Lessons

<!-- Add as discovered, newest at top -->

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
