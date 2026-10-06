# Distillation 8: Related work positioning and the worker-prompt architecture

**Source:** "GitSwarm: Decentralized Compounding Inference" (arXiv 2610.04862v1)
— Appendix A (Related Work, lines 2262–2978) and Appendix H (GitSwarm Worker Prompts, lines 6030–6808).
URL: https://arxiv.org/html/2610.04862v1

---

## Part A — Positioning vs prior work (Appendix A)

The appendix frames the paper's contribution at the intersection of inference-time
search, agent memory, multi-agent coordination, and automated research. The running
theme: how systems preserve intermediate work, support competing lines of
exploration, record dependencies, and decide what to investigate next.

### Inference-time search

- **Tree of Thoughts / LATS** (Yao et al. 2023; Zhou et al. 2024): search preserves
  intermediate states *within one search*. GitSwarm instead preserves partial
  solutions, executable artifacts, and negative results *across independently
  scheduled worker episodes* — persistence outlives any single run.
- **PDR / RSA** (Parallel-Distill-Refine, Madaan et al. 2025; Recursive
  Self-Aggregation, Venkatraman et al. 2025): parallel drafts get distilled into a
  bounded textual workspace or sub-sampled context, then aggregated/refined across
  rounds. GitSwarm shares the "build on earlier computation" goal but uses a
  different memory model: rather than repeatedly compressing drafts into a single
  workspace, it preserves individual contributions in a shared, branchable
  repository; each commit explicitly records dependencies across branches. It also
  operates purely at inference time (PDR additionally trains models on its
  operator).
- **Kim et al. (2026)** (long-horizon coding agents): summarize each agent
  trajectory into structured records (hypotheses, progress, failures) for
  tournament voting and PDR-style refinement. GitSwarm differs in *how* work is
  preserved and organized: full individual contributions and artifacts in a
  branchable repo, with workers independently choosing what to extend, vs.
  distilled compact contexts for selection.

### Agent memory

- **Reflexion** (verbal feedback for subsequent trials), **ExpeL** (extracted
  insights from prior tasks), **Voyager** (accumulated reusable skills),
  **MemGPT** (long-term context management), **A-MEM** (linked, updated memories).
  The paper grants these establish the value of external memory; its focus is a
  *shared, versioned contribution history* in which independent workers pursue
  *competing* approaches and explicitly declare dependencies *across* them — the
  memory is collective, branched, and dependency-typed, not per-agent.

### Multi-agent systems

- **Multi-agent debate** (Du et al. 2023), **Mixture-of-Agents** (Wang et al.
  2025): interaction across model instances/layers to improve answers.
- **AutoGen** (configurable conversation patterns), **MetaGPT** (specialized
  roles in a prescribed software workflow), **AgentVerse** (configurable teams and
  collaboration patterns), **Magentic-One** (orchestrator assigns work and revises
  plans), **GPTSwarm** (agent operations and communication as an optimizable
  computation graph — its graph specifies how a multi-agent program *executes*,
  while GitSwarm's graph records how published contributions *accumulate* over
  time).
- GitSwarm's contrast: no assigned roles, no optimized communication topology.
  *Homogeneous* workers independently choose whether to explore, repair, verify,
  integrate, or nominate existing work; the harness only centrally manages
  execution and applies deterministic plurality readout.

### Version-control agent systems (closest relatives)

- **AgentGit** (Li et al. 2025): version-control ops (commit, branch, rollback)
  in agent workflows for recovery and alternative paths.
- **CAID** (Geng and Neubig, 2026): parallel Git worktrees and branch-and-merge
  for async software engineering — but with a central manager that plans and
  delegates subtasks.
- **SwarmResearch** (Virk et al., 2026): coding agents on separate Git branches
  directed by a Shepherd Agent that steers search and adjusts parallelism.
- GitSwarm's contrast: the repository is not a manager's coordination tool but a
  persistent substrate for *self-directed* contributions. Each accepted commit
  has a physical Git parent *and* must declare semantic dependencies via
  `informed_by=`, including cross-branch references. Nomination records stay
  separate from the contribution graph, so evaluating a candidate never alters its
  ancestry.

### The concurrent work: Agora (Zhang et al., 2026)

