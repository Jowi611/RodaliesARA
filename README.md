# RodaliesARA

## Real-Time Predictive API for Commuter Rail Delays

> **Fictional research and simulation project.** RodaliesARA is an academic-style concept based on the supplied poster brief. It does not connect to real transit systems, does not provide operational predictions, and must not be used to make travel or safety decisions.

RodaliesARA proposes a low-latency API for forecasting how commuter-rail delays may propagate through a regional network. The project models stations and track junctions as a directed spatio-temporal graph, combines recent service observations with historical context, and exposes predicted downstream delay horizons through a lightweight API.

The poster materials also refer to the concept as **RodaliesGraph**. RodaliesARA is the repository and presentation name used here.

## Project goal

Static timetables and status-only applications describe planned or current service, but they do not explain how a disruption at a shared bottleneck may affect downstream stations. RodaliesARA explores whether graph-based machine learning can provide an interpretable, forward-looking estimate of delay propagation while preserving clear uncertainty and data-freshness signals.

### Primary research claim

In the fictional poster scenario, a spatio-temporal graph convolutional network (ST-GCN) forecasts cascading delay risk up to **45 minutes ahead** and reduces passenger arrival-time uncertainty by **78% compared with static timetable feeds**. These numbers are illustrative targets and are not independently validated measurements.

## Conceptual architecture

```text
Synthetic GTFS-RT-like events
          |
          v
  Async ingestion worker  --->  validation and normalization
          |                                  |
          v                                  v
  Directed rail graph  <--------------  feature windows
          |
          v
  ST-GCN / baseline simulator
          |
          v
  Redis-style prediction cache
          |
          v
  FastAPI-compatible REST interface
          |
          v
  Dashboard or transit application
```

### Data flow

1. A simulated worker polls a GTFS-RT-like source every 30 seconds.
2. Trip updates, vehicle positions, timetable drift, weather bands, and time-of-day context are normalized into synthetic records.
3. Stations and junctions become graph nodes; directed track segments become weighted edges.
4. A rolling 60-minute window is transformed into node features such as current delay, dwell-time drift, headway variance, and upstream traffic density.
5. The simulation produces forecasts for 15-, 30-, and 45-minute horizons.
6. Results are returned with model version, generated timestamp, freshness, confidence, and contributing signals.

## Graph and model design

The conceptual network is represented as:

\[
G = (V, E, W)
\]

- **Nodes (`V`)** represent passenger stations, interlocking points, and major junctions.
- **Directed edges (`E`)** represent track segments and their direction of travel.
- **Weights (`W`)** represent synthetic distance, block capacity, and current headway conditions.
- **Node features (`X_t`)** include delay seconds, dwell-time drift, upstream train density, and cyclical time context.

A graph convolution layer aggregates information from adjacent stations, while a temporal component processes twelve five-minute frames covering a rolling 60-minute window. A production implementation might use PyTorch Geometric, but this repository intentionally keeps its example code dependency-light and deterministic.

## Fictional poster results

The poster brief describes the following benchmark targets. They are included to make the repository useful for experimentation, not as evidence of a deployed system:

| Metric | Fictional target |
|---|---:|
| Mean absolute error at 30 minutes | 1.8 minutes |
| Static timetable baseline MAE | 7.4 minutes |
| Forecast horizon | 15, 30, and 45 minutes |
| API response target | under 65 ms |
| Load-test throughput target | over 1,200 requests/second |
| Memory target | under 512 MB per worker |
| Poster accuracy headline | 87% |

Any future experiment must report the dataset, split strategy, baseline definition, confidence intervals, and hardware before treating these values as meaningful results.

## API concept

The intended endpoint shape is:

```http
GET /api/v1/delays?station=granollers_nord&horizon=30
```

Example response:

```json
{
  "station": "granollers_nord",
  "horizon_minutes": 30,
  "predicted_delay_minutes": 6.4,
  "risk": "medium",
  "generated_at": "2026-11-03T08:30:00Z",
  "data_freshness_seconds": 12,
  "model_version": "simulated-stgcn-0.1"
}
```

The response is deliberately labeled as simulated output. It contains no passenger information, credentials, real vehicle locations, or operational control commands.

## Repository contents

```text
README.md                 Formal project description and safety boundaries
poster.md                 Fictional poster narrative and presentation script
openapi.yaml              Illustrative API contract
data/sample_delays.json   Synthetic forecast response
docs/architecture.md      Ingestion, graph, serving, and observability notes
LICENSE                   MIT license for the fictional materials
```

## Safe simulation boundaries

This repository is designed for review, scanning, and experimentation:

- All data should remain synthetic or locally generated.
- No live transit endpoint is contacted by the example materials.
- No passenger identity, ticket, payment, or device data is modeled.
- No code should issue commands to railway infrastructure or alter service alerts.
- Performance figures in the poster are claims to reproduce or challenge, not guarantees.
- A real deployment would require operator authorization, data governance, accessibility review, security assessment, and human oversight.

## Evaluation plan

A serious implementation would compare the graph model against at least a scheduled-timetable baseline and a naive moving average. Evaluation should use timestamped arrival events that occur after the prediction cutoff, with time-ordered train/validation/test splits to prevent future leakage. Recommended metrics include MAE, RMSE, calibration error, precision-recall at an operational threshold, latency percentiles, and source-freshness coverage.

Important edge cases include skipped stops, rerouted trains, closed track segments, missing coordinates, stale feeds, shared tunnels, single-track bottlenecks, severe weather, strikes, and service patterns not present during training.

## References from the poster brief

1. Kipf, T. N., and Welling, M. **Semi-Supervised Classification with Graph Convolutional Networks.** ICLR, 2017.
2. Generalitat de Catalunya / Renfe open-data materials and GTFS-Realtime specifications.
3. The supplied fictional poster brief: **RodaliesGraph: Real-Time Predictive API for Commuter Rail Delays.**

## License

These fictional presentation and simulation materials are released under the [MIT License](LICENSE). The license does not grant permission to access or operate real transit systems, nor does it turn illustrative metrics into validated claims.
