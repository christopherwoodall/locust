# GitSwarm paper distillations — index

**Source paper:** "GitSwarm: Decentralized Compounding Inference" (Vedant Shah, Anirudh Goyal, et al., Meta Superintelligence Labs) — [arXiv 2610.04862v1](https://arxiv.org/html/2610.04862v1)

Each file below is an original-terms distillation of part of the paper, produced 2026-10-06 by a fan-out of reader agents. Every file ends with a "Relevance to locust" section mapping the paper's findings onto this project. No long verbatim passages from the paper are reproduced.

## Reading order

1. [01-compounding-inference-paradigm.md](01-compounding-inference-paradigm.md) — the paradigm itself: why inference-time work should persist and compound; the paper's three claims; limitations (causal-attribution gap, overhead); the RL-for-collective-discovery future.
2. [02-shared-computation-graph.md](02-shared-computation-graph.md) — the data model: persistence / branchability / attribution; the G_t computation graph; physical parent vs mandatory `informed_by=` semantic dependencies. **The vault's blueprint.**
3. [03-worker-protocol-and-readout.md](03-worker-protocol-and-readout.md) — the agent contract: async protocol (budget B, concurrency C), Contribute / Nominate / Abstain, live dashboard + in-flight logs, decentralized plurality readout.
4. [04-benchmarks-and-scaling.md](04-benchmarks-and-scaling.md) — results: ProgramBench ~79.4% vs ~65.1% forced baseline; 30/30 IMO proofs; scaling curves; the Forced baseline's purpose.
5. [05-sustained-research.md](05-sustained-research.md) — the three GPU-backed architecture tasks (RMT, Looped Transformer, NanoChat): cross-lineage inheritance, failure recovery, negative-result reuse.
6. [06-accumulation-evidence.md](06-accumulation-evidence.md) — the metrics that prove reuse: 94.7% of contributions built upon; 82–93% transitive ancestry; artifact consumption stages (Accessed → Actively reused → Winner-linked); workflow archetypes.
7. [07-coordination-dynamics-and-tradeoffs.md](07-coordination-dynamics-and-tradeoffs.md) — emergent division of labor, abstention as anti-redundancy (~1/6 workers), selective inspection (~5–7 commits each), async-vs-sync and decentralized-vs-central readout experiments.
8. [08-related-work-and-worker-prompts.md](08-related-work-and-worker-prompts.md) — positioning vs prior work (incl. the concurrent Agora); the worker-prompt architecture (prompt assembly, provenance format, required artifacts) — **the paper's closest thing to an implementation spec.**

## The one-paragraph version

GitSwarm organizes inference-time computation so intermediate work persists and compounds: homogeneous agents work asynchronously in a shared, branchable Git repo; every contribution is an atomic commit with a declared `informed_by=` dependency set; workers Contribute, Nominate, or Abstain; final selection is a deterministic plurality vote. The result: 94.7% of contributions get built upon, selected solutions' ancestry covers 82–93% of the graph, and the swarm beats matched-compute single-agent baselines on proofs, programs, and sustained GPU research.
