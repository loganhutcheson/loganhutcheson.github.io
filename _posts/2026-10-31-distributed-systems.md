---
layout: post
title: "distributed systems"
description: "MIT 6.824 / 6.5840 labs in Go—key/value server, MapReduce, and Raft (3A–3D) passing under go test -race—plus two notes on coding with agents a year apart."
---

working through [MIT 6.824](https://pdos.csail.mit.edu/6.824/) in Go to get real reps on concurrency, RPC retries, and consensus without hiding behind a framework. all three labs live in one repo with their original commit histories preserved.

[source](https://github.com/loganhutcheson/distributed-systems)

## key/value server (`kv-server/`)

- linearizable `Get`, `Put`, and `Append` over an unreliable network with dropped and duplicated RPCs.
- client-ID and sequence-number deduplication with retry loops so repeated writes never double-apply.
- passes `go test -race ./kvsrv` cleanly.

## MapReduce (`map-reduce/`)

- ~660 lines of Go split across `coordinator.go`, `worker.go`, and `rpc.go`: task scheduling, intermediate file partitioning, atomic temp-file renames, and straggler timeout reassignment when a worker dies mid-job.
- passes the full `test-mr.sh` suite, including worker crash recovery.

## Raft (`raft/src/raft/raft.go`)

- elections (3A), log replication and fast-backup (3B), persistence (3C), and snapshots / `InstallSnapshot` (3D).
- originally built through 3C by hand (with early 2025 agent friction), then finished and refactored in [PR #1](https://github.com/loganhutcheson/distributed-systems/pull/1) into 788 lines of lock-disciplined Go with a dedicated `sync.Cond` applier and snapshot-aware slice log.
- all **28 tests (3A–3D)** pass under `go test -race`, verified across 3 consecutive stress runs on the hardest unreliable/crash/snapshot tests.

---

## an ode to agents — September 8, 2025

from [`raft/src/raft/an_ode_to_agents`](https://github.com/loganhutcheson/distributed-systems/blob/main/raft/src/raft/an_ode_to_agents) (`cad8ee5`):

> When I started this project in 2023-24, there was the development loop:
>
> **[1]**
> Understand the code context youself.
> Ask GPT questions.
> Enter them yourself.
>
> **[2]**
> Then, in 2025 agents came out and the loop changed to:
> Ask GPT to fix code.
> GPT understands code context.
> GPT changes code.
> Review changes.
>
> What I have found for the last 2 months of trying to complete this project, is that the new agent loops is incompatible for complex system development and maitenance. The more power I give to these agents, in understanding and changing the context themselves, the less I am able to modify and understand the system myself. In example, I have the entire 100% of the code context in my own brain in the loop #1. Now in loop 2, I started with 100% of the code context, but as GPT makes changes, I lose that understanding. I now no longer have the competency to debug and maintain the code. That is human out of the loop.
>
> So, my current conclusion, is that the human must be in the loop to a greater extent beyond just review of changes. My only hope for completing this project is reverting the last 2 months of changes, and doing it all over myself with loop [1].
>
> Linus himself states:
> VIBE = Very Inefficient But Entertaining

## an ode to agents, part 2 — November 25, 2026

from [`an_ode_to_agents_revised.txt`](https://github.com/loganhutcheson/distributed-systems/blob/main/an_ode_to_agents_revised.txt) (`c4ee26c`):

> Well, in mid 2025 I wrote "an_ode_to_agents". Essentially, I let—what was it then—Cursor? GPT-5? take over the development of my raft. It couldn't get the test cases passing. I remember spending at least one entire work session, but likely multiple sessions, and came to the conclusion "complex system development is not compatible with agentic human out of the loop...the more I give to these agents the less I can maintain them myself."
>
> Well, it is November 2026. A year ago, I was half right.
> Gemini Neon 4 one shot this project in 10 minutes.
>
> I don't really know how to feel - you know? I knew it was smarter. But it BARREL dozed it. Demolished. Plowed. I worked my ass off on this project, many nights at the library.
>
> Okay, some professional thoughts:
>
> 1. While I cannot fully own the complexity of this system I can test it end-to-end and be a part of the verification process.
> 2. Software is now a black hole. Slop needs a new term, for this isn't slop. This is just pure efficiency and technology replacing an outdated development process.
> 3. The training and context data matters. This is the smaller hill that I will die on. I have submitted the intellectual and contextual ownership of the entire project. Agents are better. But, the fact that I worked my ass off. I wrote long, painstaking notes. I calculated. That history is present in the commits. That matters. Agents built off it. They perfected my imperfections. Let's hope I don't die on this hill in one year from now...
