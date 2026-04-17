# workflows/

Synthesized, reusable workflow patterns for using Claude Code (or any agentic CLI) to manage a Linux desktop or server. These are the "outputs" of the pipeline — the playbooks worth keeping.

A workflow entry should include:

- **Trigger**: when you'd reach for it.
- **Preconditions**: what must be true (permissions, tools installed, MCPs available).
- **Steps**: the actual flow — what Claude does, what the user does.
- **Guardrails**: what to avoid, what to confirm before acting.
- **Source notes**: links to the `analysis/` and `refined/` entries it came from.