Agora is the nearest neighbor: developed in parallel, also studying compounding
research by self-directed agents over a persistent Git-backed contribution
graph, with no assigned tasks or central planner, Git-parent dependency
retention (including multiple parents), and searchable frontier views plus
diversity-aware recommendations for navigation.

How GitSwarm claims to differ, in three points:

1. **Parent vs. provenance separation.** GitSwarm splits each commit's physical
   Git parent from mandatory semantic `informed_by=` references — a worker can
   physically build on one branch while explicitly citing discoveries from
   others. Agora represents dependencies through Git parent relationships only.
2. **Selection protocol.** GitSwarm selects final candidates through worker
   nominations and deterministic plurality readout; Agora relies on frontier
   views and diversity-aware recommendations.
3. **Evaluation emphasis.** GitSwarm measures declared cross-branch
   dependencies, trace-visible artifact consumption, and selected-solution
   ancestry (plus controlled substrate tests and GPU-backed research); Agora's
   extended study emphasizes research diversity and independent verification.
   Presented as complementary evidence about repository-mediated collective
   research, not a performance comparison.

### Blackboard / global-workspace lineage

- **HEARSAY-II** (Erman et al. 1980; Nii 1986): independent knowledge sources
  opportunistically read from and contribute to a common workspace.
- **Global Workspace Theory** (Baars 1988), Shanahan's competition-and-broadcast
  realizations (2006; 2008), and Goyal et al. (2022) on learned specialist
  modules communicating through a capacity-limited shared workspace.
- GitSwarm inherits the shared-workspace principle but changes its temporal and
  structural role: not a transient common state for coordinating computation
  within a cognitive cycle, but a *durable, branchable history of contributions
  across agent episodes*, with explicit dependencies, where competing lines
  persist and can recombine. The focus is accumulation of reusable computation
  over long horizons, not just coordination.

### Autonomous experimentation / evolutionary search

- **Karpathy's AutoResearch** (single-agent loop of modify/train/evaluate),
  **AI Scientist** (Lu et al. 2024; idea→paper), **AI Scientist-v2** (progressive
  agentic tree search under an experiment manager), **Agent Laboratory** (staged
  research workflow), **AlphaEvolve** (evolutionary selection over an evaluated
  program population), **AIRA2** (agentic experimentation + consistent evaluation
  + async multi-GPU).
- The paper's question against these: how independent research episodes can
  additionally preserve hypotheses, failed experiments, code, and evidence in an
  *auditable contribution graph* without a central agent picking each next
  experiment.
- Two finer contrasts:
  - **ARTS** (Juneja et al. 2026): a central reasoning model inspects earlier
    experiment logs and picks the next search-tree node, condensing history into
    short insights or test-time-trained weights. GitSwarm has *no central
    scientist*: workers with fresh contexts retrieve from an unpruned,
    never-summarized repo, can merge separate branches into one contribution,
    and select via decentralized nomination.
  - **Meta-Harness** (Lee et al. 2026): like GitSwarm, exposes full uncompressed
    history for selective retrieval — but proceeds sequentially, with every
    stored candidate a complete harness in a fixed directory. GitSwarm allows
    bounded *partial* contributions that later workers extend, giving a higher
    branching factor and more exploration.
- Benchmarks for autonomous research are noted as motivation for evaluation
  beyond fixed-answer coding tasks (MLAgentBench, MLGym, ResearchGym,
  AIRS-Bench); GitSwarm instead examines how independently scheduled workers
  accumulate code and experimental evidence through shared persistent memory.

### The two comparison tables

**Table 1 — system mechanisms** (how related systems organize intermediate
work): four columns — *retained state*, *alternative paths*, *recorded
dependencies*, *next-work control*. It is explicitly a comparison of system
designs, not a performance ranking. Rows cover ToT/LATS (search states,
reasoning tree, search edges, search policy), Reflexion/ExpeL (reflections and
experiences, successive attempts, episode/insight memory, individual agent),
MetaGPT/AutoGen (messages and task artifacts, configured interactions, workflow
and messages, roles or configured dialogue), GPTSwarm (executable agent graph,
graph variants, operation-level edges, graph optimizer), AgentGit (versioned
workflow states, rollback and branches, state lineage, application workflow),
CAID (Git worktrees and commits, parallel branches and merges, code-integration
lineage, central delegator), SwarmResearch (Git branch states, parallel search
agents, branch-level history, shepherd agent), AI Scientist-v2 (research
experiments, progressive search tree, search-and-experiment lineage, experiment
manager), AlphaEvolve (candidate database, evolutionary population, candidate
variations and scores, evolutionary selection), AIRA2 (experiment records,
async GPU workers, experimental feedback, research agents and harness), Agora
(append-only Git contributions, shared research DAG, Git-parent dependencies,
independent workers and frontier suggestions), and GitSwarm itself (atomic Git
contributions, persistent Git branches, Git parent + required cross-branch
`informed_by=`, independent workers and plurality readout).

