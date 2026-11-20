# Stock-trading-order-matching-engine-AOOP
# Stock Trading Order Matching Engine

A high-throughput, in-memory **Stock Trading Order Matching Engine** built in **Java 21** using strict price-time (FIFO) priority. Designed to eliminate relational database row-lock contention during market-open micro-bursts while enforcing production-grade Advanced Object-Oriented Programming (AOOP) invariants, failure resilience, and deterministic crash recovery.

---

## Target Scale & Performance Goals

| Metric | Target / Benchmark |
| :--- | :--- |
| **Daily Retail Volume** | 10M+ orders/day (~444 ord/s average) |
| **Market-Open Peak Burst** | 4,400 orders/sec (10x spike) |
| **Matching Latency Target** | < 100 μs (in-memory hot path) |
| **Live Market Data Fan-Out** | Decoupled non-blocking L2 tick broadcast |
| **Chaos Resilience** | Zero double executions under 30% injected failure rate |

---

## System Architecture

The system is structured as a **backend-first Java 21 application** (paired with a lightweight live web dashboard for order book visualization and burst simulation) split into **four isolated Bounded Contexts**:

```text
[ Order Gateway ] ──(Bounded Queue / ACL)──▶ [ Matching Core (Single-Thread CLOB) ]
   Idempotency &                                     │                    │
   Invariant Checks                        (Immutable Outbox)    (Volatile L2 Snapshot)
                                                     ▼                    ▼
                                           [ Portfolio Ledger ]    [ Market Data ]
                                           PostgreSQL & WAL        Observer Tick Feed
```

### End-to-End Order Pipeline
1. **Order Ingress:** Idempotency-key deduplication blocks duplicate client retries.
2. **Validation:** Sealed type and domain invariant checks reject illegal orders before matching.
3. **Hot Path (Symbol Matcher):** Thread-confined `TreeMap` (Red-Black Tree) + `ArrayDeque` (FIFO) + `HashMap` (O(1) cancel) per stock symbol with zero locks.
4. **Outbox Ledger:** Asynchronous WAL & Transactional Outbox persistence to PostgreSQL with optimistic versioning against write-skew.
5. **Live Fan-Out:** Lock-free `volatile` Level-2 order book snapshots and Observer broadcasts.

---

## AOOP Engineering Alignment (CO1–CO6)

| Unit / CO | Focus Area | Implementation in Engine |
| :--- | :--- | :--- |
| **U1–U2 (CO1)** | Value Objects & Invariants | Immutable `Price` (`long` paise), positive `Quantity`, strongly typed `OrderId`, zero public setters, `filled <= orderQty`. |
| **U3 (CO2)** | Sealed Contracts & LSP | `sealed` order hierarchy (`Limit`, `Market`, `StopLoss`) verified by a shared JUnit contract test suite. |
| **U4 (CO3)** | Behavioral Patterns | `Strategy` (Price-Time / extensible Pro-Rata), explicit `State` machine (`Open → Partial → Filled → Cancelled`), non-blocking `Observer`. |
| **U5 (CO4)** | Failure Survival | Idempotency keys, Circuit Breaker, explicit `UNKNOWN` timeout state, Transactional Outbox + WAL replay. |
| **U6 (CO5)** | Concurrency & JMM | Per-symbol thread confinement, `volatile` L2 snapshots, optimistic ledger versioning, JMH locked vs. lock-free benchmarks. |
| **U7 (CO6)** | Modularity & Boundaries | 4 Bounded Contexts, strict schema ownership, Ports & Adapters (Dependency Inversion), ArchUnit boundary tests, ADRs. |

---

## Dataset & Simulation Strategy

Because exchange engines process live event streams rather than static tables, we use a **hybrid dataset**:
* **Real Historical Price Seeding:** Historical NSE/LOBSTER market data (e.g., `TCS`, `INFY`, `RELIANCE`) to seed realistic price levels and volatility.
* **Deterministic Java Order-Flow Simulator:** Replays seeded market flow while injecting `4,400/s` market-open bursts, `StopLoss` cascades, and `30%` duplicate/network-failure chaos events.

---

## Tech Stack

* **Core:** Java 21 LTS, NIO
* **Data Structures:** `TreeMap`, `ArrayDeque`, `HashMap`
* **Persistence:** PostgreSQL (Portfolio Ledger & Outbox), Append-Only WAL
* **Build, Testing & Benchmarking:** Maven 3.9, JUnit 5, ArchUnit, JMH (Java Microbenchmark Harness)
* **Observability:** VisualVM, JDK Flight Recorder (JFR)
* **Demo UI:** Lightweight Web Dashboard (Live L2 Order Book & Burst Trigger)

---

## Project Roadmap

### Phase 1: Architecture & Design (Review-1) — Completed
- [x] Domain modeling, capacity math, and market benchmarks
- [x] 5-year literature survey (2021–2026) and research gap analysis
- [x] Bounded-context architecture, pipeline design, and AOOP CO1–CO6 mapping

### Phase 2: Core Engine & Contracts (Review-2) — In Progress
- [ ] Immutable value objects (`Price`, `Quantity`, `OrderId`) and `sealed` order hierarchy
- [ ] Core `OrderBook` (`TreeMap` + `ArrayDeque` + `HashMap`) with Price-Time matching
- [ ] Shared LSP JUnit 5 contract test suite & ArchUnit module-boundary tests
- [ ] Deterministic Java Order-Flow Simulator (hybrid NSE/LOBSTER seeding)

### Phase 3: Resilience, Benchmarks & Dashboard (Final)
- [ ] Idempotent Gateway, Bounded Queue back-pressure, WAL & Transactional Outbox
- [ ] 30% chaos-injection test suite and deterministic crash-replay verification
- [ ] JMH benchmarks (thread-confined lock-free vs. locked baseline) + JFR profiling
- [ ] Lightweight live web dashboard for real-time order book visualization

---

## Team & Academic Info

* **Course:** Advanced Object-Oriented Programming (`25CS2205E`)
* **Institution:** Department of CSE, KLH (Deemed to be University), Hyderabad
* **Project Guide:** Dr. Sudha, Professor, Dept. of CSE

| Name | Register Number |
| :--- | :--- |
| Padmavati | 2510030085 |
| Konda Venkata Srinath | 2510030106 |
| Satvik Pachipulusu | 2510030227 |
