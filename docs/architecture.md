# Architecture notes

## Runtime path

1. **Adapters** ingest GTFS-RT/AVL positions, service alerts, weather bands, and event calendars.
2. **Normalizer** maps operator-specific messages to stable train, station, and disruption entities.
3. **Quality gate** validates timestamps, deduplicates events, records source age, and marks missing inputs.
4. **Feature store** maintains rolling windows for dwell time, headway, preceding-train delay, and corridor congestion.
5. **Forecast service** computes calibrated multi-horizon probabilities and attaches model and freshness metadata.
6. **API gateway** applies rate limits, exposes `/forecasts` and `/health`, and emits request metrics.

## Reliability principles

- A stale source is visible in the response rather than silently treated as current.
- If required inputs fail quality thresholds, the API returns `503` or a degraded response according to the endpoint contract.
- Model versions are immutable and included in every forecast for auditability.
- The scheduled-time baseline remains available as a fallback and monitoring comparator.
- Logs contain operational identifiers and timestamps, not passenger identity or ticket data.

## Observability

Recommended metrics include forecast latency, source freshness, missing-feature rate, calibration error by horizon, drift by line and station, and the ratio of forecasts displayed beside official alerts. Alerts should distinguish infrastructure incidents from data-pipeline failures.

## Deployment shape

A production implementation could run adapters and stream processing on a managed event bus, keep recent features in a low-latency store, and deploy the stateless forecast API behind a regional gateway. Batch retraining should be isolated from the online path and promoted only after time-ordered validation and operator review.