**Table 2 — seven design properties**: shared memory persisting across worker
episodes; durable branches for competing approaches; self-directed work
selection (no central task planner); absence of predefined specialist roles;
asynchronous workers; explicit dependencies across branches; and dependency
records that distinguish the physical starting state from other sources used
by a contribution ("separate sources"). Each cell is marked full (✓), partial/
configuration-dependent (∼), or absent (–), judged on the protocols described
in the cited papers — not every configuration the frameworks could support.
MetaGPT scores – across the board; AutoGen and AgentGit are mostly ∼; CAID and
SwarmResearch have shared memory, durable branches, and async workers but not
self-direction (they keep a central manager/shepherd), with CAID ∼ on
cross-branch deps; only GitSwarm claims ✓ on all seven — in particular the last
two (cross-branch deps, separate physical vs. semantic sources), which are the
differentiators.

---

## Part B — Appendix H: the worker prompt architecture

### H.1 Prompt assembly (how prompts are composed)

The appendix documents the exact prompts used across four setups: ProgramBench
with Codex (GPT-5.5-high) workers, ProgramBench with GPT-5.5 workers,
IMOProofBench-Advanced with Gemini-3.1-Pro workers, and the RMT and Looped
Transformer research runs. The assembly rule is compositional:

- Every worker's **system prompt** = global component (H.2) + task addendum
  (H.3), concatenated with a blank line between them. Gemini workers add a
  sequential shell-interaction extension (H.4) at the end.
- The **user prompt** (H.5) is task-specific and separate.
- Codex workers additionally get a **transport override after the user prompt**;
  system prompt + user prompt + override are passed to the Codex CLI together as
  a single input, on top of Codex's built-in instructions.
- Table 14 enumerates which components each setup receives. Research-task runs
  use a modified global component: the nomination action and all
  nomination-related text are removed (selection happens post-run by validation
  score instead of worker nominations); the opening selection description and
  the dashboard description are rewritten accordingly; one sentence drops the
  `gs_inspect` command; and a short autoresearch selection policy — end with a
  commit or an abstention, never nominate — is inserted before the task addendum.

### H.2 The global system prompt (the worker's contract)

In my own words, the contract's structure and rules:

**Identity and medium.** The worker is told it is an autonomous participant in
an asynchronous search, collaborating with other workers through a shared Git
repository. It works in its own Git worktree of a persistent repo shared by all
workers; the repository *is* a graph of candidate solutions, and each
graph-expansion commit adds one candidate node.

**The central conceptual distinction: parent vs. provenance.** A candidate's
Git parent identifies the repository state it was created from (physical
ancestry). The `informed_by=` provenance field identifies the earlier commits
the worker directly relied on (semantic influence). These are explicitly
different relationships — the physical starting state and the intellectual
sources are recorded separately. This is the same separation that distinguishes
GitSwarm from Agora in the related-work section.

**The swarm as discovery process.** A candidate node need not be a final answer;
it may be a stepping stone that later workers inspect, test, criticize, repair,
extend, combine, or replace — but every node must still stand alone as a
candidate answer (preserving whatever task-specific structure and artifacts the
task requires).

**Selection mechanics.** There is no mutable "current best" and no exclusive
selector. Any worker may nominate one existing candidate by creating a
rationale-bearing nomination commit; each valid nomination is exactly one vote.
At run end, the most-voted candidate wins, with ties broken deterministically
by lexicographic full commit SHA. Zero valid nominations means falling back to
the repository root.

**Execution interface.** Workers act through a structured bash execution tool
in their worktree (Codex workers used a guarded MCP tool inside a task
container for ProgramBench and Codex's native shell for research tasks; Gemini
workers were not given a tool definition — the harness executed fenced bash
blocks from their responses; GPT-5.5 ProgramBench workers used a mix of fenced
blocks and a structured tool depending on budget). Every response must call the
tool; prose alone is not executed.

