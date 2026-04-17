# Modules — Common Operations for Agentic Desktop Management

This is the organizing spine of the repo. Each module is a **bucket of related operations** where Claude (or any agentic CLI) can provide meaningful value-add over a plain terminal. Modules are defined by:

- **Scope** — what lives in this bucket, what doesn't
- **Routine operations** — the pre-bucketed common tasks
- **AI value-add** — what makes this worth an agent, vs. bash/scripts/man pages
- **Existing commands** — slash commands already mapped in `source-material/linux-desktop-plugin/`
- **Gaps** — what's missing

Cross-cutting filters:
- "Is this routine?" (runs frequently → worth a command)
- "Does the agent add value?" (interpretation, correlation, judgement → yes; mechanical → maybe script instead)

---

## 1. Debugging & Crash Diagnosis

**Scope**: Post-incident forensics and live troubleshooting. Crashes, freezes, slowdowns, failed services, boot problems, unexplained behavior.

**Routine operations**
- Diagnose a crash that just happened (journalctl, coredumps, dmesg correlation)
- Investigate a slowdown (CPU/IO/memory correlation, process tree)
- Review boot (`systemd-analyze`, failed units, blame the slow units)
- Tail / analyze system logs with context
- Debug folder/file permission issues
- Bluetooth / peripheral misbehavior
- Printer diagnosis

**AI value-add**
- **Log correlation**: stitching journalctl + dmesg + app logs around an incident timestamp. A human opens three terminals; the agent reads all three and tells you the story.
- **Pattern recognition**: "this OOM happened after a Chromium + Ollama collision" — recognizing known failure modes across noisy logs.
- **Hypothesis → evidence loop**: agent proposes a cause and immediately queries for confirming/disconfirming evidence.
- **Natural-language entry point**: "why did my laptop freeze around 3pm" is faster than knowing which tool to reach for.

**Existing commands** (from plugin namespace `debugging-*`, `logging-*`)
- `debugging-diagnose-crash`, `debugging-diagnose-slowdown`
- `debugging-boot-check-boot-logs`, `debugging-boot-failed-boot-services`, `debugging-boot-review-boot`
- `logging-analyze-journal-errors`, `logging-check-failed-units`, `logging-monitor-system-resources`, `logging-tail-system-logs`
- `configuration-permissions-debug-folder-permissions`

**Gaps**
- No "post-mortem write-up" command that produces a durable record of what happened.
- No way to tag an incident and link it to future recurrences.

---

## 2. Install & System Maintenance

**Scope**: Getting software onto the machine, keeping it current, pruning what's no longer used. Spans apt/snap/flatpak/brew, GitHub releases, language package managers (pipx, npm), CLI tooling, and routine upgrades.

**Routine operations**
- Install a program (pick the best source — native apt > 3rd-party repo > snap > flatpak > brew > GH release > pip)
- Install directly from a GitHub repo URL
- Run a full system upgrade
- Configure unattended / auto-updates
- Check apt health (broken deps, held packages, unused repos)
- Audit third-party repos (are any now upstream?)
- Evaluate installed software (what did you install and still use?)
- Identify unused packages for removal
- Setup specific CLIs (aws, b2, rclone, gh, pipx, sdkman, yadm, brew)

**AI value-add**
- **Source selection**: "install Obsidian" → agent picks the right channel for your system and explains the tradeoff. Saves the 5-minute comparison tab opening.
- **Dedup detection**: spotting that the same tool is installed via apt *and* flatpak.
- **Third-party-repo rationalization**: checking whether a 3rd-party repo is still needed, or whether the package is now in official repos.
- **Post-install verification**: not just `apt install`, but confirming the binary is on PATH, the service is enabled, and a smoke test works.
- **Safer uninstall**: simulating what would be removed before doing it.

**Existing commands**
- `installation-clis-*`, `installation-install-from-gh`, `installation-install-this`, `install-package-best`, `install-github-program`
- `package-management-check-apt-health`, `-check-third-party-repos`, `-configure-auto-updates`, `-evaluate-installed-software`, `-identify-unused-packages`
- `system-health-system-upgrade`

**Gaps**
- No rollback/undo record for installs.
- No "software inventory snapshot" for reproducing this machine on a fresh install.

---

## 3. File & Folder Organization

**Scope**: Bringing order to filesystems. Sorting, consolidating, splitting, tidying, repo hygiene. Primarily `~/` and `~/repos`.

**Routine operations**
- Tidy the desktop
- Organize loose files into sensible folders
- Consolidate near-duplicate folders
- Flatten overly-nested hierarchies
- Chunk a too-large directory into subfolders
- Separate by filetype (photos/video/docs)
- Suggest a folder structure for a messy tree
- Delete old repos / stale clones
- Organize repos into the established structure (`~/repos/github/my-repos/`, etc.)

**AI value-add**
- **Semantic grouping**: the agent can *read* file contents and group by topic, not just extension. A script can't tell that three PDFs belong to the same project.
- **Naming conventions**: enforcing snake_case / YYYY-MM-DD prefixes across a messy tree.
- **Duplicate vs. near-duplicate**: deciding when two files are "the same enough" to consolidate.
- **Explanation before action**: showing the proposed reorg as a diff and asking before executing — hard to get from a shell pipeline.

**Existing commands** (namespace `fs-optimisation-*`, `repositories-*`)
- `fs-optimisation-chunk-chunk-this-dir`, `-consolidate-consolidate-folders`, `-flatten-flatten`, `-separate-separate-by-filetype`, `-separate-separate-photos-and-video`, `-tidy-up-desktop-tidy`, `-tidy-up-organize-loose-files`, `-idate-suggest-folder-structure`
- `repositories-delete-old-repos`, `repositories-organise-repos`

