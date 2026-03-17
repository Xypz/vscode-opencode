# AGENTS.md - n8n Automations Project

## Project Overview

This is an **n8n automation project** containing workflow JSON files that run on a local n8n instance. The project also includes custom opencode skills for agentic workflow development.

---

## Build / Test / Lint Commands

### n8n Local API

Interact with local n8n instance using the API:
- **Base URL**: Read from `.env` file (`N8N_HOST`)
- **API Key**: Read from `.env` file (`N8N_API_KEY`)
- **Header**: `X-N8N-API-KEY` (not `Authorization`)

```bash
# Example: List workflows
curl -H "X-N8N-API-KEY: $N8N_API_KEY" $N8N_HOST/api/v1/workflows
```

### Workflow Management

- **Import workflow**: Use n8n UI → Workflows → Import from File
- **Test locally**: Deploy via API and check n8n execution logs
- **No traditional tests**: n8n workflows are tested manually via the UI

### Project Scripts (`.opencode/`)

The `.opencode/` directory contains a minimal Node.js setup (bun-based):
```bash
cd .opencode
bun install  # Install dependencies
```

---

## Code Style Guidelines

### General Principles

1. **Simplicity First**: Make each change as simple as possible. Minimum necessary code.
2. **Zero Laziness**: Find root causes. No temporary patches. Senior developer standards.
3. **Minimal Impact**: Changes should only touch what's strictly necessary.

### n8n Workflow JSON Conventions

1. **Node IDs**: Use simple sequential IDs without dashes
   - ✅ Good: `n1`, `n2`, `n3`
   - ❌ Bad: `schedule-trigger`, `http-reddit`

2. **Required Settings**: Always include `settings` object
   ```json
   "settings": {
     "executionOrder": "v1",
     "availableInMCP": false
   }
   ```

3. **Connections**: Use explicit input indices for Merge nodes
   ```json
   "Node Name": {
     "main": [[{"node": "Merge", "type": "main", "index": 0}]]
   }
   ```

4. **Minimal JSON**: Keep workflow JSON minimal when creating via API
   - Include only required fields
   - Let user configure AI nodes and credentials via UI

### JavaScript in Code Nodes

1. **Output Format**: Always return array with `json` key
   ```javascript
   return [{ json: { miDato: "valor" } }];
   ```

2. **Defensive Programming**: Never assume fields exist
   ```javascript
   const value = data?.user?.id;
   ```

3. **Error Handling**: Always use `try/catch` blocks

4. **Sanitization**: Use `encodeURIComponent()` for dynamic URL parameters

### Security Rules (CRITICAL)

1. **ZERO Secrets**: NEVER write API keys, tokens, or passwords in code
   - Use n8n's native Credentials system
   - Store secrets in `.env` (already gitignored)

2. **No Hardcoded Credentials**:
   - ✅ Use n8n Credentials
   - ❌ Never: `apiKey: "sk-xxx"` in workflow JSON

### Documentation

1. **Workflow README**: Create for each workflow with:
   - Installation steps
   - Credential configuration
   - Node structure diagram
   - Customization options

2. **Lessons**: Document every bug fix in `tasks/lessons.md`:
   ```markdown
   ### YYYY-MM-DD

   **Problema:** _Describe el error_

   **Solución:** _Cómo se resolvió_

   **Regla creada:** _Prevención para el futuro_
   ```

---

## Agent Workflow

### Planning Mode

- Enter **planning mode** for any non-trivial task (3+ steps or architecture decisions)
- If something goes wrong, **STOP and replan** immediately
- Write detailed specifications upfront

### Verification Before Done

- NEVER mark a task as complete without testing
- Compare behavior between main code and changes
- Ask: "Would a Staff Engineer approve this?"
- Check n8n node logs to verify correctness

### Self-Improvement Loop

- After ANY user correction: update `tasks/lessons.md`
- Write rules to avoid repeating the same mistake
- Review lessons at the start of each session

---

## File Structure

```
/Users/xy/vscode/
├── workflows/           # n8n workflow JSON files
│   ├── reddit-bitcoin-radar.json
│   └── README.md
├── tasks/
│   ├── lessons.md       # Bug fixes and learned patterns
│   └── todo.md          # Current task tracking
├── .opencode/
│   ├── skills/superpower/  # Custom opencode skills
│   └── package.json
├── RULES.md             # Project rules (Spanish)
└── .env                 # Secrets (gitignored)
```

---

## Key Resources

### n8n API Endpoints (v1.x)

- `GET /api/v1/workflows` - List workflows
- `POST /api/v1/workflows` - Create workflow
- `GET /api/v1/workflows/:id` - Get workflow
- `PUT /api/v1/workflows/:id` - Update workflow
- `DELETE /api/v1/workflows/:id` - Delete workflow

### Reference Repositories

- `enescingoz/awesome-n8nTemplates` - Ready-to-import workflows
- `Zie619/n8n-workflows` - Advanced automation examples

---

## Testing Workflows

1. Deploy workflow to local n8n via API or UI
2. Trigger manually or wait for schedule
3. Check n8n execution panel for errors
4. Verify output in Telegram/Slack/etc.
5. Update `tasks/lessons.md` with any issues found