**The workflow — Review → Choose action → Commit:**

1. **Review (inspect completed and in-flight work).** Start with `git log
   --graph --oneline --all`. The prompt enumerates inspection commands (status,
   branch, show, diff, grep, ls-tree) plus custom coordination tools:
   `gs_explore`, `gs_inspect <sha>`, `gs_inflight` (concise activity logs and
   visible working-tree state of in-flight workers), `gs_inflight inspect`,
   `gs_inflight read`, and `gs_dashboard` for a current coordination snapshot
   (total/remaining worker budget, in-flight counts, nomination tallies — the
   dashboard does *not* summarize the repo for you). Rules: don't treat commit
   subjects as substitutes for reading the work; peer worktrees are strictly
   read-only; in-flight content is unfinished coordination evidence, not durable
   provenance — never copy or depend on it unless it later appears in a
   committed node that you cite through `informed_by=`. Completed nominations
   are selection state, not behavioral evidence of correctness — workers are
   warned not to herd-copy the apparent majority, and a visible nominee is not
   automatically a serious contender.

2. **Choose one of three actions:**
   - **Action 1 — Make a contribution.** Scope one bounded contribution
     privately (gap, concrete deliverable, target commits, observable success
     criterion, abort condition; the success criterion applies to the scoped
     contribution, not the whole query), then *publish* the scope advisory via
     `gs_inflight announce --stage scoping --summary ... --target <sha>` —
     advisory, not a request; no scheduler grants or reserves work. Fork once
     with `gs_fork <starting_commit_sha> <short-task-name>` from the most
     convenient starting tree (sole Git parent = physical starting state;
     every target commit must still appear in `informed_by=`). Materialize the
     work; write **MY_THOUGHTS.md** (intended contribution, semantic sources
     and how they were used, approach reasoning, scope-validity/abstention
     checks, work performed and evidence, unresolved uncertainty, TODOs/next
     steps); validate; then write **EVALUATION.md** (candid, evidence-based
     self-assessment: what it accomplishes, what was actually checked, known
     gaps/risks, most useful next work). A negative result may be committed
     only if it directly satisfies the scoped deliverable. If the work becomes
     redundant or unworthwhile mid-flight, abstain immediately.
   - **Action 2 — Nominate the best existing solution.** No new solution,
     repair, or synthesis — just compare existing candidate nodes and pick the
     current best. Publish that a critical graph comparison is underway
     without disclosing a tentative preference (nomination judgments stay
     independent until ballot commits publish). Use ancestry, diffs, existing
     evaluations, and evidence; spend deep effort only on serious contenders;
     don't treat pass counts from different candidate-authored test suites as
     comparable, and don't infer quality from the live tally, recency, branch
     length, or popularity. Branch from the chosen candidate's exact SHA,
     declare the nomination with `gs_nominate_best_solution`, then document in
     **BEST_SOLUTION.md**: the nominated SHA, the candidates and evidence
     compared, the evidence-based case for it, known limitations, and any
     prior nomination records that informed the comparison. The
     nomination-record branch must otherwise preserve the candidate's tree
     unchanged.
   - **Action 3 — Abstain.** When no work would add meaningful information or
     progress — e.g., everything feasible would duplicate completed or
     in-flight work. State the substantive reason and run `gs_abstain
     --rationale`. Abstention does not consume a worker-budget slot (the
     scheduler may launch a replacement worker); no empty commits.

3. **Artifact authorship.** MY_THOUGHTS.md and EVALUATION.md are worker-specific
   with one authorship entry. Shared human-readable artifacts (BEST_SOLUTION.md,
   experiment reports, continuation notes) carry an update history at the top —
   prepend `Updated by: Worker <WORKER_ID> | Scoped commits: <sha1>,...`
   without deleting earlier entries. The header stays out of source code and
   generated files (authorship there comes from Git history and commit
   provenance).

4. **Commit, publish any nomination, and exit.** Exactly one commit. Format:
   `[<hyphenated descriptive type>|informed_by=<sha1>,<sha2>,...] <summary>`.
   Provenance is mandatory — every target commit in the scope must be listed,
   even a single source; abbreviated SHAs are fine if unique. Nominations use
   the fixed type `best-solution-nomination`. The scheduler validates the
   pending declaration, exact nominated parent, provenance, unchanged candidate
   tree (except BEST_SOLUTION.md), and rationale, then publishes one immutable
   ballot; the nomination commit is metadata and cannot itself receive votes.
   After the commit is detected, the harness ends the worker episode.

