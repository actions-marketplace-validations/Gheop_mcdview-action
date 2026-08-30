# mcdview-action

Generate or update an interactive ER diagram from a database schema on
[mcdview.dev](https://mcdview.dev) — straight from your CI. No account, no
install: one HTTP call, a stable link you keep in your README and refresh on
every build.

The generator is open source (MIT): [github.com/Gheop/mcdview](https://github.com/Gheop/mcdview).

## Create a diagram (once)

```yaml
- uses: Gheop/mcdview-action@v1
  id: mcd
  with:
    file: db/schema.sql
    title: My database
- run: echo "Diagram at ${{ steps.mcd.outputs.url }} — keep ${{ steps.mcd.outputs.gestion }} as a secret"
```

Copy the `gestion` URL's key into a repository secret (`MCDVIEW_KEY`). It is the
only way to update or delete the diagram, so keep it safe.

## Update on every push (stable link)

```yaml
name: er-diagram
on:
  push:
    branches: [main]
jobs:
  mcdview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Gheop/mcdview-action@v1
        id: mcd
        with:
          file: db/schema.sql
          key: ${{ secrets.MCDVIEW_KEY }}
      # from the 2nd version on, a diff link is available:
      - if: steps.mcd.outputs.diff
        run: echo "Schema changed — diff: ${{ steps.mcd.outputs.diff }}"
```

The public link never changes; each run adds a version (history and rollback on
the management page).

## Fail the build on a bad schema

`mcdview --diagnose` runs on every upload. Gate your build on it:

```yaml
- uses: Gheop/mcdview-action@v1
  with:
    file: db/schema.sql
    key: ${{ secrets.MCDVIEW_KEY }}
    fail-on: ok        # fail unless the diagnostic is "ok"
```

`fail-on: anomaly` fails only on `no_table` / `error` (a likely parser or format
problem), tolerating content anomalies. `off` (default) never fails on the
diagnostic.

## Inputs

| input | required | default | description |
|-------|----------|---------|-------------|
| `file` | yes | | schema file: `.sql` (PostgreSQL/MySQL/SQLite and ~15 dialects), `.dbm`, `.dbml`, `.prisma`, `.mwb`, `.rb`, `.mmd`, `.ts` |
| `key` | no | | management key to update an existing diagram (a secret); omit to create |
| `title` | no | | diagram title (creation only) |
| `lang` | no | `en` | diagram language, `fr` or `en` (creation only) |
| `expire` | no | `0` | ephemeral link: days before auto-deletion (creation only); `0` = permanent |
| `fail-on` | no | `off` | fail when the diagnostic is worse than `ok` / `anomaly`; `off` disables |

## Outputs

| output | description |
|--------|-------------|
| `url` | public URL of the interactive diagram |
| `gestion` | management URL (creation only) |
| `diff` | diff URL (updates only, from the 2nd version) |
| `status` | diagnostic status: `ok`, `no_table`, `anomaly`, `error` |

## License

MIT
