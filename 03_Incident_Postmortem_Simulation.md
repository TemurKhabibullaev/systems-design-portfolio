# Production Incident Postmortem

## Incident Summary
Order execution latency spiked from 1ms to 20ms during peak trading volume.

## Impact
- Delayed order confirmations
- Temporary PnL degradation
- 0 data loss

## Root Cause
Lock contention introduced after configuration change increasing limiter burst rate.

## Timeline
12:02 – Alert triggered
12:05 – Bridge opened
12:09 – Hotfix rollback
12:14 – Latency normalized

## Corrective Actions
- Added lock contention metrics
- Implemented lock striping
- Added canary performance validation

## Lessons Learned
Performance regressions require controlled rollout and benchmark gating.
