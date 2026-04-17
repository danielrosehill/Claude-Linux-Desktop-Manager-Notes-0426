# Plugin + Workspace Pairing — Execution Plan

Date drafted: 2026-04-17

## Goal

Evolve `linux-desktop-plugin` from a stateless bag of 240+ commands into a paired system:

- **Plugin** (`linux-desktop-plugin`) — stateless operations. Runs anywhere. Unchanged behaviour when no workspace is present.
- **Workspace** (`desktop-ops-workspace-template`) — per-machine state: machine profile, ops log (SQLite), templates, CLAUDE.md contract.

Discovery via `DESKTOP_OPS_WORKSPACE` env var + a `.desktop-ops-workspace` marker file. No hard dependency either way.

---

## Storage: SQLite primary

The ops log is a local SQLite file — `ops.db` — gitignored. Rationale:

- Ops records are structured (timestamp, machine, op_type, package, outcome, error). 90% of queries are `WHERE` filters, not similarity search.
- Claude writes SQL fluently; `sqlite3` is on every Linux box.
- One file, no service, no network, no API key.
- Private data stays local — never committed, never sent to a cloud vector DB.
- If semantic "find similar past incidents" becomes a real need, add the `sqlite-vec` extension later — same file, additive.

### Schema sketch

```sql
CREATE TABLE machines (
  id INTEGER PRIMARY KEY,
  hostname TEXT UNIQUE NOT NULL,
  distro TEXT,
  kernel TEXT,
  de TEXT,
  hardware_summary TEXT,
  first_seen TEXT NOT NULL,
  last_seen TEXT NOT NULL
);

CREATE TABLE ops (
  id INTEGER PRIMARY KEY,
  ts TEXT NOT NULL,                 -- ISO8601
  machine_id INTEGER NOT NULL REFERENCES machines(id),
  op_type TEXT NOT NULL,            -- incident|install|reorg|fix|inventory|readonly
  command TEXT,                     -- plugin command name, e.g. debugging-diagnose-crash
  title TEXT NOT NULL,              -- one-line summary
  body TEXT,                        -- markdown details
  outcome TEXT,                     -- success|partial|failure|n/a
  tags TEXT                         -- comma-separated: bluetooth,pipewire,ollama...
);

CREATE INDEX idx_ops_machine_type_ts ON ops(machine_id, op_type, ts DESC);
CREATE INDEX idx_ops_tags ON ops(tags);

CREATE TABLE installs (                -- specialised view for package history
  id INTEGER PRIMARY KEY,
  op_id INTEGER NOT NULL REFERENCES ops(id),
  package TEXT NOT NULL,
  source TEXT,                         -- apt|snap|flatpak|brew|gh|pip|...
  version TEXT,
  action TEXT NOT NULL                 -- install|remove|upgrade
);

CREATE INDEX idx_installs_package ON installs(package);
```

Specialised tables (like `installs`) can grow over time; `ops` stays the common spine.

---

## Part 1 — Update the plugin

Given SQLite-primary, the plugin update is minimal.

1. **Bump version** to 2.0.0.
2. **README**: add "Pairs with `desktop-ops-workspace-template`" section; keep standalone-use docs.
3. **No per-command edits needed** — the workspace's `CLAUDE.md` carries the logging contract.

---

## Part 2 — Create the workspace template repo

**Repo**: `desktop-ops-workspace-template` — marked as GitHub template.

**Structure**:

```
desktop-ops/
├── .desktop-ops-workspace       # marker file, empty
├── .gitignore                   # ignores ops.db, ops.db-*, machine.local.md
├── CLAUDE.md                    # workspace contract — see below
├── .claude/
│   └── settings.json            # exports DESKTOP_OPS_WORKSPACE=${CLAUDE_PROJECT_DIR}
├── README.md                    # onboarding
├── schema.sql                   # creates ops.db on first run
├── machine.md                   # public machine profile (committed)
├── machine.local.md             # private profile notes (gitignored)
├── templates/                   # markdown templates Claude fills for `body` field
│   ├── incident.md
│   ├── install.md
│   ├── reorg.md
│   └── fix.md
└── scripts/
    ├── init-db.sh               # sqlite3 ops.db < schema.sql
    ├── register-machine.sh      # inserts/updates row in machines table
    └── log.sh                   # helper: log.sh <op_type> <title> [--command X] [--tags a,b]
```

