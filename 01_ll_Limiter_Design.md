# Low-Latency Rate Limiter  
### Architecture, Performance & Production Hardening

## Overview

This project is a production-style, low-latency rate limiting service built in Go and designed with the constraints of high-throughput and latency-sensitive systems in mind.

Primary design goals:

- Deterministic O(1) decision path
- Zero allocations in the hot path (bench-verified)
- Concurrency safety under high parallel load
- Runtime-selectable algorithms
- Per-key isolation
- Observability-first architecture
- Production-ready containerization
- CI-validated correctness

The system simulates characteristics found in real-world control-plane or trading-infrastructure services where performance regressions, allocation spikes, or contention issues directly impact reliability.

---

## Functional Capabilities

### Multiple Algorithms

Supported algorithms:

- Token Bucket
- Sliding Window

Algorithm selection is runtime-configurable via environment variables, which allows:

- Controlled A/B testing
- Behavioral tuning per deployment
- Infrastructure-level experimentation
- Zero rebuild required for switching strategy

---

### Per-Key Isolation

Each key maintains independent limiter state.

Benefits:
- Prevents hot-user amplification
- Models API gateway isolation
- Mimics multi-tenant protection strategies
- Avoids cross-client performance degradation

---

### HTTP Control Plane

Exposed endpoints:

- `GET /v1/allow`
- `GET /healthz`
- `GET /metrics`

Design principles:
- Business logic isolated from HTTP transport layer
- Minimal request handling overhead
- Deterministic handler latency
- Observability instrumentation without decision-path pollution

---

## Performance Design

### Zero Allocations in the Hot Path

The `Allow()` decision path:

- Performs no heap allocations
- Avoids slice growth
- Avoids reflection
- Avoids dynamic resizing during steady state
- Uses bounded critical sections

Benchmarks confirm:
0 allocs/op

Why this matters:

- Reduces GC pressure
- Prevents tail latency spikes
- Ensures predictable performance under load
- Mirrors performance discipline in low-latency systems

---

### Concurrency Model

Current design is optimized for correctness and bounded latency.

Future optimization path:
- Sharded bucket maps
- Lock striping
- Reduced contention under extreme parallel load

---

## Observability & Metrics

The service exposes Prometheus metrics at `/metrics`.

### RED Metrics Implemented

- Total requests
- Allowed requests
- Rejected requests
- Latency histogram
- In-flight request gauge

Design considerations:

- Metrics observation happens outside the algorithm math
- Histogram buckets use default bounded distribution
- Label cardinality minimized

---

## Benchmarking & Performance Verification

Benchmarks included in-repo:
go test -bench=. -benchmem

Scenarios tested:

- Single-key high-frequency load
- Multi-key distributed load
- Variable cost requests
- Concurrency simulation

Benchmark goals:

- Verify zero allocations
- Measure ns/op latency
- Identify contention characteristics
- Ensure algorithm parity comparison

---

## CI/CD & Production Hardening

### GitHub Actions Pipeline

Automated pipeline includes:

- Formatting enforcement
- Race detection (`-race`)
- Unit tests
- Benchmark capability
- Deterministic build validation

This ensures production-readiness and repeatable correctness.

---

### Dockerized Production Image

Built with multi-stage Docker build:

- Builder stage
- Distroless runtime stage
- Non-root execution user

Benefits:

- Minimal attack surface
- Reduced CVE exposure
- Smaller image size
- Predictable execution environment
- Production deployment readiness

---

## Failure Modes & Risk Analysis

Identified risks:

- Memory growth from unbounded key cardinality
- Lock contention under extreme concurrency
- High-cardinality metric labels
- Time drift affecting refill precision

Mitigations:

- Key TTL policies (future extension)

---

## Architectural Intent

This project intentionally demonstrates:

- Concurrency reasoning
- Allocation discipline
- Latency awareness
- Algorithm abstraction
- Observability-first mindset
- Production containerization standards
- CI-integrated validation

---

## Future Extension Path

Planned improvement areas:

- Sharded internal state map
- Distributed Redis-backed limiter
- Cross-node token synchronization
- Hot-key contention analysis
- Advanced latency profiling (pprof/eBPF)
- Deterministic benchmark gating in CI

---

## Summary

This limiter represents a production-style control-plane service built with focus on:

- Deterministic performance
- Concurrency safety
- Observability discipline
- Measurable latency behavior
- Containerized production deployment
- Continuous validation via CI

The system demonstrates readiness for environments where performance, reliability, and deterministic behavior directly impact business outcomes.

