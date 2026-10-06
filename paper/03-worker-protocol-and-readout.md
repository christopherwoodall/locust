# 03 — Asynchronous Worker Protocol, Live Swarm State, Decentralized Readout

**Source:** "GitSwarm: Decentralized Compounding Inference" — arXiv 2610.04862v1, Sections 2.3, 2.4, 2.5
**URL:** https://arxiv.org/html/2610.04862v1
**Distilled:** 2026-10-06. Paraphrased in my own words; no long verbatim quotes.

---

## 1. The asynchronous worker protocol (§2.3)

A GitSwarm run is governed by two numbers: a **budget B** of *charged* worker episodes, and a **maximum concurrency C**. The harness keeps up to C workers active at any time, launching replacements as slots open up.

- **Homogeneous, role-less workers.** Every worker runs the same model and follows the same behavioral contract. There are no predefined roles — no manager, no judge, no reviewer. Differentiation emerges from what each worker chooses to do, not from an assigned title.
- **Isolated Git worktrees.** Each worker operates in its own isolated Git worktree. It inspects the repository, picks a starting commit, and independently decides how to advance the task.
- **Three terminal actions.** Every episode ends with exactly one of:
  1. **Contribute** — publish one *validated, atomic commit* containing new work: a partial solution, an experiment, a test, or a refinement.
  2. **Nominate** — pick an existing candidate and publish a separate *nomination record* carrying a rationale, without modifying the candidate itself.
  3. **Abstain** — terminate with no publication when no useful additional work is identified, or when equivalent work is already underway elsewhere.
- **Abstention is free of budget charge.** A voluntary abstain triggers a replacement worker without consuming B (though it still costs inference compute). All other episode outcomes consume the budget. This creates an incentive structure where workers self-allocate effort adaptively — exploration, refinement, experimentation, verification, integration, and selection — because spending a budgeted episode on duplicated or useless work is wasteful, while stepping aside costs nothing against the budget.
- **Validation enforces publication requirements, not correctness.** The gate a contribution must pass checks that it meets task-specific *publication* requirements (format, completeness, hygiene) — it does not attempt to certify that the solution is *correct*. Correctness is settled elsewhere (by the decentralized readout / by validation scoring), keeping the write path permissive for honest partial work while screening out garbage.

## 2. Live swarm state (§2.4)

Workers see the current state of the run — and of each other — through two interfaces that **report state but never assign work**. There is no dispatcher, no task queue, no orchestration. The paper is explicit that these are read-only awareness surfaces.

**a. Dashboard (`gs_dashboard`).** Each worker gets a snapshot at spawn time and can query the live state. It reports:
- remaining worker budget,
- workers currently in flight,
- the nomination tally — *explicitly flagged as not evidence of correctness*.

(For research tasks run against wall-clock budgets, budget/nomination fields are swapped for shared GPU/CPU/job usage, a low-utilization warning, and the best validation score with its commit.)

**b. In-flight activity (`gs_inflight`).** Uncommitted work is invisible in the repository, so each worker maintains a **public activity log**: it announces its scope — the gap it's addressing, the deliverable it aims for, its success and abort criteria — *before* editing, then posts progress updates. Peers can list active workers, read a worker's log, and inspect its uncommitted changes, but cannot modify them. The stated purpose: help workers avoid taking up work already being undertaken by an in-flight peer.

Together: the dashboard says *what the run looks like*, the activity logs say *who is doing what right now*. Both are observation channels, never command channels.

## 3. Decentralized readout (§2.5)

Consensus on the final commit is formed across the workers, not delegated to a privileged judge:

- After the worker budget is exhausted, the harness selects the candidate with the **most nominations** — a deterministic plurality vote over nomination records, with deterministic tie-breaking.
- **Nomination records live outside the computation graph.** Evaluating a candidate does not alter its ancestry: workers can assess existing candidates while others keep constructing new ones, and the eventual winner's lineage is never polluted by the act of voting on it.

Coupled with the earlier sections: persistent commits, cross-branch reuse, and decentralized worker decisions let computation accumulate across otherwise independent episodes.

---

## Relevance to locust

- **The coordination contract for a joining agent.** GitSwarm's worker protocol is essentially a membership contract for any agent that joins a shared-state swarm: (1) you work in isolation against a shared substrate and publish atomic, validated units of work; (2) you may endorse others' work with a rationale, but endorsement never mutates the endorsed artifact; (3) you abstain — and get replaced — when you have nothing useful to add, at no penalty. A locust skill can encode exactly this: *same behavioral contract for every agent, no privileged roles, publish don't commandeer.*
- **Abstention as a first-class protocol action** maps onto locust's "do nothing is a valid move" posture — a joining agent should know that stepping aside is cheaper than duplicating a peer's in-flight work, and the system must make that cheap by design.
- **The in-flight logs are a direct precedent for the command channel.** GitSwarm's `gs_inflight` is precisely what locust's command channel is for: a public, read-only-by-peers announcement of *scope, gap, deliverable, success/abort criteria* before touching shared state, plus progress updates. The command channel's core convention could be: every agent announces intent before acting, peers can observe but not commandeer, and the purpose is collision-avoidance, not task assignment.
- **State interfaces report, never assign.** This is a design rule worth lifting verbatim into locust's channel conventions: dashboards and activity feeds give workers awareness; no surface in the system hands out orders. That keeps the swarm decentralized even as it scales — coordination emerges from shared visibility, not from a scheduler.
- **Validation as publication-gate, not truth-gate.** Locust's secret filter and write-path checks should follow the same split: enforce *publication requirements* (no secrets, well-formed, attributable) on every write, but leave *correctness* of the memory content to be judged by readers and by accumulated evidence — never block honest partial knowledge at the gate.
- **Readout outside the computation graph.** Endorsement metadata (nominations, upvotes, flags) should live alongside — not inside — the memory pages it evaluates, so evaluating a page never rewrites its history. In an Obsidian vault: ballot/sidecar files, never in-place edits to the page being judged.
