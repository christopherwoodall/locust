# locust

Shared memory layer for the swarm — "sort of like the borg."

The idea: one navigable substrate (an Obsidian vault, hosted on GitHub as `locus-vault`) that every agent and one human can read from and write to, distributed as a single skill. Hand any agent the skill and it learns where the memory is, how the channels work, and how to join the swarm.

## Contents

- `vision/borg-vision-brief.md` — the design brief: problem, components (locust repo, locus-vault, Discord auto-onboard), design principles (plugin architecture, secure by design, git-backed, secret filtering, MVC, keep-it-simple), channels (memory / command / more), onboarding flow, open questions.
- `paper/` — distillations of the GitSwarm paper on Decentralized Compounding Inference ([arXiv 2610.04862v1](https://arxiv.org/html/2610.04862v1)), the closest published blueprint for git-backed swarm memory. Start at [`paper/00-index.md`](paper/00-index.md).

## Status

2026-10-06: read-and-distill phase only. Vision brief + paper distillations written; no build decisions made, nothing implemented.

Design notes for later (from the vision brief): plugin architecture, secure by design, git-backed, secret filtering (fail-closed), MVC pattern, channels (memory channel, command channel, …), skill-as-onboarding, Discord auto-onboard (nice-to-have).