### `CLAUDE.md` contract (workspace)

Tells Claude:

- You are in the desktop-ops workspace for machine `<hostname>`.
- The ops log lives in `ops.db` (SQLite). Schema in `schema.sql`.
- **Before any destructive op**: query `ops` for prior entries — same command, same package, same tag — summarise relevant history to the user.
- **After any non-readonly op**: insert a row. Use `scripts/log.sh` or direct SQL. `op_type` inferred from the command namespace (`install-*` → install, `debugging-*` → incident, `fs-optimisation-*` → reorg, etc.).
- Keep `machine.md` accurate — update when hardware, distro, or major config changes.
- Never commit `ops.db` — it's gitignored on purpose.

### First-run bootstrap

`scripts/init-db.sh` runs `sqlite3 ops.db < schema.sql` if `ops.db` is missing. `register-machine.sh` inserts the current host from `hostnamectl` + `lscpu` + `lsblk` + GPU detection.

---

## Part 3 — Publish

1. **Plugin**: commit version bump + README, push to `github.com/danielrosehill/linux-desktop-plugin`.
2. **Workspace**: create `danielrosehill/desktop-ops-workspace-template`, push, enable "Template repository" in settings.
3. **Cross-link** both READMEs.

---

## Onboarding flow

```bash
# 1. Create your personal workspace from the template
gh repo create my-desktop-ops --template danielrosehill/desktop-ops-workspace-template --private --clone
cd my-desktop-ops

# 2. Bootstrap — creates ops.db, registers this machine
./scripts/init-db.sh
./scripts/register-machine.sh

# 3. Install the plugin (standard Claude Code plugin install)

# 4. Open Claude Code in the workspace dir — settings.json sets DESKTOP_OPS_WORKSPACE

# 5. Run any /command — Claude logs structured rows into ops.db automatically
```

---

## Example queries (what this buys you)

```sql
-- Every bluetooth issue on this machine
SELECT ts, title, outcome FROM ops
WHERE machine_id = (SELECT id FROM machines WHERE hostname = 'current')
  AND tags LIKE '%bluetooth%'
ORDER BY ts DESC;

-- Has this package been reinstalled multiple times?
SELECT package, COUNT(*) FROM installs
GROUP BY package HAVING COUNT(*) > 1;

-- Last 10 incidents across all machines
SELECT m.hostname, o.ts, o.title, o.outcome
FROM ops o JOIN machines m ON m.id = o.machine_id
WHERE o.op_type = 'incident'
ORDER BY o.ts DESC LIMIT 10;
```

Claude writes these ad hoc from natural-language questions — the schema is small enough to fit in context.

---

## Open questions

1. **Workspace visibility** — template repo public (reusable by others) or private?
2. **Plugin marketplace** — is `linux-desktop-plugin` listed in a Claude Code marketplace that needs a version bump too?
3. **Multi-machine sync** — `ops.db` is gitignored per-machine. If user wants cross-machine history, future option: commit a nightly `ops.export.jsonl` (redacted) or sync `ops.db` via a separate private channel (rclone to B2, etc.). Out of scope for v1.

---

## Recommended order of execution

1. Build workspace template repo locally (`schema.sql`, `CLAUDE.md`, templates, scripts, `.gitignore`).
2. Smoke-test: init-db → register-machine → insert a fake op row → query it.
3. Minimal plugin update (README + version bump to 2.0.0).
4. Push both repos; mark workspace as GitHub template.
5. End-to-end test on this machine: run one real plugin command, confirm Claude writes a row to `ops.db`.
