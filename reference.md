# n8n Reference — Look Up As Needed

This file is reference material. Claude consults it mid-build when needed — it is not read every session.

---

## MCP Server Tools

### Documentation & Validation (no n8n API key needed)

| Tool | Purpose |
|------|---------|
| `tools_documentation` | Reference docs for all MCP tools |
| `search_nodes` | Full-text search across 1,500+ n8n nodes; filter by core/community/verified |
| `get_node` | Detailed node info: properties, versions, breaking changes, migration guides |
| `validate_node` | Validate a single node config (profiles: strict, runtime, ai-friendly, minimal) |
| `validate_workflow` | Full workflow validation: structure, connections, expressions |
| `search_templates` | Search 2,700+ workflow templates by keyword, node, category, complexity |
| `get_template` | Retrieve full workflow JSON from a template |

### Instance Management (require `N8N_API_URL` + `N8N_API_KEY`)

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

> `n8n_generate_workflow` is cloud-only — not available on self-hosted.

---

## n8n Skills

| Skill | Activates When |
|-------|---------------|
| **n8n MCP Tools Expert** | Selecting and calling MCP tools — highest priority |
| **n8n Workflow Patterns** | Designing workflow architecture |
| **n8n Node Configuration** | Configuring individual nodes, property dependencies |
| **n8n Expression Syntax** | Writing `{{ }}` expressions, data mapping |
| **n8n Validation Expert** | Interpreting validation errors |
| **n8n Code JavaScript** | Writing Code node logic in JavaScript |
| **n8n Code Python** | Writing Code node logic in Python |

---

## Expression Syntax

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

## Node Configuration Notes

- **HTTP Request** — set timeout; handle 4xx/5xx with `Continue On Fail` + error branch
- **Code node (JS)** — no external libraries; access input with `$input.all()` or `$input.first()`; always return an array of objects
- **Code node (Python)** — no external libraries (no requests, pandas, numpy); same return rules
- **If/Switch** — label all branches; always include a default/else branch
- **Loop Over Items** — reduce batch size for rate-limited APIs
- **Wait** — prefer Webhook resume over fixed delays for human-in-the-loop flows
- **Set node** — use only to reshape data; document what fields are added/removed

---

## Error Handling Standards

Every production workflow must include:
1. Error branch on each external API call, or a top-level Error Trigger workflow
2. Logging of failed items (Google Sheet, database, or n8n execution log)
3. Notification (Slack or email) for critical failures
4. `Continue On Fail` set intentionally — not to silently swallow errors

---

## Credential Handling

- Never write API keys, tokens, or passwords into workflow parameters
- Reference credentials by name as configured in the n8n UI credential store
- For environment-specific secrets, use n8n Variables (`$vars.NAME`) at instance level
- Flag any test credentials in the workflow description for replacement before production

---

## Known Limitations

- **Default parameter values** frequently cause runtime failures — always validate before execution
- **Response size cap** — MCP responses capped at 1MB; large workflows may be truncated
- **Community node searches** — require explicit `source` filter in `search_nodes`
- **Offline tools** — documentation/validation work without an API key; management tools require one
- **`n8n_generate_workflow`** — cloud-only, not available on self-hosted
