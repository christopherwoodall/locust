# GitSwarm distillation 4: Benchmarks and inference-time scaling

**Source:** "GitSwarm: Decentralized Compounding Inference", arXiv 2610.04862v1
https://arxiv.org/html/2610.04862v1 — Section 3.1 (Experimental Setup), Section 3.2 (Benchmark Performance and Inference-Time Scaling), Appendix D (Benchmark Performance Details).

## a. Experimental setup

Two fixed-answer benchmark settings:

- **IMOProofBench-Advanced** — 30 mathematical proof problems (all 30 evaluated).
- **ProgramBench** — program reconstruction from documentation plus an execute-only reference; a fixed subset of 50 tasks, stratified by language, difficulty, test-suite size, and runtime.

Nine programming tasks and six proof problems were used during development (the six proofs are included in the full evaluation), so the main figures report held-out-from-development tasks for programs only.

**Models.** Proof runs use GitSwarm with Gemini-3.1-Pro and GPT-5.5-high workers. ProgramBench runs use Codex (GPT-5.5-high) workers and GPT-5.5 workers at default reasoning effort. GitSwarm worker concurrency is 5 for problem-solving tasks.

**The "Forced" baseline.** Each baseline is the corresponding single agent (bash, mini-SWE-agent, or Codex for proofs; repeatedly-continued Codex (GPT-5.5-high) for programs) that is *prompted to continue after it stops on its own*, and evaluated at its intermediate natural-stopping checkpoints. The name is the paper's: the single agent is being forced to keep spending compute.

**Compute accounting.** The budget B counts *charged worker episodes*; voluntary abstentions are replaced without consuming budget (though they still cost inference). Programming budgets: 100/200/300/500 episodes; proofs: 20/40/80/120/160. Token counts are per model request, summed over all requests by a worker/agent: input tokens include the full prompt with all previously accumulated context, output tokens include newly generated tokens (hidden reasoning included). Reported values are per-problem totals averaged across problems.

## b. Headline results

**ProgramBench (50 shared tasks):** GitSwarm with Codex workers improves from 73.1% at 100 workers to **79.4%** at 500; with GPT-5.5 workers from 64.6% (100) to 71.2% (300). The forced-continued Codex baseline stays between **63.1% and 65.1%** across all checkpoints.

**IMOProofBench-Advanced:** With Gemini-3.1-Pro, mean fully-correct proofs rise from 23.3 (20 workers) to 29.0 (160), though the curve is not monotonic (a local dip at budget 80). The forced mini-SWE-agent baseline tops out at 23; the continued bash agent reaches 26.5 at similar input-token budget. With GPT-5.5-high workers, GitSwarm solves 28 of 30 at a budget of 20 workers and **all 30** at budgets of 40 and 80.

**Honest caveat (theirs):** Best-of-N baselines with an oracle verifier also reach the benchmark ceiling at *lower* token consumption, so saturating this benchmark does not by itself establish a unique advantage for GitSwarm.

## c. Inference-time scaling behavior

- Performance rises with worker budget over the tested ranges, but **gains diminish at larger budgets** (ProgramBench).
- **Input-token costs grow as workers inspect an expanding repository** — the shared history itself is a cost driver: every new episode's context includes more repo to look at.
- Appendix D adds: proof construction follows the same broad trend but with more run-to-run variation and **earlier saturation**; program reconstruction keeps benefiting from larger budgets. Also notes the proof cross-budget curves are descriptive rather than a controlled worker-count estimate, because configurations differ across runs.

## d. What the Forced baseline comparison isolates

The Forced baseline is the "more compute for one agent" counterfactual: keep prompting the same single agent after it would naturally stop, and evaluate along the way. GitSwarm beating it at comparable checkpoints therefore isolates the **swarm structure** — parallel workers, branching, a shared repo where discoveries and failures accumulate and get reused — from **raw inference compute**. The paper's scaling curves are explicitly plotted against both worker budget and actual input/output tokens consumed, so the comparison is about organization of compute, not amount of compute.

## Relevance to locust

- **More workers help, with diminishing returns** — swarm size is a real lever, but not a linear one. A locust deployment should expect strong gains going from 1 to a handful of agents, then flattening.
- **Inspection cost grows with history size.** This is the single most actionable finding for locust: the shared memory that makes compounding possible is also the thing that makes each episode's context bigger. A vault that grows unbounded will tax every agent's input budget. Design implications: keep memory indexed and summarized so inspection stays sublinear (links, per-channel namespaces, rollup notes), budget memory size as an operational cost, and prefer pruning/compression strategies over unbounded accumulation.
- **Structure beats brute compute.** The Forced baseline shows that just running one agent longer doesn't catch a well-organized swarm. For locust, the investment should go into the *coordination substrate* (the vault, the channels, the conventions for reuse and recovery of failed ideas) rather than raw token spend per agent.
