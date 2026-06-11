# Progress Log

## Format
Each entry: [Date] — [Project] — [Milestone completed] — [Next step]

---

## Log

<!-- Entries go here, newest at top -->
2026-06-11 — Google Search Updates Tracker — Simulated the new pipeline locally against the real "A new resource for optimizing for generative AI in Google Search" post + its referenced guide (curl + turndown), confirming extraction, triage link detection, and combined markdown all work as designed. Found and fixed a bug: Turndown doesn't strip `<style>` tags, leaking ~1.8KB of raw CSS into the referenced guide's markdown. Patched "Post to Markdown" and "Referenced Page to Markdown" nodes to strip `<style>` blocks before conversion. Pushed via API (workflow vxtvokNoNbi2SaZN, versionCounter 25). — Next: verify live in n8n editor — pin the "A new resource..." RSS item and run step-by-step; confirm final summary reflects the guide's content (mythbusting, action items) and a self-contained post still works via the false branch.
2026-06-11 — Google Search Updates Tracker — Added a "follow the real source" pipeline: fetch full blog post page, triage LLM decides if the post just announces a separate guide/doc with the real details, conditionally fetches & converts that referenced page, then summarizes from combined content. Pushed via API (workflow vxtvokNoNbi2SaZN, versionCounter 24). — Next: verify in n8n editor — test with "A new resource for optimizing for generative AI in Google Search" (should follow the link to /search/docs/fundamentals/ai-optimization-guide) and with a self-contained post (no referenced guide branch).
