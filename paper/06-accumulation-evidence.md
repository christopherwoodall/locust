# Accumulation Evidence — how to measure whether shared memory is actually reused

**Source:** GitSwarm: Decentralized Compounding Inference, arXiv 2610.04862v1 (https://arxiv.org/html/2610.04862v1), Section 3.4 ("Evidence for Compounding Computation") and Appendix F.

---

## What is being measured

GitSwarm workers accumulate work in a shared, branchable git repository. The paper asks: does later work actually build on earlier work, or do agents just work in parallel and dump results side by side? It measures reuse three independent ways, because no single metric tells the whole story:

1. **Declared dependencies** — what workers *say* they built on (self-reported links).
2. **Trace-visible artifact consumption** — what workers *demonstrably* read/ran (measured from execution traces, not claims).
3. **Selected-solution transitive ancestry** — how much of the contribution graph flows into the *winning* output (the strong test of compounding).

The measurements pool 200 ProgramBench (program reconstruction) runs and 150 IMOProofBench-Advanced (mathematical proof) runs across worker budgets.

---

## Metric 1: Declared dependencies

Every contribution records its Git parent plus any additional commits named in a mandatory `informed_by=` field (a declared-dependency mechanism, described in Section 2.2 of the paper).

- Workers cite earlier contributions in nearly every eligible episode: **99.9% on ProgramBench, 96.2% on IMOProofBench-Advanced**.
- Later workers build upon **94.7% (ProgramBench) vs 44.4% (proofs)** of published contributions by naming them as Git parents or `informed_by=` sources.
- Important caveat from the appendix: a declared dependency does not by itself prove the worker consumed the referenced artifact — it records intent, so it must be cross-checked against trace evidence (Metric 2).

## Metric 2: Trace-visible artifact consumption

Workers are prompted to leave informative, reusable artifacts in their commits, meant to be consumed by future workers. Since declared dependencies don't establish actual consumption, the paper also analyzes artifact consumption directly from execution traces.

- On ProgramBench, **79.9–82.5% of eligible voluntary artifact bundles are actively reused**, often across Git branches.
- On the proof runs, workers opened 583 of 759 computational probe bundles; 172 were later executed or adapted while citing their producers; 52 fed contributions in the selected solution's ancestry.
- Required reasoning/evaluation artifacts (protocol files like `MY_THOUGHTS.md`, `EVALUATION.md`) are accessed in roughly **98–99% of eligible revisions** in both domains.

The headline interpretation: persistent memory is genuinely inspected and used — but observed reuse is not the same as causal contribution to performance, and reads that never reach the final solution still count as coordination.

## Metric 3: Selected-solution transitive ancestry

Following both Git-parent and `informed_by=` edges recursively from the selected winner shows which part of the contribution graph was *carried into the solution* rather than merely produced during the run (task root and nomination records excluded).

- **ProgramBench: 81.6–92.6%** of the contribution graph is covered by the winner's transitive ancestry, on average across budgets. Git-parent ancestry alone accounts for only ~a quarter; declared cross-branch dependencies connect most of the rest. In other words, repeated local composition from a few sources produces a broad final ancestry.
- **Proof runs: 21.3–33.0%.** The final solutions appear relatively early in the graph; later workers evaluate and nominate existing candidates rather than extending them.
- Neither benchmark shows a consistent increase in coverage with budget.
- Concrete example (FFmpeg, budget 100): the winner's transitive ancestry covers 83.7% of the graph; only a few late commits lie outside it.

Why Git-parent ancestry is small (~25%) in both: with up to five concurrent workers per problem, several physical branches advance at once, so a single linear chain cannot contain most contributions. The cross-branch synthesis mechanism (`informed_by=`) does the real work.

---

## Artifact categories and consumption stages (Appendix F.2)

### Artifact categories

Categories were derived bottom-up and separately per benchmark (by reviewing paths, contents, diffs, commit metadata, and declared dependencies of files workers created). Counting uses **logical bundles**: files created together in one artifact directory that serve the same role count as one bundle, since raw file counts are highly skewed (some workers committed thousands of related output files from one experiment).

ProgramBench (6 categories):
- **Captured evidence** — raw outputs, logs, diffs, snapshots, transcripts from executions.
- **Human-readable findings** — prose recording an observed behavior, diagnosis, or validation result.
- **Exploratory probe** — executable experiment used to discover unknown behavior.
- **Validation harness** — repeatable executable check for an understood behavior or implementation.
- **Input fixture** — input or corpus created to exercise candidate behavior.
- **Artifact documentation** — index or instructions for using an artifact collection.

IMOProofBench-Advanced (4 + residual):
- **Computation or probe** — script/notebook for symbolic, numerical, or combinatorial checks (dominant: 759 of 863 bundles).
- **Captured evidence** — stored output from a proof-related computation or external check.
- **Auxiliary proof note** — informal derivation, lemma, case analysis, or critique outside the submitted proof.
- **Proof-edit aid** — helper used to transform, compare, or inspect proof text.
- **Other research artifact** — rare files not covered by the four.

Production is **sparse and bursty across tasks**: different problems favor different kinds of evidence, and many runs create no voluntary artifact of a given type. More compute only raises some categories (human-readable findings: 32% of B100 runs → 62% of B500 runs); the rest stay problem-dependent.

### Consumption stages

Three stages separate weak from strong reuse:

1. **Accessed** — a later trace explicitly reads, searches, parses, or compares a read-only artifact, or opens an executable artifact. (Passive inheritance and directory listings do *not* count.)
2. **Actively reused** — a read-only artifact informs a later contribution; an executable probe/test/tool/fixture is executed, imported, adapted, or supplied to another computation. For voluntary artifacts, the consumer must also cite the producing commit.
3. **Winner-linked** — the consuming contribution lies in the selected winner's combined Git-parent + `informed_by=` ancestry.

Findings: the proof runs show similarly frequent access but much lower active reuse than ProgramBench — most opened computation/probe bundles are never executed or adapted by a citing contribution. But that need not mean the artifacts were useless: a computation may rule out an approach after being read, while the metric requires it to be run or adapted and cited. Fewer than a quarter (22.6%) of proof protocol-file revisions connect to the winner through a consuming contribution, vs. ProgramBench artifacts usually entering its broad selected ancestry. Consumption and final synthesis are separate outcomes.

---

## Workflow archetypes (Appendix F.3)

From commit graphs and `informed_by=` edges, the paper identifies recurring collaboration patterns (descriptive labels, not imposed roles; one graph can contain multiple patterns, and the dominant pattern for a problem can change with budget):

- **Late integration** (example: Delta B100) — separately developed branches improve different parts of a candidate; a broad synthesis combines independently developed repairs late in the run (at ~80%), and the selected candidate builds on the strongest integration. Decisive work depends on several mature branches, not one steadily refined line.
- **Divide-and-conquer** (example: Pandoc B100) — related but more specialized: separate groups develop distinct subsystems (document structure + inline syntax on one branch; output modes + code-block handling on another) before the branches join.
- **Search-and-pivot** (example: Cheat B500) — workers spend the first 35% of the run on one subsystem (CLI/config), then renderer experiments appear and are quickly combined; renderer work becomes the dominant selected lineage and ends in a final renderer synthesis. A gradual change in the dominant search direction, not an abrupt replacement of all earlier work.

---

## The ProgramBench / proof asymmetry

The same substrate produces two different accumulation regimes:

| | ProgramBench (code) | IMOProofBench-Advanced (proofs) |
|---|---|---|
| Declared-dep citation rate | 99.9% | 96.2% |
| Contributions later built upon | 94.7% | 44.4% |
| Voluntary bundle active reuse | 79.9–82.5% | much lower (172/583 opened probes executed/adapted) |
| Winner transitive ancestry | 81.6–92.6% | 21.3–33.0% |
| Winner appearance timing | late (median 93–98% of run) | early (median 9–18% of run) |
| Mature work mix | sustained construction/synthesis | shifts early to selection (44→70% of episodes as budget grows) |

Interpretation: programming **accumulates** contributions into a final implementation; proof-solving **uses** earlier work to verify and compare candidates, selecting among them. Low proof ancestry does not mean the rest of the proof work was useless — alternative proofs, critiques, and validation can decide *which* candidate is selected without becoming ancestors of the exact winning commit. Persistent state supports both **synthesis and selection**; winner ancestry measures only the former. The two behaviors must be measured separately.

---

## Relevance to locust

This section is the evaluation methodology for locust's memory channel. The paper answers "how do we know the shared memory actually compounds?" — that is exactly the question locust must answer about its vault. Adapt its three metrics:

1. **Dependency declarations** — when an agent writes to the memory channel, it should name which existing vault pages/commits it built on (the `informed_by=` analog: explicit backlinks or dependency fields in front matter). Coverage target: the fraction of new contributions that declare sources, and the fraction of published pages that are later cited.
2. **Transitive ancestry of selected work** — when a deliverable is produced from vault knowledge, trace its dependency graph back through declared links. The "compounding ratio" (how much of the vault's content flows into shipped work) is the primary health metric: a vault everyone writes to but nobody's outputs reference is a write-only graveyard.
3. **Consumption stages** — track the three-stage funnel per artifact type: **Accessed** (agent opened/read a page) → **Actively reused** (the page informed a later contribution, with citation) → **Winner-linked** (the consuming work was selected/shipped). Access without reuse and reuse without selection are both informative — they measure the difference between a browsable archive and a compounding one.

Design takeaways for the skill: make declaring dependencies cheap and mandatory (like the paper's required `informed_by=` field); make artifacts categorized so consumption can be measured per type (evidence, findings, probes, harnesses, fixtures, docs — the paper's categories map naturally onto vault page types); and expect asymmetric reuse across task types — exploratory/verification work may never enter a final deliverable's ancestry yet still be valuable, so the system must credit both synthesis and selection, not just ancestry.
