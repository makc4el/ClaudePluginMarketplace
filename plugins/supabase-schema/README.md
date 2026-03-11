# supabase-schema

**Automatic Supabase schema documentation for Claude Code.**

Extracts the full database schema — tables, columns, types, required fields, primary keys, foreign keys, and RPC functions — from any Supabase project and writes it to `SUPABASE_SCHEMA.md`. Runs automatically every session so Claude always has accurate, up-to-date schema context when planning queries, migrations, or feature work.

---

## What it does

- Fetches the OpenAPI/Swagger definition from your Supabase REST endpoint
- Generates a clean Markdown file with every table's columns, types, constraints, and relationships
- Lists all RPC functions with type annotations (vector search, vault, etc.)
- Writes `SUPABASE_SCHEMA.md` next to your `.env` file (or wherever you configure it)

**Example output:**

```markdown
### `orgs`

| Column       | Type        | Req | Default            | Notes    |
|--------------|-------------|:---:|--------------------|----------|
| `id`         | `uuid`      | ✅  | `gen_random_uuid()` | 🔑 PK   |
| `user_id`    | `uuid`      | ✅  |                    |          |
| `label`      | `text`      | ✅  |                    |          |
| `instance_url` | `text`    | ✅  |                    |          |
| `created_at` | `timestamptz` |   | `now()`            |          |
```

---

## Requirements

- **Node.js v14+** — no npm packages, uses only built-in modules
- A Supabase project with a service role key
- The key needs read access to the project's REST schema endpoint

---

## Installation

```bash
claude plugin install supabase-schema@local-plugins
```

Or install directly from the plugin directory:

```bash
claude plugin install /path/to/supabase-schema
```

---

## Credentials

The plugin resolves credentials in this order (first match wins):

| Priority | Source |
|----------|--------|
| 1 | `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY` environment variables |
| 2 | `.env` file — walks up directory tree from `cwd` until root |
| 3 | `.env` file in common subdirectories: `backend/`, `server/`, `api/`, `supabase/`, `context-sync/`, `app/` |
| 4 | `SUPABASE_ENV_FILE=/path/to/.env` override |

**Minimum required in `.env`:**
```
SUPABASE_URL=https://<project-ref>.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...
```

> The script only uses the REST schema endpoint — it never writes to your database.

---

## Output path

| Priority | Resolves to |
|----------|-------------|
| 1 | `SUPABASE_SCHEMA_OUTPUT=/custom/path/schema.md` (absolute or relative) |
| 2 | `SUPABASE_SCHEMA_DIR=/custom/dir/` → writes `SUPABASE_SCHEMA.md` there |
| 3 | Same directory as the `.env` file that was found |
| 4 | Current working directory |

---

## Usage

### Automatic (SessionStart hook)

`SUPABASE_SCHEMA.md` is regenerated at the start of every Claude Code session. No action required.

### Manual refresh

Run the slash command:

```
/supabase-schema:refresh
```

Or run the script directly:

```bash
node /path/to/plugin/scripts/extract-schema.js
```

With overrides:

```bash
SUPABASE_URL=https://myproject.supabase.co \
SUPABASE_SERVICE_ROLE_KEY=eyJ... \
SUPABASE_SCHEMA_OUTPUT=./docs/db-schema.md \
node scripts/extract-schema.js
```

---

## Environment variable reference

| Variable | Required | Purpose |
|----------|----------|---------|
| `SUPABASE_URL` | ✅ (or via .env) | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | ✅ (or via .env) | Service role key for schema access |
| `SUPABASE_ENV_FILE` | optional | Explicit path to a `.env` file |
| `SUPABASE_SCHEMA_OUTPUT` | optional | Full output file path (overrides all) |
| `SUPABASE_SCHEMA_DIR` | optional | Output directory (filename fixed to `SUPABASE_SCHEMA.md`) |

---

## Committing the output

`SUPABASE_SCHEMA.md` is a generated file — commit it so your whole team (and Claude in every session) benefits from it:

```bash
git add SUPABASE_SCHEMA.md
git commit -m "docs: add Supabase schema reference"
```

Add it to `.gitignore` instead if you prefer to regenerate it fresh every session (e.g. when schema changes frequently).
