# n8n Workflow Builder — Claude Code Guide

This project uses Claude Code to build, validate, and deploy n8n workflows via the n8n MCP server and n8n Skills library.

- **n8n MCP server**: https://github.com/czlonkowski/n8n-mcp
- **n8n Skills**: https://github.com/czlonkowski/n8n-skills

---

## Session Start (Do This First, Every Time)

- Read `tasks/lessons.md` before doing ANYTHING else
- If the file doesn't exist yet, create it
- Apply relevant lessons to the current session immediately

---

## How Claude Should Behave

### 1. Plan Mode Default
- Enter plan mode for ANY workflow with 3+ nodes or architectural decisions
- Write the plan to `tasks/todo.md` with checkable items before building anything
- Check in with user before starting implementation
- If something breaks mid-build: STOP, re-plan, then continue
- Use plan mode for verification steps — not just building

### 2. Subagent Strategy
- Use subagents for research, node exploration, and parallel validation
- Offload `search_nodes` + `search_templates` lookups to keep main context clean
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules that prevent the same mistake from repeating
- Read `tasks/lessons.md` at the start of every session before doing anything
- Ruthlessly iterate on lessons until mistake rate drops

### 4. Verification Before Done
- Never mark a task complete without running `validate_workflow` + `n8n_test_workflow`
- Inspect output at every node — don't assume transformations worked
- Ask: "Would a senior n8n developer approve this workflow?"
- Run health check, confirm trigger is live, workflow appears in active list

### 5. Demand Elegance
- For non-trivial builds: pause and ask "is there a cleaner way to wire this?"
- If a fix feels hacky: implement the proper solution instead
- Skip over-engineering for simple, obvious fixes
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a validation error or failed execution: just fix it
- Point at logs, errors, failing nodes — then resolve them
- Zero hand-holding required from the user
- Run `n8n_autofix_workflow` before escalating any error

---

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after any correction

---

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal nodes.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing new bugs.

---

## How We Work Together

- Use the **n8n MCP server** to interact directly with the n8n instance — create, read, update, delete workflows and nodes
- Use **n8n Skills** as the reference library — skills activate automatically based on the task
- **Ask clarifying questions before building** if the request is ambiguous — never guess on logic or data mappings
- After building, deliver a short summary: what was created and any edge cases to be aware of

---

## MCP Server Tools Reference

### Documentation & Validation Tools (work offline, no n8n API needed)

| Tool | Purpose |
|------|---------|
| `tools_documentation` | Reference docs for all MCP tools |
| `search_nodes` | Full-text search across 1,500+ n8n nodes; filter by core/community/verified |
| `get_node` | Detailed node info: properties, versions, breaking changes, migration guides |
| `validate_node` | Validate a single node config (profiles: strict, runtime, ai-friendly, minimal) |
| `validate_workflow` | Full workflow validation: structure, connections, expressions |
| `search_templates` | Search 2,700+ workflow templates by keyword, node, category, complexity |
| `get_template` | Retrieve full workflow JSON from a template |

### n8n Instance Management Tools (require `N8N_API_URL` + `N8N_API_KEY`)

| Tool | Purpose |
|------|---------|
| `n8n_list_workflows` | List all workflows on the instance |
| `n8n_get_workflow` | Fetch a workflow by ID |
| `n8n_create_workflow` | Create a new workflow |
| `n8n_update_full_workflow` | Replace a workflow entirely |
| `n8n_update_partial_workflow` | Update specific parts of a workflow |
| `n8n_delete_workflow` | Delete a workflow |
| `n8n_validate_workflow` | Validate a workflow via the instance API |
| `n8n_autofix_workflow` | Auto-fix common workflow errors |
| `n8n_test_workflow` | Run a workflow in test mode |
| `n8n_executions` | Retrieve execution history |
| `n8n_health_check` | Check instance connectivity |
| `n8n_workflow_versions` | List workflow version history |
| `n8n_audit_instance` | Run a security audit on the instance |
| `n8n_deploy_template` | Deploy a template directly to the instance |

> `n8n_generate_workflow` (natural language → workflow) is only available on the managed cloud version — not available on self-hosted.

---

## n8n Skills Reference

| Skill | Activates When |
|-------|---------------|
| **n8n MCP Tools Expert** | Selecting and calling MCP tools — highest priority skill |
| **n8n Workflow Patterns** | Designing workflow architecture (webhook, HTTP API, DB, AI, scheduled) |
| **n8n Node Configuration** | Configuring individual nodes, property dependencies |
| **n8n Expression Syntax** | Writing `{{ }}` expressions, data mapping, variable access |
| **n8n Validation Expert** | Interpreting validation errors, troubleshooting from error catalogs |
| **n8n Code JavaScript** | Writing Code node logic in JavaScript |
| **n8n Code Python** | Writing Code node logic in Python (note: no external libraries available) |

---

## Workflow Build Process

### 1. Understand Requirements
- Ask clarifying questions if the trigger, data sources, or success criteria are unclear
- Identify: trigger type, data inputs/outputs, integrations needed, error handling expectations
- Confirm scope before building — don't start if requirements are ambiguous

