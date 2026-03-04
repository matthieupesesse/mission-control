# Architecture Review Baseline (Task #4)

_Date:_ 2026-03-04  
_Project:_ Mission Control  
_Reviewer:_ Henri

## Scope

Baseline architecture review of the current Mission Control codebase to establish:
- current system shape,
- major strengths,
- highest-priority risks,
- prioritized remediation roadmap.

This is a static review (repository + runtime DB/task context), not a load/perf benchmark.

## 1) Current Architecture Snapshot

## Runtime/Stack
- Next.js 16 + React 19 + TypeScript 5.7
- SQLite (`better-sqlite3`) in WAL mode
- Single deployable app process with in-process scheduler and event bus
- Real-time updates via SSE + gateway websocket integration
- Role-based auth (viewer/operator/admin) via session cookie or API key

## Core Subsystems
- **UI shell + panels:** app-router SPA style shell for operations surface
- **API layer:** broad REST surface (`/api/*`) for tasks, agents, integrations, settings
- **Data layer:** SQLite + migration system + helper utilities
- **Automation layer:** scheduler for backup/cleanup/heartbeat/webhook retry/claude scan
- **Integration layer:** webhooks, gateway connectivity, direct CLI connections

## Deployment Topology (today)
- Primarily single-node deployment pattern (app + DB local filesystem).
- Backup strategy is local file snapshot + retention pruning.

## 2) Strengths (What is already solid)

1. **Pragmatic single-binary style architecture**
   - Low operational complexity and fast local/prod setup.
2. **Security posture improved in key areas**
   - Timing-safe API key compare, CSRF origin checks for mutating requests, host allowlist controls.
3. **Operational maturity signals**
   - Built-in backup + cleanup + heartbeat monitoring and audit logging.
4. **Good extensibility seams**
   - Distinct modules for webhooks, scheduler, gateway, agent sync, and device identity.
5. **Strong feature completeness for orchestration MVP**
   - Tasks, agents, notifications, comms, token tracking, workflows, integrations.

## 3) Key Risks / Gaps

## P0/P1 (Near-term critical)

### R1. Single-process blast radius
App server, scheduler, integration workers, and DB access all run in one process. A heavy/broken background task can degrade API responsiveness.

**Impact:** latency spikes, partial outage, harder SLO guarantees.

### R2. SQLite scaling boundary for write-heavy multi-actor scenarios
WAL helps, but high-frequency writes across activities/notifications/tokens/comments can become contention points as concurrency grows.

**Impact:** lock contention, slower writes, growth pain under fleet scale.

### R3. Missing explicit domain boundaries
The codebase is modular, but many concerns remain in shared app runtime. This increases coupling and complicates independent scaling.

**Impact:** slower change velocity and higher regression risk.

## P1/P2 (Important)

### R4. Scheduler timing/ownership in multi-instance future
Current scheduler pattern assumes one runtime owner. In horizontal deployment, duplicate job execution risk must be explicitly controlled.

### R5. Security hardening still has known CSP compromise
`unsafe-inline` remains, as documented. Acceptable short-term for compatibility, but still a meaningful policy gap.

### R6. Limited explicit SLO/error-budget instrumentation
There is logging, but baseline architecture lacks visible service-level targets and degradation policy by subsystem.

## 4) Baseline Recommendations (Prioritized)

## Phase A (0-2 weeks): Hardening baseline
1. **Define architecture decision records (ADRs)** for:
   - single-process strategy,
   - SQLite retention period,
   - scaling trigger thresholds.
2. **Introduce subsystem health/SLO dashboard baselines**:
   - API p95 latency,
   - DB lock/wait frequency,
   - scheduler task duration/failure rates,
   - webhook queue retries.
3. **Add explicit scheduler singleton guard** (DB lease/lock or designated leader instance).
4. **CSP hardening plan** with inventory of inline style dependencies and migration path.

## Phase B (2-6 weeks): Separation of concerns
1. **Extract background workers** (scheduler/webhooks/scans) into separate process role(s).
2. **Implement write-path guardrails**:
   - batching/debouncing for noisy activity streams,
   - capped retention defaults with policy docs.
3. **Create clear module contracts** between UI/API/domain/integration paths.

## Phase C (6-12 weeks): Scale-ready posture
1. **Prepare storage abstraction for dual mode**:
   - SQLite for single-node,
   - optional Postgres profile for multi-node scale.
2. **Define horizontal deployment model**:
   - worker/web role separation,
   - leader election/lock strategy,
   - cross-instance event propagation strategy.
3. **Formalize resilience tests** (chaos-lite): task flood, webhook flood, worker restart, DB pressure.

## 5) Suggested Target Architecture (Next Baseline)

- **Web/API role:** stateless request handling + auth + API contract
- **Worker role:** scheduler, webhooks, periodic scanners, heavy async jobs
- **Shared data plane:** DB + durable event/outbox table
- **Observability layer:** metrics + structured logs + alert thresholds
- **Security layer:** tightened CSP + host allowlist + RBAC + audit already in place

This preserves current product velocity while creating a clean migration path to higher scale.

## 6) Immediate Actions for Task Tracking

Recommended task decomposition from this baseline:
1. ARCH-01: Scheduler singleton/leader lock
2. ARCH-02: SLO + health metric baseline instrumentation
3. ARCH-03: Background worker extraction design doc
4. ARCH-04: CSP hardening implementation plan
5. ARCH-05: Data growth and retention policy (activities/notifications/tokens)

---

## Conclusion
Current architecture is strong for an advanced orchestration MVP and local/self-hosted operation. The key next step is controlled separation of background workloads from request path and defining objective scale trigger points before growth pressure forces reactive redesign.
