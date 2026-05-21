# n8n Workflow Builder — Claude Code Guide
This project uses Claude Code to build, validate, and deploy n8n workflows via the n8n MCP server and n8n Skills library.

n8n MCP server: https://github.com/czlonkowski/n8n-mcp
n8n Skills: https://github.com/czlonkowski/n8n-skills

---

## Session Start (Do This First, Every Time)
- Read tasks/lessons.md — if it doesn't exist, create it
- Apply relevant lessons immediately before doing anything else
- Read tasks/progress.md — if it doesn't exist, create it
- If progress.md has entries, state the last completed milestone and ask: "Should I continue from here?"

---

## How Claude Should Behave

### 1. Plan Mode
Any workflow with 3+ nodes or architectural decisions requires full Plan Mode.

Plan Mode is collaborative and step-by-step. Claude presents one step at a time, waits for approval or discussion, and only advances when the user confirms.

**Step 0 — 📋 Project Brief Review**

Before any planning begins:

- Look for tasks/project-brief.md — if it exists, read it fully
- If no brief exists, ask: "Do you have a project brief I should read first?"
- After reading, present a Brief Summary covering:
  - What Claude understood the automation to do
  - Key inputs and outputs identified
  - Any ambiguities that need resolving before planning
- User confirms the summary before Step 1 begins
- If the brief pre-fills some steps, note it — but still walk through each step explicitly

Claude plans from understanding, not assumptions.

**Steps 1–11 (one at a time, in order):**

**Step 1 — 🎯 GOAL**
One sentence. What does this automation do?

**Step 2 — 🔄 LOGIC FLOW**
Trigger → Step → Decision → Output. Plain language, no nodes yet.

**Step 3 — 🏗️ ZONES**
Map each zone to its purpose, intended node, and key config.

**Step 4 — 🗺️ DATA MAPPING**
Track what the data looks like as it moves through each zone — before writing any expressions.

**Step 5 — 📝 NODES MAPPED TO ZONES**

| Zone | Node | Config Note |
|------|------|-------------|

**Step 6 — 🔌 TOOLS & INTEGRATIONS**
Every external service this automation touches.

**Step 7 — 🔑 API / CREDENTIALS CHECKLIST**

| Service | What's Needed | Where to Get It |
|---------|--------------|-----------------|

**Step 8 — 🧩 EDGE CASES**
The "what if" list — empty responses, duplicates, API limits, null fields.

**Step 9 — ⚠️ ERROR HANDLING PLAN**
For each zone: what might go wrong and how it gets handled.

**Step 10 — 🧪 TESTING PLAN**
Four layers:
- Unit — each node in isolation using pinned data
- Integration — full flow end-to-end with real data
- Edge Case — deliberately trigger failure scenarios
- Regression — what to recheck after any future edits

For each layer: what to test, how to execute in n8n, pass/fail criteria. End with a numbered execution sequence from first node to live activation.

**Step 11 — 🏁 MILESTONES**
Group steps into logical, testable chunks. Each milestone needs:
- A clear name
- What "done" means
- 2–3 edge cases that must pass before marking complete

A milestone is only done when its edge cases pass — not just the happy path.

---

**After Step 11 is approved:**

- Write the compiled plan to tasks/todo.md with checkable items
- Present a clean readable summary of the full plan
- Ask: "Ready to start execution?"
- Do not touch a single node until confirmed

---

### MILESTONE EXECUTION PROTOCOL

**This protocol is mandatory. It cannot be skipped, abbreviated, or overridden.**

- Build ONLY the nodes assigned to the current milestone
- After the last node of that milestone is placed: **STOP COMPLETELY**
- Do not begin, scaffold, or touch any node from the next milestone
- Output exactly this message:

  > 🛑 **Milestone [X] complete.**
  > Nodes built: [list them]
  > Please test and confirm before I continue.

- Wait for the user to type **"confirmed"** or **"continue"**
- Only then begin the next milestone
- If the user says anything other than confirmation, treat it as feedback and resolve it first before proceeding

**HARD RULE: Building ahead of milestone confirmation is a violation of this guide. This applies even if subsequent nodes seem obvious, simple, or already planned.**

---

