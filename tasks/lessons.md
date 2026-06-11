# n8n Lessons Learned — General Patterns

These are reusable patterns that apply across n8n projects, not tied to any one workflow. Project-specific lessons live in that project's own `tasks/lessons.md` on the F: drive.

## Format
Each entry: [Lesson] — [Why it matters / fix]

---

## Lessons

<!-- Add as discovered, newest at top. Move project-specific details to the project's own lessons.md instead. -->

### n8n PUT /workflows/{id} — exact body shape
- PUT body must contain ONLY `name`, `nodes`, `connections`, `settings`, `staticData` — omit `id`, `createdAt`, `updatedAt`, `active`, `tags`, `versionId` from a GET response.
- `settings` must be exactly `{"executionOrder": "v1"}`. Extra fields like `callerPolicy`, `availableInMCP`, `binaryMode` (present in GET responses) cause `400: "request/body/settings must NOT have additional properties"`.
- Always do ONE comprehensive fetch → all mutations → ONE push. Multiple separate fetch/patch/push cycles can silently drop nodes added in a prior push.

### Anthropic/Claude HTTP Request nodes — auth and body encoding
- **Auth:** use `authentication: "predefinedCredentialType"` + `nodeCredentialType: "anthropicApi"` (NOT `genericCredentialType` + `httpHeaderAuth`, which can send `Authorization: Bearer` instead of `x-api-key` → 400).
- **Body:** use `contentType: "raw"` + `specifyBody: "string"` + a manual `content-type: application/json` header (NOT `contentType: "json"` + `JSON.stringify()`, which double-encodes the body → `model: Field required`).
- Credential to use: see tasks/reference.md → Credential Handling.

### Code node filters must guard against upstream error items
When upstream nodes use `continueOnFail: true`, failed items flow downstream as `{error: "..."}` objects with no real fields — they can slip past date/dedup checks since they have no `pubDate`/`link`. Always add `if (item.json.error !== undefined) continue;` as the first guard in any filter/dedup Code node, including fallback paths.

### RSS feed verification before using rssFeedRead
Verify a source URL is real RSS/Atom (`<rss` or `<feed` present in the raw response) before using `rssFeedRead` — otherwise it throws XML parsing errors (`Attribute without value`, `Unexpected close tag`, etc.). If not real RSS, use `httpRequest` (GET, response format text) + a Code node producing a structured item `{title, link, pubDate, contentSnippet, feedSource}`.

### Diagnosing blocked requests: UA filter vs genuine Cloudflare
"Returns Cloudflare/blocked page" can mean two different things:
- **Basic User-Agent filter** — fix by adding a browser `User-Agent` header (e.g. Chrome UA string) to the httpRequest node.
- **Genuine Cloudflare JS challenge** — needs a real headless browser. Use Firecrawl (`POST https://api.firecrawl.dev/v1/scrape`, `contentType: "raw"`, body `{"url": "...", "formats": ["markdown"]}`, response `{data: {markdown, metadata}}`) or an Apify article-extractor actor.

Test which one you're dealing with: `Invoke-WebRequest -Uri $url -UserAgent "Mozilla/5.0 ..."` — real content = UA filter; still blocked = genuine Cloudflare.

### HTML-to-Markdown (Turndown) doesn't strip `<style>` tags
n8n's Markdown node (`htmlToMarkdown` mode, uses Turndown) leaves `<style>...</style>` blocks in the output as escaped CSS text, wasting LLM tokens. Strip before conversion in the expression feeding the Markdown node:
```
={{ $json.field.replace(/<style[\s\S]*?<\/style>/gi, '') }}
```
Applies to any template-heavy site (e.g. Google devsite pages) scraped via HTML Extract → Markdown.

### Sending JSON bodies via httpRequest
Always set `contentType: "raw"` with a manual `content-type: application/json` header when sending a JSON string body. `contentType: "json"` combined with a pre-stringified body double-encodes it.