### H.3 Task addenda (per-benchmark specializations)

Three addenda, appended to the global component:

- **ProgramBench addendum.** Goal: create an original codebase whose compiled
  executable reproduces the reference program's behavior, inferred from bundled
  docs and by running an execute-only reference. Candidate-node invariant:
  every graph-expansion commit's tree must contain a self-contained,
  executable `compile.sh` that produces `./executable` in the repo root —
  including exploratory or stepping-stone nodes. Constraints: no copying,
  wrapping, executing, linking, or runtime dependency on the reference program;
  never inspect the reference binary's bytes (run it and observe CLI/stdout/
  stderr/exit/timing); generated executables must not be committed; worker
  branches inherit the parent's tracked tree (build artifacts are untracked and
  not inherited); build each compared candidate in a separate private
  directory/worktree to avoid contamination; validation via differential
  testing (same inputs, compare stdout/stderr/exit/timing/formatting); edge
  cases (invalid flags, malformed/empty/large/Unicode input, boundary cases);
  and claims in collaboration artifacts must distinguish observed evidence
  from inference.
- **IMOProofBench addendum (task contract).** An IMO-style problem; the final
  scored artifact is `SOLUTION.md` from the selected candidate — a claimed
  complete answer must put the complete proof there; other files are supporting
  work only. An LLM judge grades on the 0–7 IMO scale (7 = complete rigorous
  proof, 6 = sound core with only minor gaps, 1 = substantial partial progress,
  0 = incorrect; a wrong answer presented as correct scores 0).
- **RMT addendum** (the Loop addendum is identical with "RMT" swapped to
  "Loop"). An empirical architecture-research task: read TASK.md,
  research_task.md, cluster_management.md before allocating compute; preserve
  the complete RMT workbench in every candidate; use protected development
  evaluations only for serious candidates; record actual evidence, never invent
  metrics; a tested negative result or improved experiment config counts as a
  useful candidate; durably record limitations and unresolved uncertainty so
  future workers interpret the evidence correctly; each candidate should carry
  a validatable `submission/final_model` bundle when ready for comparison.
  Research-discipline rules: use idle queue-wait time for other useful work
  rather than idling; build on prior workers' analysis/audits/tooling/
  validation/queueing by running the actionable experiments they enable;
  favor new empirical evidence over another preparatory layer; watch for search
  collapse (workers clustering around one mechanism) and reopen the search
  with a materially different, well-motivated hypothesis; novelty alone is not
  enough; cancel work when separated observations show it is stuck/invalid;
  every submitted job must be terminal before the worker exits.

### H.4 Model- and transport-specific additions

Two documented adaptations:

- **Sequential shell interaction (Gemini-3.1-Pro).** Pilot runs showed Gemini
  workers emitting an entire contribution workflow in one response, sometimes
  predicting command outputs and branching from guessed commit SHAs; when
  `gs_fork` failed, the remaining commands still ran and committed on the
  placeholder branch, producing invalid results. The extension forces
  incremental interaction: one evidence-dependent step per response, wait for
  output before the next step; `gs_fork` must be a standalone step (exactly one
  bash block, no other commands in the response); before branch confirmation,
  only read-only inspection and coordination are allowed — no file creation,
  modification, deletion, or running interpreters/scripts/tests/builds/
  experiments; never mix the contribution and nomination workflows. The harness
  enforces this too: it runs blocks with `set -e`, stops the remainder of a
  response after the first failed command, and rejects pre-branch commands
  that aren't read-only inspection, coordination, standalone `gs_fork`, or
  standalone `gs_abstain`. Graph, scope, dependency, and nomination rules are
  unchanged — only the command sequencing is constrained.
- **Codex transport override (ProgramBench).** Remaps the execution interface:
  Codex workers' only interface is a guarded MCP tool (`pb.container_bash`)
  running in the worker's private Git worktree inside a network-disabled
  container. Any instruction about returning fenced bash blocks is superseded
  — call the MCP tool instead. All GitSwarm policies are preserved exactly;
  finish one bounded contribution or abstention, then stop.

