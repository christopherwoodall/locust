# Locust: shared memory layer for the swarm — vision brief

**Date:** 2026-10-06. **Status:** read-and-distill only; no build decisions made. **Evening update:** added link-memory (summaries + deep reach) and multi-project namespace sections per his word.

## Elevator pitch

Locust is a shared memory layer connecting every agent and one human into a single navigable system. The shared state lives in an Obsidian vault that is also a GitHub repo ("locus-vault"), so both humans and agents can read, write, and navigate it with tools they already know. Distribution is a single skill: hand any agent the skill and it learns where the memory is, how the channels work, and how to join the swarm — "sort of like the borg".

## Problem statement

Agents that only talk to one human forget everything between sessions, and they cannot see what sibling agents know. Today's per-agent memory (notes, indexes, scattered files) is write-once and read-never by anyone except the agent that wrote it. The result is duplicated work, lost context, and no accumulation of knowledge across the fleet. The missing piece is not a smarter model — it is a shared, navigable, durable substrate that every agent can read from and write to, in a form the human can also open, search, and understand.

## Core components

### a. The `locust` repo (code + the skill)

The home repository for the project. It holds the implementation and, critically, the skill itself: the single artifact that teaches an agent how to use the system. Keeping the skill in-repo means the onboarding path is one action — hand an agent the skill — and the skill's version is pinned to the code's version. The repo's current contents (paper distillations, vision briefs) are the seed of the design process.

### b. The `locus-vault` repo (Obsidian vault on GitHub)

The shared memory itself. An Obsidian vault gives the memory a form humans can open and navigate directly — pages, links, backlinks, graph view — while remaining plain markdown files that agents can read and write without special tooling. Hosting it on GitHub makes it the canonical remote: clone/pull for read, commit/push for write, with full history and the option to host other things alongside the memory (indexes, configs, auxiliary artifacts — "something like a repo called locus-vault where we can also add other things").

### c. Discord auto-onboarding

A nice-to-have: the system should be able to onboard itself to a Discord server, so the swarm gets a communication surface without manual setup. Explicitly deferred from the core — desirable, not required.

## Design principles

### Plugin architecture

The system grows by plugins, not by rewriting the core. Channels (see below) and capabilities (Discord bridge, new memory types, new filters) should be swappable modules with a small stable interface, so one contributor's lane cannot break another's.

### Secure by design

Security is a property of the architecture, not a checklist applied afterward. Assume hostile or careless inputs at every boundary: agents write memory, agents read memory, and agents act on what they read. Every layer — storage, filtering, channels, plugins — must treat untrusted content as the default case.

### Git-backed

The vault is a git repository. That buys durability, history, blame, branching, and merge semantics for free, using infrastructure everyone already understands. Conflicts, rollbacks, and audit trails are git operations, not custom code. The GitHub repo is the source of truth; local clones are working copies.

### Secret filtering

Nothing secret enters the shared memory, ever. A filtering layer sits on the write path: credentials, keys, tokens, and personal identifiers are detected and blocked (or redacted-at-the-boundary with an explicit, logged override) before they land in the vault. Since the vault is a public-or-shared GitHub repo by design, a secret that reaches it is a secret burned — the filter must be fail-closed.

### MVC pattern

Model / view / controller separation for the system's internals: the model is the vault content and its schema, the view is how it is rendered (Obsidian UI for humans, file/structured reads for agents, Discord surfaces), and the controller is the logic that reads, writes, filters, and routes between channels. New views and controllers plug in without touching the model.

### Keep it simple (just a skill)

The whole system must be expressible as a skill. No dashboards to install, no services to deploy, no accounts to provision before an agent can participate. If a design choice requires infrastructure beyond "clone the repo, read the skill, act," it loses to a simpler one. The skill is the distribution mechanism and the documentation.

## Channels

"Lanes" was the working term; "channels" is the chosen one. A channel is a named partition of the system's traffic — a scoped place where a specific kind of activity happens, with its own conventions for what gets written, who writes it, and how it is read back.

- **Memory channel:** the read/write substrate. This is the vault content itself — durable shared knowledge, indexed and linkable. Writes go through the secret filter; reads are open to any swarm member. The channel where the swarm's knowledge compounds.
- **Command channel:** the control plane. Operational traffic — tasking, status, coordination signals between agents and the human. Distinct from memory because commands are ephemeral and addressed, not archival and browsable.

The architecture leaves room for more channels (an alerts channel, an artifacts channel, per-project channels) without redesign: adding a channel is adding a namespace with conventions, ideally as a plugin, not a core change.

## Link memory: summaries with deep reach