### 2. Subagent Strategy
- Use subagents for research, node exploration, and parallel validation
- Offload search_nodes + search_templates to keep main context clean
- One task per subagent

### 3. Self-Improvement Loop
- After any correction: update tasks/lessons.md with the pattern
- Read tasks/lessons.md at session start — every session, no exceptions
- Write rules that prevent the same mistake repeating

### 4. Verification Before Done
- Never mark complete without running validate_workflow + n8n_test_workflow
- Inspect output at every node — don't assume transformations worked
- Run health check, confirm trigger is live, workflow appears in active list

### 5. Demand Elegance
- Ask "is there a cleaner way to wire this?" before finalising non-trivial builds
- If a fix feels hacky, implement the proper solution
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a validation error or failed execution: just fix it
- Run n8n_autofix_workflow before escalating any error
- Zero hand-holding required

---

## Client-Facing Presentation Standards
Every workflow is a client deliverable. A non-technical person must open it and understand what's happening.

**Node labelling:**
- Every node gets a descriptive plain-English name
- Never: "HTTP Request 3", "Set", "If", "Code"
- Good: Get Customer from HubSpot, Check If Email Exists, Send Slack Alert
- Names should read like a sentence left to right

**Sticky notes:**
- Top of workflow: 2–3 sentence overview in non-technical language
- On any zone or node cluster that isn't obvious
- On all conditional logic (If/Switch branches)
- On any credentials that need configuring before use

**Colour coding:**
- 🟦 Blue — trigger / entry point
- 🟩 Green — data processing / transformation
- 🟨 Yellow — decision / conditional logic
- 🟥 Red — error handling
- 🟪 Purple — external API calls / integrations
- ⬜ Grey — utility / formatting nodes

**Layout:**
- Nodes flow left to right
- Visual spacing between zones
- Error branches below the main flow, not inline

---

## Workflow Build Process
1. **Understand** — clarify trigger, data, integrations, success criteria before starting
2. **Search** — run search_templates before building from scratch; use search_nodes + get_node to find and validate the right nodes; prefer core nodes over community
3. **Build** — follow node naming and sticky note standards; set credentials by reference; add error branches on every external API call
4. **Validate** — run validate_workflow; use n8n_autofix_workflow if needed; run n8n_test_workflow with real data; inspect every node output
5. **Deploy** — run n8n_health_check; activate only after validation passes; confirm trigger is live

---

## Safety Rules
- Never edit production workflows directly — test in a dev workflow first
- Do not activate workflows that modify production data without explicit confirmation
- Do not delete or overwrite workflows without confirmation
- No emails, Slack messages, or webhook calls to external systems during test runs
- Confirm sandbox testing before touching any CRM, database, or billing system
- Flag any workflow that could cause unbounded loops before building
- Back up workflows before AI-assisted modifications
- **Never build beyond the current confirmed milestone — even if subsequent nodes seem obvious or simple**

---

## Decisions Log
Append-only. See tasks/decisions.md for the full log and entry format.

At end of each session, ask Claude: "Summarise the key decisions you made this session for my decisions log." Paste entries into tasks/decisions.md.

---

## Quality Checklist (before marking complete)
- [ ] tasks/lessons.md reviewed at session start
- [ ] tasks/project-brief.md read and Brief Summary confirmed
- [ ] All 11 planning steps completed and approved
- [ ] Full plan written to tasks/todo.md and confirmed before building
- [ ] All nodes have descriptive plain-English names
- [ ] Nodes flow left to right, grouped by zone
- [ ] Colour-coded sticky notes added (overview + non-obvious logic)
- [ ] No hardcoded credentials or secrets
- [ ] Error branches on all external API calls
- [ ] validate_workflow passed; n8n_autofix_workflow run if needed
- [ ] n8n_test_workflow run with representative data
- [ ] Workflow activates without errors
- [ ] Post-build summary delivered: what was built + known edge cases
- [ ] tasks/lessons.md updated if any correction was made
- [ ] tasks/decisions.md updated with key decisions from this session
- [ ] Each milestone confirmed by user before the next one began

---

For expression syntax, node config notes, MCP tools, and known limitations — see tasks/reference.md
