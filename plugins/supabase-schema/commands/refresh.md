---
description: Re-fetch the Supabase schema and update SUPABASE_SCHEMA.md
allowed-tools: Bash
---

# supabase-schema: refresh

Re-run the schema extractor and overwrite `SUPABASE_SCHEMA.md` with the latest table definitions from Supabase.

## Instructions

Run the extraction script:

```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/extract-schema.js"
```

After it completes, confirm to the user:
- How many tables were found
- How many RPC functions were found
- The path where SUPABASE_SCHEMA.md was written
