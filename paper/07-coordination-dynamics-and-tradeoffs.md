# Coordination dynamics and design tradeoffs

**Source:** GitSwarm: Decentralized Compounding Inference — https://arxiv.org/html/2610.04862v1
**Covers:** §3.5 (Adaptive Coordination and Design Tradeoffs), Appendix C (controlled substrate tests), Appendix G (worker activity, coordination, tradeoffs). Distillation in own words; see the paper for exact figures and tables.

## 1. Emergent division of labor, no assigned roles

GitSwarm workers are never assigned solver/verifier/integrator roles — not by the scheduler and not by the prompt. Yet when researchers label each worker episode after the fact (bottom-up taxonomies, built separately per benchmark), a clear division of labor appears, and it adapts to the task:

- **ProgramBench (program reconstruction):** a short bootstrap wave creates initial candidates, then workers keep doing capability expansion, fidelity repair, and candidate synthesis until termination. Nomination (selection) rises only near the end — eating into, but never fully replacing, implementation work. Empirical validation stays rare, appearing mostly at larger budgets. The winning candidate is produced very late: median first-appearance at 93.1–98.0% of run progress. The winner is usually a final repair or integration that inherits older work.
- **IMOProofBench-Advanced (mathematical proofs):** the opposite regime. Workers shift early and strongly toward selection among existing proofs. Selection occupies 64.8–82.2% of episodes in the final fifth of a run; the selected proof appears at a median of just 8.6–17.5% of run progress; the share of selection episodes rises from ~44% to ~70% as the budget grows from 20 to 160 workers. Later workers mostly compare and ratify an early candidate.

Takeaway: one decentralized protocol adapts its workflow to the problem type — program reconstruction keeps accumulating implementation changes, while proof runs increasingly use the repository to evaluate work that already exists. About three quarters of ProgramBench episodes end in a contribution commit at every budget; only ~3.1% of charged episodes produce nothing accepted.

## 2. Redundancy avoidance via voluntary abstention

Workers can explicitly abstain when their intended contribution is already covered by published or in-flight work. Abstainers are replaced without consuming the charged budget, but they still spend tokens on the inspection that led them to abstain.

- **Rate:** roughly one in six ProgramBench workers abstains as redundant; on proofs it's under 2.7% (0.65–2.62% across budgets).
- **Why the difference:** ProgramBench abstainers mostly cite overlapping work that is *still in flight* — collisions from asynchronous overlap. In 97% of matched cases, the conflicting commit landed *after* the abstaining worker's episode began; these workers weren't idle, they had edited, built, tested, and probed before checking the graph late and stopping. Proof abstainers instead cite a *completed, already-published* proof and stop quickly (median five to seven tool turns).
- **Calibration:** a score-blind review of ~1,400 ProgramBench abstention traces found 94.9% of them justified — the cited work materialized and covered the intended contribution. Abstention usually prevents a duplicate commit.
- **Cost:** ProgramBench abstainers consume about 6–7% of total input and output tokens across budgets; proof abstainers much less (1–3% of input, under 1% of output).
- **Limit:** abstention avoids duplicate *publication* but cannot recover the computation already spent. The main failure risk is premature abstention citing a path that later fails and is never restored.

## 3. Selective inspection of the growing history

Workers do not reread the whole graph. Inspection stays bounded while history grows:

- ProgramBench workers open about five to seven distinct candidate commits each; proof workers about three — increasingly small fractions of the available history. On B100 runs, workers 90–99 have ~72 candidates available but open only ~6 each. Their absolute look-back distance grows (to ~150 commits at B500), but the *fraction* of spawn-time history they reach keeps shrinking.
- Workers use summaries and declared dependencies to focus on one integrated candidate plus a few relevant alternatives. The proof workers' reach falls from ~82% of history at small budgets to ~54% at the largest.
- Broader comparison concentrates near startup (orientation) and, on ProgramBench, during the final selection pulse.

The repository functions as a *searchable memory*, not a transcript every worker rereads. The paper cautions that tested budgets don't prove this stays effective at much larger graph sizes.

## 4. Voting dynamics and selection timing

Nominations are autonomous, not a fixed final phase. Timing differs by regime:

- **ProgramBench:** votes arrive in a terminal pulse. The first vote moves from ~68% of normalized progress at budget 100 to ~16% at budget 500, but the total run grows proportionally — forming the first submittable candidate takes a roughly fixed amount of absolute work. The winner appears only 8–11.5 worker episodes before termination.
- **Proofs:** votes accumulate steadily and almost linearly after startup. First nomination moves from ~32.5% of progress at B20 to ~4% at B160; mean nominations per problem rise from 9.1 to 115.6.
- **Caveat:** a nomination is a worker preference, not an independent verification. Repeated nominations for one commit hash are coordination signals, not independent proofs of correctness. Also, because votes attach to exact commit hashes, each late repair creates a new target and splits support — yet a strong late integration can still win after a small, rapid voting pulse.

## 5. Appendix C: controlled tests isolating each substrate property

Three synthetic tasks, each isolating one substrate property with Gemini-3.1-Pro workers across three seeds:

- **Persistence — Hidden Rulebook:** a hidden rulebook of facts and unary rules forms dependency chains. Shared persistent memory (each worker reads all previous workers' completed observations) holds high relevant-record coverage as targets or chain depth grow. Two independent baselines with ~33% *more* total queries flatline well below shared memory (~70–75% and ~20–23% coverage) and drop with depth. Persistence cuts the rediscovery cost of extending sequential investigations.
- **Branchability — Hidden Maze:** workers explore a hidden tree looking for a treasure. Branch-native memory (checkpoints organized as a tree of resumable endpoints, prompt explicitly asks workers to inspect and restore one) substantially reduces the repeated-move fraction versus flat persistent memory (which stores all prior moves but forces every worker to retrace from the entrance). Caveat: the treatment also changed the prompt to make restoration explicit, so it tests a practical memory interface, not branchability alone.
- **Attribution — Retracted Evidence:** an archive of source notes, derived notes, and conclusions, with one intermediate artifact from a retracted lab invalidating all downstream conclusions. Explicit attribution metadata (producer + parent IDs on each row) reduces the fraction of the archive workers must read before the first exact submission, at every lineage depth, ambiguity, and fan-out level — the gap widening with depth. Recorded dependencies let workers follow and invalidate affected lineages instead of reconstructing every link from content.

**Scope caveat (C.4):** these are controlled illustrations of *why* the properties may help; they do not establish that any single property caused GitSwarm's benchmark improvements. That would need interventions inside the full system holding compute, worker behavior, and other capabilities constant.

## 6. Architectural tradeoff experiments (G.6)

On nine ProgramBench development tasks (GPT-5.5 workers, budget 50, five concurrent, three seeds — small dev-set experiments, not statistical proof):

- **Async vs synchronous execution:** async (launch a replacement when a slot frees) takes 38.0 min vs 49.3 min for synchronized five-worker waves — sync costs ~30% more wall-clock, consistent with straggler latency. Scores: 62.06±2.44% async vs 63.11±0.58% sync — a small observed gap of +1.05 pp for sync, with no claim of equivalence.
- **Readout:** decentralized nomination 62.06±2.44% in 38.0 min; a privileged *central selector* (one final worker picks an existing candidate) scores 64.40±1.49% in 34.9 min (+2.34 pp, faster); a *central aggregator* (one final worker synthesizes a new candidate) scores 58.19±4.53% in 43.9 min (−3.87 pp, slower). Forcing a final synthesis step was neither more accurate nor more efficient than decentralized nomination.

The paper's posture: async execution and decentralized readout are kept as the default because the tradeoffs are small and they buy latency/robustness — the dev experiments characterize the tradeoffs rather than proving equivalence.

## 7. Related: causal-attribution and cost limits (from §4)

Worth noting for later use: the mandatory `informed_by=` field records *worker-declared* dependencies, and observed artifact consumption evidences *use*, not causal influence. Useful computation need not appear in the winner's ancestry (e.g., the Looped Transformer run recovered an initially rejected idea; negative results informed later work). And persistent collaboration has overhead that grows with the graph: workers spend input tokens inspecting previous contributions, and redundant/unsuccessful episodes spend compute without producing accepted work.

---

## Relevance to locust

Operational lessons from the GitSwarm evidence, mapped onto the locust design (git-backed Obsidian vault as shared memory, skill as onboarding, channels as namespaces):

- **Selective inspection is the answer to history growth.** The swarm never rereads the whole graph; it reads summaries and declared dependency links, then opens a bounded handful of relevant items. For locus-vault, this means: invest in good per-page summaries, backlinks/metadata, and a convention for recording "what I built on" (GitSwarm's `informed_by=` is a template). A growing vault doesn't break the system as long as navigation is selective rather than exhaustive.
- **Abstention is the anti-redundancy mechanism.** One in six construction workers voluntarily stands down when equivalent work is already in flight or committed — and 94.9% of those calls were correct. A locust command channel should include an explicit "checking before claiming work" norm (GitSwarm prompts workers to check for overlap before choosing scope) plus a lightweight "I abstain, here's why" signal. It prevents duplicate commits even if it can't save the inspection tokens already spent.
- **No fixed roles; coordination through shared state.** Division of labor (bootstrap → expand/repair/synthesize → nominate; or construction → early selection on proofs) emerged from workers reading the same graph, not from role assignments. Locust can stay simple: hand every agent the same skill, let the vault's state be the coordinator. Emergent specialization beats assigned lanes.
- **Async + decentralized readout is worth small tradeoffs.** Async execution cuts wall-clock ~23% for ~1 pp of score; decentralized nomination loses ~2.3 pp to a central selector but avoids a privileged final actor. For locust: no central scheduler, no central arbiter for what lands in the vault — decentralized nomination plus audit trails keeps the architecture simple and robust, at a tolerable quality cost.
- **Attribution metadata pays for itself.** Explicit producer/parent links cut the search cost of lineage reconstruction, and the gap widens as the vault deepens. Locust's secret filter and write path should record authorship and parentage at write time; reconstructing provenance later is the expensive route.
- **Branch-native structure for parallel lines.** The Hidden Maze result favors branchable history (branches, not one flat log) so parallel explorations don't pay the cost of each other's prefixes. In the vault: per-project branches or clearly namespaced pages rather than a single linear dump.
