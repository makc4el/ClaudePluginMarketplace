# vanbrunt

Claude Code plugin for vanbrunt project release utilities.

## Commands

### `/vanbrunt:release`

Runs the master sync git workflow:

1. Syncs `master` and `staging` branches with remote
2. Creates a dated branch: `modarchenko/master_sync_YYYYMMDD_n1`
   - Auto-increments suffix (`n1` → `n2` → `n3`) if branch already exists
3. Resets soft to `master` (keeps staging changes staged)
4. Commits with message `Master Sync YYYYMMDD n1`
5. Pushes with upstream tracking

## Usage

```
/vanbrunt:release
```

Run from within the vanbrunt repository (or any repo using this workflow).
