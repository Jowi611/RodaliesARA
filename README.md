# RodaliesARA: Real-Time Predictive API for Commuter Rail Delays

A fictional poster presentation and reference implementation concept for predicting commuter-rail delays in the Rodalies network.

> **Fictional research project.** The data, metrics, and endpoints in this repository are illustrative and are not an operational service.

## Poster

Read the complete presentation in [`poster.md`](poster.md). It covers motivation, system design, methodology, evaluation, limitations, and deployment.

## API concept

The OpenAPI contract is available at [`openapi.yaml`](openapi.yaml). Example response data is in [`data/sample_delays.json`](data/sample_delays.json).

## Core claims

- Forecasts a train's probability of arriving at least 5, 15, or 30 minutes late.
- Returns calibrated confidence, contributing signals, and a freshness timestamp.
- Supports corridor, station, and train-level queries for passenger-information systems.
- Uses a privacy-preserving design: no passenger identity or ticket data is required.

## Repository structure

```text
poster.md                 Full fictional poster presentation
openapi.yaml              Predictive API contract
data/sample_delays.json   Illustrative API response
docs/architecture.md      Architecture and deployment notes
```

## Discussion questions

1. How should uncertainty be communicated when a prediction can influence travel choices?
2. What governance is needed before model output is shown beside official service alerts?
3. How can the system remain useful during sensor outages or novel disruptions?

## License

This fictional presentation is released under the MIT License. See [`LICENSE`](LICENSE).