### H.5 User prompts

Two templates shown (RMT's task-description block is noted but not listed):

- **ProgramBench user prompt.** States the collective goal (reconstruct a
  codebase reproducing the reference binary's behavior from docs + the binary
  alone), locates the execute-only reference (`./executable_orig`), notes no
  pre-installed dependencies and no internet, disqualifies finding original
  source / installing the tool / wrapping the binary / runtime dependence on it,
  restates the candidate invariant (every graph-expansion node must have a
  committed `compile.sh` producing `./executable`, even exploratory nodes),
  clarifies that read/copy failures on the reference are expected (observe
  behavior instead), and frames differential checks as investigative tools, not
  deliverables. Then: **Git-Swarm worker context** (worker ID, worktree path,
  initial branch at the repository root SHA) and a **swarm status at spawn**
  (total/remaining worker budget, in-flight worker count, valid nominations
  cast, distinct candidates nominated, current nomination tally by candidate
  SHA) — explicitly labeled a spawn-time snapshot, with the instruction to run
  `gs_dashboard` for a current one, and the warning that nomination counts are
  coordination state, not evidence of candidate quality: visible nominees are
  not a shortlist, and zero-nomination candidates remain fully eligible.
- **IMOProofBench user-prompt template.** The problem statement slot
  (`<PROBLEM_STATEMENT>`), then: write a complete rigorous proof, graded 0–7
  with 7 flawless IMO standard; every claim explicitly justified, no
  hand-waving; each step follows from the previous or a cited theorem; all
  cases handled; clear organization (key observations, main argument,
  conclusion).

---

## Relevance to locust

Appendix H is the closest thing in the paper to an implementation spec for a
swarm skill, and its patterns map directly onto locust's needs:

1. **Skill as composed prompt layers.** GitSwarm assembles each worker's
   operating contract from a *global component* (invariant protocol) + a *task
   addendum* (per-lane specialization) + a *transport override* (per-model
   execution interface). Locust's skill can mirror this: a global skill layer
   (vault layout, channel conventions, secret-filter rule, commit/commit-message
   conventions) + per-channel addenda (memory channel vs. command channel rules)
   + per-transport notes (how a given agent executes writes). New channels
   become new addenda, not core rewrites — the plugin architecture in prompt
   form.
2. **`informed_by=` as the provenance convention.** Locust's memory channel
   needs exactly this: the physical commit (git parent) tells you *where* work
   happened; the semantic `informed_by=` list tells you *what it built on*.
   Adopting that mandatory-provenance commit-message convention gives the vault
   a queryable cross-agent dependency graph for free — and GitSwarm's finding
   that workers genuinely do cite cross-branch work suggests agents will
   actually use it.
3. **Required collaboration artifacts.** MY_THOUGHTS.md (reasoning trace), 
   EVALUATION.md (candid self-assessment), BEST_SOLUTION.md (comparative
   rationale with authorship history prepended) are a template for locust's
   artifact conventions: worker-scoped files for trace, assessment files for
   judgment, and append-only-history headers on shared artifacts — all
   human-readable in Obsidian and machine-citable by agents.
4. **Review → choose action → commit lifecycle.** The review protocol (read
   completed *and* in-flight work, don't treat subjects as substitutes for
   reading, treat vote tallies as coordination state not evidence) is directly
   reusable as locust's contribution discipline; the three actions
   (contribute / nominate-or-equivalent / abstain) and the "abstain rather
   than manufacture redundant work" rule answer locust's open conflict-resolution
   and channel-taxonomy questions with a working default.
5. **Nomination/consensus separated from the contribution graph.** GitSwarm
   keeps ballots as metadata commits that cannot themselves be voted on —
   evaluation never mutates ancestry. For locust, that is the template for the
   command channel: operational traffic (votes, tasking, status) lives as
   namespaced records that never alter memory-channel history.
6. **Cautionary pattern — the Gemini transport lesson.** A model that emitted
   whole workflows in one response and predicted command outputs had to be
   constrained to incremental, output-dependent steps with harness enforcement.
   Locust's skill should assume heterogeneous agents and enforce sequencing at
   the tooling layer (not just the prompt): read-before-write, wait-for-output,
   and fail-closed filtering — which is also where the secret filter belongs.
