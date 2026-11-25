---
layout: post
title: "distributed systems"
description: "MIT 6.824 / 6.5840 labs in Go: a race-clean RPC key/value server, MapReduce, and a 1,000-line Raft implementation still fighting go test -race."
---

working through [MIT 6.824](https://pdos.csail.mit.edu/6.824/) in Go to get real reps on concurrency, RPC retries, and consensus without hiding behind a framework. all three labs live in one repo with their original commit histories preserved.

[source](https://github.com/loganhutcheson/distributed-systems)

## key/value server (`kv-server/`)

- linearizable `Get`, `Put`, and `Append` over an unreliable network with dropped and duplicated RPCs.
- client-ID and sequence-number deduplication with retry loops so repeated writes never double-apply.
- passes the full `go test -race ./kvsrv` suite cleanly.

## MapReduce (`map-reduce/`)

- ~660 lines of Go split across `coordinator.go`, `worker.go`, and `rpc.go`: task scheduling, intermediate file partitioning, atomic temp-file renames, and straggler timeout reassignment when a worker dies mid-job.

## Raft (`raft/src/raft/raft.go`)

- ~1,000 lines covering leader elections (3A), log replication and `prevLogIndex` log-matching (3B), and state persistence with fast-backup conflict resolution and stale `AppendEntries` rejection (3C).
- **honest status:** 3A–3C are implemented, snapshots (3D) aren’t written yet, and `go test -race` still catches concurrency bugs under heavy contention. Raft is humbling—it’s the best reminder that “looks right when you read it” means nothing until the race detector agrees.
