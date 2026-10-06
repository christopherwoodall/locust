# Distillation 2/8: GitSwarm — The Shared Computation Graph

**Source:** "GitSwarm: Decentralized Compounding Inference", arXiv 2610.04862v1
(https://arxiv.org/html/2610.04862v1), lines 318–412 (Sections 2, 2.1, 2.2).
Distilled 2026-10-06. Paraphrased in our own words; no long verbatim quotes.

## The idea in one paragraph

GitSwarm runs many identical worker agents asynchronously over one shared Git
repository. Workers decide for themselves what to explore, verify, refine, or
combine. The repository isn't just storage — it's the coordination mechanism:
every published piece of work is an atomic commit, and the graph of commits
plus *declared* dependencies becomes a record of who built on whose work,
including across branches, without anyone needing to merge.

## (a) Three required properties of shared memory

The paper argues that accumulating computation *without prescribing how agents
organize their work* demands three properties of the shared memory:

1. **Persistence — intermediate discoveries survive their authors.**
   Workers are ephemeral; their findings must outlive them. A discovery that
   dies with the episode that found it can never compound. Persistence is what
   turns N independent episodes into something more than N independent
   episodes.

2. **Branchability — competing approaches develop without prematurely
   committing to a single solution.**
   Exploration needs parallel lines of work that disagree. Forcing everything
   onto one branch (or into one consensus) too early kills the diversity that
   search depends on. Branching lets rivals mature side by side.

3. **Attribution — subsequent workers can identify dependencies between
   contributions, including across branches.**
   Cross-branch reuse only works if workers can *find* what other branches
   discovered and credit them. Attribution is the addressing system: it tells a
   worker what's out there to build on.

Together these support what the paper calls *adaptive, decentralized
control*: workers independently choose their next move from the evolving
shared state, while the harness coordinates execution (concurrency, isolation,
validation, publication) without assigning roles or a problem-solving
sequence. The harness moves the pieces; it doesn't think for them.

## (b) The shared computation graph formalism

For a task, GitSwarm initializes a Git repo with the task spec as an
immutable root commit v₀. Accepted contributions form an evolving graph
**G_t = (V_t, E_t)**, where V_t is the root plus the first t published
contributions.

- Each contribution **v_i is an atomic Git commit** with a **physical parent
  p_i ∈ V_{i−1}** — the state the worker started from.
- That parent, crucially, *need not capture all the information used* in the
  contribution.
- To record additional dependencies, each worker **declares a set of semantic
  sources U_i ⊆ V_{i−1}** in a mandatory `informed_by=` field. The physical
  parent must be included in U_i.
- The graph then contains **both** Git-parent edges **and** extra
  semantic-dependency edges: E_t = {(p_i, v_i)} ∪ {(u, v_i) : u ∈ U_i ∖ {p_i}}.
  Edges point from an earlier contribution to the one that depends on it.

## (c) The key split: physical ancestry vs declared semantic dependencies

This is the core design insight:

- **Physical ancestry is mechanical.** It says which repository state the
  worker checked out and built on. It's the diff's baseline — what was
  literally in the working tree.
- **Semantic dependencies are declared.** The `informed_by=` set says what
  actually *informed* the work — which discoveries, from any branch, shaped
  the contribution.

The two are kept separate, deliberately. And the honesty constraint matters:
semantic dependencies are *declared by workers, not independently established
as causal influence*. The system doesn't claim to prove causation — it
records the worker's own account of what informed it. That's enough for
reuse: a later worker can walk the graph and find "everything that built on
discovery X", even when X lived on another branch.

## (d) Extend one branch while citing discoveries from others — no merge required

Because the semantic layer sits on top of the Git layer, a worker can:

- physically extend branch A (parent = A's tip), while
- declaring semantic dependence on discoveries from branch B (in U_i),

without merging B into A. The graph captures the cross-pollination; the
branches stay divergent. Reuse becomes *linking*, not *absorbing*. This is how
knowledge flows laterally across competing lines of work while each line
retains its independence.

## (e) Why competing approaches are preserved rather than collapsed early

Branchability exists so no one has to pick a winner before the evidence is
in. The graph model makes preservation cheap and visible:

- Competing approaches are just branches — coexisting by construction.
- Cross-branch citations (`informed_by`) let good ideas propagate between
  branches *without* forcing consolidation, so diversity of approach doesn't
  block knowledge flow.
- Selection (which branch wins) is deferred to a separate, decentralized
  readout process (Section 2.5), keeping the *search* phase distinct from the
  *selection* phase.

Premature collapse is the failure mode: merge everything into one line and
you lose the parallel experiments that the later selection step needs to
choose between.

## Relevance to locust

This section is the data-model blueprint for the locust vault — and it maps
surprisingly well onto what the vision brief already sketches:

- **The vault as G_t.** A git-backed Obsidian vault where each committed
  memory/artifact is a node, and every write carries an explicit "what
  informed this" provenance set, is exactly the shared computation graph.
  Obsidian's links give human navigation; the commit graph plus a mandatory
  `informed_by`-style field give machine-navigable attribution. This is the
  attribution requirement operationalized.
- **Branchability in a memory vault.** Today the brief assumes one main line
  of vault history. GitSwarm says: let agents branch freely when exploring
  competing hypotheses or designs, and let cross-branch citations propagate
  discoveries without merges. Open question for build phase: do we want
  branch-per-investigation in the vault, or is one main line plus explicit
  provenance enough for v1? (Keep it simple favors the latter; this paper
  favors keeping the former available.)
- **Memory channel provenance.** The memory channel's write convention should
  include "what did you read before writing this" — the declared semantic
  sources. That one field is what turns a pile of markdown into a
  *compounding* memory. Without it, later agents can't trace why something
  was written or what it's safe to supersede.
- **Declared, not proven, attribution.** The paper's honesty note transfers
  directly: when an agent writes a memory citing sources, we record its
  *claim* of what informed it. The system doesn't need to verify causality —
  it needs to preserve the claim, because that's what later workers navigate
  by. This also bounds scope: the secret filter and validation sit on the
  write path; the graph itself is just faithful recording.
- **Separation of search and selection.** GitSwarm defers picking a winner to
  a later readout. For locust: agents (and the human) can accumulate divergent
  notes/plans across the swarm; deciding which one is canonical is a separate
  step, not a precondition for writing. The command channel is a natural home
  for that selection traffic.

Epistemic note: this distillation covers only the formalism (Sections 2–2.2).
Evidence for whether the graph actually compounds in practice lives in the
experiments (Sections 3+), covered by other distillation tasks in this batch.
