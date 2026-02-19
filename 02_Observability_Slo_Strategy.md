# Observability & SLO Strategy
### Reliability Engineering Approach for Low-Latency Systems

## Philosophy

In latency-sensitive systems (API gateways, trading control planes, infrastructure services), observability must:

- Detect regressions quickly
- Avoid high-cardinality explosions
- Avoid distorting performance
- Support fast incident process
- Enable measurable reliability targets

---

# 1. RED Metrics Strategy

For request/decision-based services like the rate limiter, the RED framework applies:

- Rate → requests per second
- Errors → rejected decisions / failures
- Duration → latency histogram

### Implemented Metrics

- `ll_limiter_requests_total`
- `ll_limiter_allowed_total`
- `ll_limiter_rejected_total`
- `ll_limiter_request_duration_seconds`
- `ll_limiter_in_flight`

Key design choices:

- Limited label cardinality (algorithm only)
- No per-key labels (prevents metric explosion)
- Histogram placed outside math

---

# 2. Latency Awareness Strategy

In a control-plane decision service:

- p50 shows general performance
- p95 reveals mild contention
- p99 reveals tail amplification and lock contention

Latency histogram supports:

- Regression detection
- Performance gating in CI
- Alert threshold definition

---

# 3. SLI (Service Level Indicator) Definition

For this service:

### Availability SLI
% of valid requests returning a successful HTTP response.

Formula:
successful_requests / total_requests

### Latency SLI
% of requests under threshold latency (~2ms internal processing)

### Correctness SLI
% of limiter decisions matching expected algorithm behavior (unit-tested)

---

# 4. SLO (Service Level Objective) Definition

Example SLO for a trading-related control-plane service:

- 99.99% availability
- 99% of decisions < 2ms
- 0% allocation regressions in hot path

Error budget concept:

If SLO is 99.99%, allowable failure rate is 0.01%.
That error budget represents release and risk tolerance.

---

# 5. Slow/Fast Burn Rate Alerting Strategy

Avoids noise (single errors)
- Fast burn (5m / 1h)
- Slow burn (1h / 6h)

This reduces noise while still detecting real incidents.

Example alert triggers:

- p99 > threshold for 5 minutes
- error rate > 1% sustained for 10 minutes
- in-flight gauge saturation

---

# 6. Alert Fatigue Prevention

Principles:

- No alerts without action
- No duplicate alerts across layers
- Alert on user impact, not internal counters

---

# 7. Dashboard Strategy

A production dashboard for this service would include:

### Panel 1
Requests/sec (by algorithm)

### Panel 2
Latency (p50 / p95 / p99)

### Panel 3
Error rate %

### Panel 4
In-flight requests (saturation detection)

---

# 8. Failure Mode Observability Mapping

| Failure Mode | Observable Signal |
|-------------|-------------------|
| Lock contention | p99 latency spike |
| Memory growth | process RSS increase |
| Metric cardinality explosion | Prometheus scrape time spike |
| Hot key overload | latency skew distribution |

Observability must map directly to known risks.

---

# 9. Continuous Validation

Observability is tied to:

- CI benchmark gating
- Pre-deployment canary metrics
- Post-deploy latency comparison
- Error budget tracking

---

# Summary

This observability and SLO strategy demonstrates:

- Understanding of RED metrics
- Latency-aware engineering discipline
- Error budget thinking
- Burn-rate alert design
- Alert fatigue prevention
- Failure-mode mapping