**Gaps**
- No "dry-run with diff preview" command that's universal across the fs ops.
- No persistent "move log" so reorgs can be reversed.

---

## 4. Network, LAN & SSH

**Scope**: Everything below the application layer that touches connectivity. LAN topology, SSH config, firewall, VPN/tunnels, network mounts, name resolution.

**Routine operations**
- Diagnose LAN connectivity (gateway ping, DNS, routing)
- Scan the LAN (ARP) and produce a device map
- Smart ARP with hostname/vendor enrichment
- Draw a network diagram
- Health-check the network stack
- List SSH hosts / manage SSH keys
- Set up LAN SSH to a new device
- Firewall analysis
- Network mounts (NFS / SMB) setup
- Wake-on-LAN configuration

**AI value-add**
- **Topology inference**: turning ARP + routing tables + DNS into a labeled diagram a human can read.
- **SSH config authoring**: writing a clean `~/.ssh/config` entry from "I want to reach nurserypi as root with the ed25519 key".
- **Firewall rule explanation**: translating an iptables/ufw dump into "what does this actually allow."
- **Correlating across layers**: "why can't I reach X" — agent pings gateway, resolves DNS, checks firewall, checks known-hosts, all in sequence.
- **Sensitive-change confirmation**: agent should pause before touching firewall or SSH authz.

**Existing commands**
- `network-lan-diagnose-lan-connectivity`, `-lan-ssh-setup`, `-scan-lan`, `-smart-arp`
- `network-diagram`, `network-health`
- `configuration-ssh-list-ssh-connections`, `-manage-ssh-keys`
- `security-firewall-analyze-firewall`
- `storage-network-mounts-setup-nfs-mounts`, `-setup-smb-mounts`
- `power-mgmt-wol-setup`

**Gaps**
- No "add new LAN host" end-to-end workflow (ARP → hostname → SSH config → known_hosts → test).
- No unified "where is this traffic going" (firewall + routing + tunnels combined).

---

## 5. Space Optimization & Cleanup

**Scope**: Disk usage. Finding what's big, what's old, what's duplicate, what's cache. Distinct from module 3 (organization) — this is about reclaiming space, not shaping structure.

**Routine operations**
- Disk usage summary (per-dir, deepest offenders)
- Find large files above a threshold
- Identify stale / rarely-accessed files
- Empty caches (browser, package manager, snap revisions, flatpak unused runtimes, journal)
- Find duplicate files
- Old repo cleanup (see also module 3)
- Ollama model pruning
- BTRFS/Snapper snapshot audit — which snapshots can go
- Drive health check (SMART) before aggressive cleanup

**AI value-add**
- **Importance ranking**: "big and old" vs "big and recent" vs "big and system-critical" — agent can tell the difference, `du` cannot.
- **Safety reasoning**: before deleting, agent checks whether the file is referenced, whether the package is still installed, whether the dir is a mount point.
- **Cache-vs-data classification**: agent knows that `~/.cache/huggingface` is safe to blow away but `~/.local/share/ollama` contains your models.
- **Explanatory report**: "you can reclaim 47 GB by doing X, Y, Z" with tradeoffs.

**Existing commands**
- `disk-usage`, `large-files`, `optimisation-large-files`
- `prune-ollama` / `ai-local-ai-ollama-prune-ollama`
- `storage-health-checks-btrfs-snapper-health`, `-check-drive-health`
- `identify-unused-packages` (overlaps with module 2)
- `repositories-delete-old-repos` (overlaps with module 3)

**Gaps**
- No single "reclaim space" dashboard that covers caches + snapshots + unused pkgs + large files + stale repos in one view.
- No "archive instead of delete" flow (tar to cold storage, then remove).

---

## 6. Adjacent modules already present (for completeness)

These weren't in the buckets you named but are already well-represented in the plugin. Flagging so we decide whether to fold them into the five core modules or keep them separate.

| Module | Existing namespaces | Merge or keep separate? |
|---|---|---|
| Hardware profiling & benchmarks | `hardware-*`, `benchmark-*`, `profile-*` | Likely **keep separate** — distinct "know thy machine" flow. |
| Display / KDE / peripherals | `display-*`, `kde-*`, `peripherals-*` | Keep separate — DE-specific. |
| Audio | `audio-*`, `optimize-pipewire`, `media-*`, codecs | Keep separate — tight scope. |
| Security (audit / av / auth / posture) | `security-*` | Keep separate — critical and distinct. |
| Dev environments (Python, Node, Docker, IDEs, yadm, SDKs) | `dev-tools-*` | Keep separate — large enough to warrant its own module. |
| AI / local AI / MCP | `ai-*` | Keep separate — rapidly evolving. |
| Backup | `backup-*` | Keep separate. |
| Power management / hibernation / WOL | `power-mgmt-*` | Could fold into Hardware. |
| Fonts | `fonts-*` | Minor, could fold into System Maintenance. |

---

## Open design questions

1. **Command naming**: the existing plugin mixes `module-subfolder-verb` (`network-lan-scan-lan`) with shorter aliases (`scan-lan`). Do we want one canonical form and redirects, or keep both?
2. **Sub-bucketing**: should each module have formal sub-buckets (e.g. debugging → `boot/`, `runtime/`, `perm/`), or stay flat with good naming?
3. **Skill vs command boundary**: some operations fit a skill better (interactive, multi-turn), others fit a command (single-shot). Needs a written rule — see `workflows/skill-vs-command.md` (TODO).
4. **Persistence**: several gaps above ("post-mortem write-up", "reorg undo log", "software inventory snapshot") share the pattern "the agent did something; we want a record." Could be a single cross-cutting pattern (`ops-log/` dir per machine?).
