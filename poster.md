# RodaliesARA: Real-Time Predictive API for Commuter Rail Delays

**Fictional poster presentation · Data Systems & Urban Mobility Track**

**Authors:** Aina Soler, Marc Vidal, and the RodaliesARA Research Group  
**Affiliation:** Centre for Applied Transit Analytics, Barcelona  
**Status:** Concept and reproducible design study

---

## 1. Motivation

Commuter rail passengers often learn about a delay only after it has already affected their journey. Official alerts are essential, but they are generally reactive: they describe an observed incident rather than the probability that a train will arrive late at a future station.

RodaliesARA asks:

> **Can heterogeneous, continuously arriving signals provide an actionable and calibrated forecast of delay risk without exposing passenger data?**

The project targets a practical interface between transport operations, passenger-information systems, and open-data research.

## 2. Research contribution

We propose a small, explainable prediction service with four properties:

1. **Real time:** ingest observations continuously and publish forecasts with explicit data freshness.
2. **Multi-horizon:** estimate the chance of at least 5, 15, and 30 minutes of delay.
3. **Explainable:** return the strongest contributing signals, not only a score.
4. **Resilient:** degrade gracefully when a feed is late, incomplete, or unavailable.

The API is designed as a complement to official alerts, never as a replacement for operational authority.

## 3. System design

```text
+-------------------+     +------------------+     +----------------------+
| GTFS-RT / AVL     | --> |                  | --> | Feature store         |
| service alerts    |     | Stream normalizer|     | rolling train/station |
| weather / events  | --> | and quality gate|     | windows               |
+-------------------+     +------------------+     +----------+-----------+
                                                                  |
                                                                  v
                                                     +------------+-----------+
                                                     | Forecast service       |
                                                     | calibrated gradient    |
                                                     | boosting + baselines   |
                                                     +------------+-----------+
                                                                  |
                                                                  v
                                                     +------------+-----------+
                                                     | REST API + monitoring  |
                                                     | forecasts, explanations|
                                                     | freshness, confidence  |
                                                     +------------------------+
```

See [`docs/architecture.md`](docs/architecture.md) for the component and reliability notes.

## 4. Data and method

The fictional study uses a six-month, privacy-preserving event stream containing:

- scheduled and observed arrival/departure times;
- train position and headway estimates;
- station dwell-time history;
- service alerts and disruption categories;
- precipitation, wind, and temperature bands;
- calendar, weekday, and special-event indicators.

A gradient-boosted classifier estimates delay thresholds at each forecast horizon. A persistence baseline and scheduled-time baseline are retained for comparison. Probability calibration is performed on a time-ordered validation window to avoid leakage from future service conditions.

### Feature safeguards

- Features are cut off at the prediction timestamp.
- Late-arriving observations carry an age and quality flag.
- Passenger identifiers, ticket records, and device locations are excluded.
- Each response includes `data_freshness_seconds` and source health metadata.

## 5. Illustrative evaluation

The following fictional results demonstrate the poster's reporting format. They are not measurements of a real network.

| Forecast target | AUROC | Brier score | Recall at 0.70 precision |
|---|---:|---:|---:|
| ≥5 min late within 15 min | 0.86 | 0.142 | 0.61 |
| ≥15 min late within 30 min | 0.89 | 0.118 | 0.68 |
| ≥30 min late within 60 min | 0.91 | 0.097 | 0.73 |

Relative to the scheduled-time baseline, the proposed model improves early detection while preserving a visible distinction between low-confidence forecasts and confirmed operational alerts.

## 6. Example API response

```json
{
  "train_id": "R4-4821",
  "station_code": "BCN-SVC",
  "generated_at": "2026-10-14T08:42:11Z",
  "data_freshness_seconds": 18,
  "predictions": {
    "5m": {"probability": 0.82, "expected_delay_minutes": 9},
    "15m": {"probability": 0.64, "expected_delay_minutes": 17},
    "30m": {"probability": 0.41, "expected_delay_minutes": 22}
  },
  "top_signals": ["preceding_train_late", "high_station_dwell", "wet_weather"]
}
```

The complete contract is in [`openapi.yaml`](openapi.yaml), with a larger illustrative payload in [`data/sample_delays.json`](data/sample_delays.json).

## 7. Responsible deployment

Predictions should be displayed as probabilistic guidance with a timestamp and an unmistakable distinction from official alerts. A production rollout would require:

- operator review and incident-response ownership;
- accessibility testing for color, language, and screen readers;
- drift monitoring across lines, stations, seasons, and disruption types;
- an audit trail for model version, input freshness, and displayed output;
- a fallback to scheduled information when confidence or data quality is inadequate.

## 8. Limitations and future work

A model trained on ordinary operations may underperform during rare failures, strikes, infrastructure works, or severe weather. Correlated delays can also cause a forecast to appear more certain than it is. Future work should test conformal prediction intervals, causal disruption features, federated evaluation across operators, and human-centered experiments measuring whether uncertainty labels improve decisions.

## 9. Takeaway

**RodaliesARA turns live operational signals into transparent, time-bounded delay risk.** Its central design choice is to make uncertainty and freshness first-class API fields, so predictive information can support—rather than obscure—official transport communication.

---

### Reproducibility note

This repository contains the fictional poster narrative, an illustrative OpenAPI contract, sample data, and architecture notes. No real-time feeds are connected and no operational prediction should be inferred from the example values.
