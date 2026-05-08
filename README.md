# claude.mdfile
CLAUDE.md behavior rules for n8n automation projects
# claude.mdfile

A personal CLAUDE.md behavior file for building n8n automation workflows with Claude Code.

**Created:** May 2026  
**Author:** Asim (asimheadwaywriting-netizen)  
**Stack:** n8n · Claude Code · MCP Server · VS Code

---

## What This File Does

This is a `CLAUDE.md` instruction file that tells Claude Code *how to behave* inside n8n automation projects — not just what tools to use, but how to think, plan, and work.

---

## What's Covered

### Behavior Rules (Boris Cherny Framework)
- **Session Start** — read lessons before every session
- **Plan Mode** — write plan to `tasks/todo.md` before building any workflow
- **Subagent Strategy** — offload research to keep context clean
- **Self-Improvement Loop** — log corrections to `tasks/lessons.md` automatically
- **Verification Before Done** — validate + test before marking complete
- **Demand Elegance** — no hacky fixes; always find the cleaner solution
- **Autonomous Bug Fixing** — resolve errors without hand-holding

### Task Management
- 6-step workflow: Plan → Verify → Track → Explain → Document → Capture Lessons

### Core Principles
- Simplicity First · No Laziness · Minimal Impact

### n8n MCP Tools Reference
- 20 MCP tools across documentation/validation and instance management
- Covers offline tools (no API key needed) and live instance tools

### n8n Skills Reference
- 7 auto-activating skills: MCP Tools, Workflow Patterns, Node Configuration, Expression Syntax, Validation, JS Code, Python Code

### Workflow Build Process
- 5-step process: Understand → Search → Build → Validate → Deploy

### Node Configuration
- Client-facing sticky notes in plain English
- Color coding by function group
- Descriptive node labels (no default names)
- Expression syntax reference table

### Safety + Error Handling
- Never edit production workflows directly
- Error branches on every external API call
- Credential handling rules

### Quality Checklist
- 13-point checklist before marking any workflow complete

---

## File Structure

```
n8n-agency/
├── CLAUDE.md                  ← this file (behavior rules)
├── lessons.md                 ← permanent lessons log
│
├── doc-processing/
│   ├── PROJECTS.md            ← what + why
│   └── todo.md                ← build plan
│
└── marketing-automation/
    ├── PROJECTS.md
    └── todo.md
```

---

## How to Use

```bash
git clone https://github.com/asimheadwaywriting-netizen/claude.mdfile
cd your-project
# place CLAUDE.md in project root
claude
```

Claude Code reads `CLAUDE.md` automatically on session start.
