## Arlie Rippin
Computer Science · Distributed Systems & Consensus

### Professional Focus
I design replicated state machines and consensus protocols with explicit failure domains, bounded queues, and deterministic replay. I focus on invariant preservation during leader changes, tail latency under controlled overload, and recovery from a single corrupted replica.

### Flagship Projects & Architecture

#### Raft-Log
A compact educational Raft implementation that serves a deterministic key-value log over HTTP while preserving durable ordered entries.

**Architecture:** The leader owns a single append pipeline, applies committed entries to a B-tree-backed key-value store, and commits through a WAL with fsync. Followers use HTTP `POST /entries` for AppendEntries and `GET /snapshot` for state transfer. A bounded mpsc queue caps pending writes at 4,096 entries, and every operation records a term, index, and checksum in the log.

**Trade-offs:** Chose synchronous fsync before every committed entry over batching for crash recovery with no acknowledged-write ambiguity, and paid roughly 18% lower write throughput. Chose an on-disk B-tree over an in-memory map to keep a 1 GiB dataset within 1.9 GiB of resident memory, and paid extra seek time on random point lookups. Chose a JSON-over-HTTP wire format over Protocol Buffers for readable captures and lower implementation risk, and paid larger request headers and serialization CPU.

**Results:** On a 12th-generation Intel Core i7 laptop, a 10,000-entry batch completed in 412 ms at 4 concurrent writers with an 8 KiB payload and 32 KiB WAL segment size. At the same workload, p50, p95, and p99 commit latency were 18 ms, 47 ms, and 93 ms. A leader crash after fsync recovered all 10,000 entries in 214 ms from the WAL and snapshot. With a 4,096-entry queue and a follower that stopped reading, application memory remained below 1.1 GiB for a 1 GiB data set.

#### MerkleKV
A storage-engine experiment that provides deterministic snapshot repair and reproducible compaction for a large key-value dataset.

**Architecture:** Keys and values are encoded in a fixed-width binary record format and indexed by a B-link tree. A snapshot worker hashes sorted Merkle ranges, while a compactor merges sorted SSTables and replays a segment journal. Replication uses a gRPC status protocol with snapshot IDs, range hashes, and resume offsets; a corrupt peer is replaced from the last verified snapshot.

**Trade-offs:** Chose append-only SSTables plus a small journal over in-place page mutation to make compaction replayable, and paid additional read amplification during merge. Chose a 2 MiB snapshot segment over a single monolithic archive to reduce repair transfer time, and paid 31 extra RPCs for a 64 MiB snapshot. Chose a binary on-disk format over JSON to reduce parsing and storage, and paid a schema migration path that must be tested before every upgrade.

**Results:** On a 12th-generation Intel Core i7 laptop, compaction of 50 million 96-byte records completed in 11 minutes 42 seconds with four compactor workers. Snapshot repair of a 64 MiB dataset transferred 6.8 MiB of changed ranges and completed in 43 seconds over a 1 Gbps loopback link. A replay test of 200 compaction runs produced the same final root hash and the same 50 million-record count. At four workers, peak RSS was 2.7 GiB for a 128 GiB input, bounded by the 4,096-entry queue and 2 MiB segment buffer.

### Technical Foundation

**Core Systems:** `Go`, `x/sync/errgroup`, `gRPC`, `pprof`

**Storage & Data:** `B-tree`, `B-link tree`, `WAL`, `SSTable`, `Merkle tree`

**Infrastructure & Observability:** `Docker`, `Prometheus`, `OpenTelemetry`, `grpcurl`

### How I Build

- Test invariants before optimizing throughput so a correctness regression fails the build instead of surviving into production.
- Bound queues and apply backpressure so a slow follower cannot consume unbounded memory.
- Record term, index, checksum, and replay offsets so failures can be reconstructed deterministically.
- Measure p50, p95, p99, throughput, and peak RSS under a pinned workload before changing a data path.

### Current Explorations

- **Raft: In Search of an Understandable Consensus Protocol** studies a readable safety model and the role of term-based leadership.
- **RFC 9110: HTTP Semantics** studies cache validation, idempotency, and retry boundaries for the wire layer.
- **Linux Cgroups v2** studies cgroup v2 memory and IO controllers as a way to bound worker memory and isolate noisy peers.

### Contact
[GitHub](https://github.com/katrinaolesen)