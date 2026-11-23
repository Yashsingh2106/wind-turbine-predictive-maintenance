# IoT Predictive Maintenance for Wind Turbines

HCLTech Career Shaper Programme | III Year Industry Project 2025-26 | Project No. 5 | Embedded IoT
Learner: Tanishq Kaushik, ABES Engineering College

A Raspberry Pi edge node reads vibration, temperature, rotor RPM and wind speed, and flags abnormal turbine behaviour
before failure. This repository contains the data analysis, the anomaly detector, the edge-node code and a web dashboard.

**Live dashboard:** https://claude.ai/artifact/KLQJCUKnKUXkAvyS47Q1pk  (replay of dataset.csv; also available as `dashboard/index.html`, open it in any browser)
**GitHub repository:** <paste your repository link here>

## Honest scope
- `data/dataset.csv` has **no fault examples** (`fault_label` is 0 in all 500 rows). Detectors learn *normal* behaviour and were
  validated with **synthetic** faults, so reported accuracy is not field accuracy.
- The dashboard is a **replay of the dataset**, not a live sensor feed. `src/edge_node.py` has a tested `--simulate` mode; its
  hardware sensor code and ThingSpeak publishing are **not tested on real hardware**.

## Structure
| Path | Purpose |
|---|---|
| `src/analysis.py` | Cleaning, EDA (8 figures), baseline model, 3 detectors, synthetic-fault validation, exports `data/model_params.json` |
| `src/detector.py` | `TurbineMonitor`: baseline residuals + rolling z-score (WARNING) + point z-score (CRITICAL) |
| `src/edge_node.py` | Raspberry Pi loop: sensors -> detector -> console / ThingSpeak MQTT (TLS). `--simulate` replays the dataset |
| `dashboard/` | `template.html` + `build_dashboard.py` -> self-contained `index.html` |
| `data/` | `dataset.csv`, `dataset_cleaned.csv`, `data_dictionary.csv`, `results.json`, `model_params.json` |
| `docs/` | Project report and synopsis (Word and PDF) |
| `figures/` | Charts and architecture diagrams |

## Run
```bash
pip install -r requirements.txt
python src/analysis.py                                   # figures, cleaned data, model parameters, metrics
python src/edge_node.py --simulate --inject-fault --limit 60   # console demo with a synthetic fault
python dashboard/build_dashboard.py                      # rebuild dashboard/index.html
```
Raspberry Pi with ThingSpeak (credentials in environment variables, never in code):
```bash
export TS_CHANNEL_ID=... TS_MQTT_USER=... TS_MQTT_PASS=... TURBINE_ID=T01
python src/edge_node.py --publish --interval 600
```

## Key results (synthetic validation, severity 1.0)
Rolling z-score: precision 71.5%, recall 75.1%, F1 0.73. Point z-score: precision 95.6%, recall 58.1%, F1 0.71.
Full details in `docs/Project_Report_IoT_Predictive_Maintenance.pdf`.

## Security and ethics
TLS for MQTT, per-device credentials, no open ports, telemetry kept apart from control networks (IEC 61400-25 guidance).
No personal data. AI assistance (Claude, Anthropic) is disclosed in Appendix A of the report.