The swarm constantly encounters links — papers, articles, docs, repos, dashboards, threads. Losing them means re-finding them later; dumping raw captures means an unsearchable pile. The convention:

- **Every saved link gets a note.** The note carries the URL, when and why it was saved, a concise summary (what it says, why it matters to us), key extracts or quotes, and project tags. The note — not the raw page — is the searchable unit.
- **Two tiers.** (1) The *summary layer*: compact, always loaded, searchable. Agents work from summaries by default. (2) The *deep layer*: the full content, captured at save time or fetched on demand. When the situation calls for it, the agent reaches past the summary into the deep capture.
- **Capture at save time matters.** Links rot, pages change, paywalls appear. Saving a snapshot — or at minimum the summary plus key quotes — at capture time preserves what we actually saw, not what the URL serves six months later.
- **Proposed (not final) frontmatter for a link note:** `url`, `saved_at`, `saved_by`, `projects[]`, `summary`, `key_points[]`, `deep_capture` (path to a snapshot, or `on-demand`).

The principle: search the summaries, reach into the depths. The vault stays navigable because the deep stuff is one hop away, never in the way.

## Multiple projects

This system is not one project — it is the memory for all of them. An agent may work the locust build today and the Tennessee code review tomorrow; both need memory, and neither should pollute the other.

- **Projects are namespaces.** Every memory-channel write belongs to a project — a top-level folder, a frontmatter field, or both (mechanics TBD). Reads default to the agent's active project; cross-project search is explicit, not accidental.
- **Cross-project links are first-class.** A finding in one project can reference a note in another without copying it. The vault is one graph; projects are views over it.
- **Onboarding scopes the agent.** Joining the swarm and being attached to project(s) are two steps of the same handoff. The skill teaches both: where the swarm lives, and which project(s) this agent works in.
- **Project lifecycle.** Projects get created, go dormant, get archived. The vault keeps all of them searchable without letting dormant projects noise up the active set. Archive conventions TBD.

The `locus-vault` "other things" clause from the original brief now has its primary meaning: the vault hosts many projects' memories side by side, plus indexes and auxiliary artifacts.

## Onboarding flow: joining the swarm

1. **Handoff:** the human (or a parent agent) gives a new agent the locust skill. That single artifact is the whole package — no separate setup docs, no credential ceremony.
2. **Discovery:** following the skill, the agent locates the system: the `locust` repo for code/conventions, the `locus-vault` repo as the shared memory remote.
3. **Access:** the agent clones or otherwise gains read/write access to the vault. The access path is whatever the skill documents; the principle is that access is part of the skill, not a separate manual step.
4. **Knowledge:** the agent reads the vault's structure and the channel conventions — what the memory channel holds, what the command channel is for, how to write without tripping the secret filter.
5. **Join:** the agent announces itself on the command channel and begins reading from and writing to the memory channel. It is now part of the swarm — assimilated, "sort of like the borg," except the collective memory is a git repo you can open in Obsidian.

## Open questions / deferred decisions

Explicitly not decided in this phase — build comes later:

- **Discord auto-onboarding** is a nice-to-have, not a requirement. How the swarm surfaces on Discord (bot? webhook relay? full plugin?) is undecided.
- **Vault hosting and access control:** public vs private repo, who gets push rights, how an agent's writes are attributed and reviewed.
- **Secret-filter mechanics:** block vs redact, the detection list, who approves overrides, and how a burned secret is rotated out of history.
- **Conflict resolution:** merge semantics when two agents write the same page; human-in-the-loop or automatic.
- **Channel taxonomy:** which channels ship at v1 beyond memory and command, and the exact conventions each enforces.
- **Skill packaging:** what the skill bundles (scripts? schemas? just docs?) and how it stays in sync with the repos.
- **The `locus-vault` "other things":** what else lives in the vault repo besides memory, and where the line is.
- **Link memory mechanics:** snapshot-at-save vs live-fetch-on-demand for the deep layer; where snapshots live and how big they may get; who writes the summary and in what format; how deep captures stay fresh or get marked stale.
- **Project namespace mechanics:** folder-per-project vs frontmatter-only vs vault-per-project; the default read scope for an agent; how cross-project search and linking work in practice; archive conventions for dormant projects.

## Relation to GitSwarm

The `paper/` directory in this repo holds distillations of the GitSwarm paper on Decentralized Compounding Inference (distilled separately; not read for this brief). No claim is made here about the paper's content. The conceptual overlap, as framed by the project's own structure, is suggestive and worth tracking during the build: a git-backed substrate for shared state, decentralized workers reading and writing it, and knowledge that compounds across participants over time. Whether GitSwarm's mechanisms map onto locust's channels, plugins, or filter design is an open question for the build phase — note it, don't assume it.