### 2. Search Templates and Nodes
- Run `search_templates` for similar existing workflows before building from scratch
- Use `search_nodes` to find the right node for each integration
- Use `get_node` to check properties, versions, and breaking changes before configuring
- Prefer official/core nodes over community nodes when both exist

### 3. Build the Workflow
- Map out the full node graph before creating any nodes
- Use descriptive node names (e.g., `Get HubSpot Contact`, never `HTTP Request 3`)
- Structure nodes left to right, grouped by function
- Add sticky notes explaining logic in plain English (client-facing)
- Group nodes visually by function with color coding
- Set credentials by reference — never hardcode API keys or secrets
- Use n8n expressions for dynamic values; validate syntax with the Expression Syntax skill
- Handle pagination for nodes that may return large datasets
- Add error branches on every node that calls an external API

### 4. Validate
- Run `validate_workflow` before deploying — use `n8n_autofix_workflow` to fix errors automatically
- Verify all nodes are connected with no dangling edges
- Confirm all required node fields are populated
- Run `n8n_test_workflow` with representative data
- Inspect output at each node to verify transformations
- Fix all errors before reporting completion

### 5. Deploy
- Activate only after validation passes
- Use `n8n_health_check` to confirm instance connectivity before deploying
- Confirm the trigger is live and the workflow appears in the active list

---

## Expression Syntax Reference

| Pattern | Use |
|---------|-----|
| `{{ $json.fieldName }}` | Current node input data |
| `{{ $node["Node Name"].json.field }}` | Data from a specific previous node |
| `{{ $('Node Name').item.json.field }}` | Item from a named node (v1 syntax) |
| `{{ $now.toISO() }}` | Current timestamp |
| `{{ $vars.MY_VAR }}` | Instance-level variable |
| `{{ $env.VAR_NAME }}` | Environment variable |

- Use `.toNumber()`, `.toString()`, `.toDateTime()` for type coercion
- Wrap potentially undefined values: `{{ $json.field ?? 'default' }}`
- For complex logic, use a Code node rather than chaining long expressions

---

## Node Configuration Best Practices

- **HTTP Request**: Set timeout; handle 4xx/5xx with `Continue On Fail` + error branch
- **Code node (JS)**: No external libraries available by default; access input with `$input.all()` or `$input.first()`; always return an array of objects
- **Code node (Python)**: No external libraries (no requests, pandas, numpy); same return format rules
- **If/Switch**: Label all branches; always include a default/else branch
- **Loop Over Items**: Reduce batch size for rate-limited APIs
- **Wait**: Prefer Webhook resume over fixed delays for human-in-the-loop flows
- **Set node**: Use only to reshape data — document what fields are added/removed

---

## Credential Handling

- Never write API keys, tokens, or passwords into workflow parameters
- Reference credentials by name as configured in the n8n UI credential store
- For environment-specific secrets, use n8n Variables (`$vars.NAME`) at instance level
- Flag any test credentials in the workflow description for replacement before production

---

## Safety Rules

- **Never edit production workflows directly** — test in a development workflow first
- Do not activate a workflow that modifies production data without explicit user confirmation
- Do not delete or overwrite existing workflows without confirmation
- Do not send emails, Slack messages, or webhook calls to external systems during test runs
- If a workflow touches a CRM, database, or billing system, confirm sandbox testing first
- Flag any workflow that could cause unbounded loops or recursive triggers before building
- Maintain workflow backups before any AI-assisted modifications

---

## Error Handling Standards

Every production workflow must include:
1. An error branch on each external API call, or a top-level Error Trigger workflow
2. Logging of failed items (Google Sheet, database, or n8n execution log)
3. A notification (Slack or email) for critical failures
4. `Continue On Fail` set intentionally — not to silently swallow errors

---

## Known Limitations

- **Default parameter values** frequently cause runtime failures — always validate node config before execution
- **Response size cap**: MCP responses are capped at 1MB; large workflows may be truncated
- **Community node searches** require explicit `source` filter in `search_nodes`
- **Offline tools**: Documentation/validation tools work without an n8n API key; management tools require one
- **`n8n_generate_workflow`** is cloud-only — not available on self-hosted MCP server

---

## Quality Checklist (before marking complete)

- [ ] `tasks/lessons.md` reviewed at session start
- [ ] Plan written to `tasks/todo.md` before building
- [ ] All nodes have descriptive names (no "HTTP Request 3", "Set", "If")
- [ ] Nodes flow left to right, grouped logically by function
- [ ] Sticky notes added for any non-obvious logic
- [ ] No hardcoded credentials or secrets
- [ ] Expressions validated against sample data
- [ ] Error branches present on all external API calls
- [ ] `validate_workflow` passed; `n8n_autofix_workflow` run if needed
- [ ] `n8n_test_workflow` run with representative data
- [ ] Workflow activates without errors
- [ ] Post-build summary delivered: what was built + known edge cases
- [ ] `tasks/lessons.md` updated if any correction was made
