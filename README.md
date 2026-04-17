# Claude as a Linux Desktop Manager — Notes & Workflow Scaffold

Planning repo for how Claude Code (or any agentic CLI) can be used to manage a Linux desktop / server: capturing Daniel's raw notes, refining them, and layering in analysis so a workflow emerges over time.

## The pipeline

```
inbox/        →   refined/      →   analysis/       →   workflows/
raw dumps         cleaned notes     Claude's take      synthesized patterns
```

1. **`inbox/`** — raw, unedited notes. Voice-typed dumps, quick thoughts, half-formed ideas. No structure required. Filename: `YYYY-MM-DD-slug.md`.
2. **`refined/`** — Claude rewrites an inbox note into a clean, structured version. Same filename, with a frontmatter pointer back to the source.
3. **`analysis/`** — Claude's commentary on the refined note: what's novel, what conflicts with other notes, what's actionable, what's missing. One analysis file per refined note.
4. **`workflows/`** — concrete, repeatable workflow patterns distilled from one or more analyses. These are the "outputs" of the repo — the reusable playbooks.

Supporting files and directories:

- **`modules.md`** — the organizing spine: the buckets of common operations (debugging, install/maintenance, file organization, network/SSH, space optimization, etc.), each with routine ops, AI value-add, existing commands, and gaps.
- **`source-material/`** — read-only reference copies (e.g. the existing `linux-desktop-plugin` with 275 mapped commands).
- **`topics/`** — cross-cutting topic pages (package management, audio, backups, MCP, etc.) that link into the relevant refined notes and workflows.
- **`templates/`** — starter templates for each pipeline stage.
- **`INDEX.md`** — rolling index of what's in the repo (what's captured, what's been refined, what's been analyzed).

## How to use it with Claude

- **Drop a new note**: put a file in `inbox/` (or paste a thought and ask Claude to file it).
- **Refine**: ask Claude to promote inbox notes → `refined/`. It rewrites for clarity and preserves the source link.
- **Analyze**: ask Claude to produce `analysis/` entries — these are where Claude adds value beyond transcription.
- **Synthesize**: ask Claude to update or create a `workflows/` entry once enough analysis exists on a topic.
- **Re-index**: ask Claude to update `INDEX.md` and the relevant `topics/` pages.

The loop is: capture → refine → analyze → synthesize → index.

## Conventions

- Dates: `YYYY-MM-DD` in filenames (sortable), `DD/MM/YY` in prose.
- Filenames: lowercase, hyphen-separated.
- Frontmatter: every note/analysis/workflow carries YAML frontmatter with `source:`, `status:`, `topics:`, `created:`, `updated:`.
- Stay flat within each stage — no deep nesting. Use `topics/` and frontmatter tags for cross-cutting structure.
