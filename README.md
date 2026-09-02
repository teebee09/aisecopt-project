# AISecOpt — AI-Enhanced Cloud SIEM for Threat Detection and Infrastructure Optimisation

**MSc Dissertation Artefact**
Author: Tijani Tobiloba
Submission: August 2026

---

## 1. What this system is

AISecOpt is a functional prototype of an AI-augmented Security Information and Event Management (SIEM) platform. It addresses a single research question:

> *How can artificial intelligence be integrated with a cloud-based SIEM architecture to improve network threat detection while simultaneously enabling predictive infrastructure monitoring and optimisation?*

The prototype does three things that conventional SIEM deployments typically keep separate:

1. **Explainable threat classification** — a supervised model classifies network flows into fifteen categories (benign plus fourteen attack types), and every prediction is accompanied by a SHAP explanation identifying which traffic characteristics drove that decision.
2. **Predictive infrastructure monitoring** — a recurrent neural network forecasts host resource utilisation from self-collected telemetry, supporting proactive rather than reactive capacity management.
3. **Cross-signal correlation** — a correlation console queries both streams and reports empirically whether security detections and infrastructure readings overlap in time.

All outputs are indexed into a single Elasticsearch instance and surfaced through a unified dashboard layer of six linked modules.

---

## 2. Quick evaluation guide for examiners

If you have limited time, these three artefacts demonstrate the core contribution:

| What to look at | Where | What it shows |
|---|---|---|
| Model training and evaluation | `notebooks/02_ml_training.ipynb` | XGBoost training, 0.936 weighted F1, per-class confusion analysis, SHAP explainability |
| Baseline comparison | `notebooks/03_baseline_comparison.ipynb` | Rule-based SIEM baseline (0.033 F1) vs. AI model (0.936 F1) on identical test data |
| Working dashboard | `dashboard/index.html` (see §4 to run) | Six linked modules reading live data from Elasticsearch |
| Correlation evidence | `dashboard/ai_secopt_incident_correlation_console.html` | Live timestamp-overlap check between detections and telemetry |

The system does not require a full rebuild to inspect. All notebooks contain saved output cells showing the results as originally executed.

---

## 3. Repository structure

```
aisecopt-project/
├── notebooks/
│   ├── 01_eda_and_cleaning.ipynb      Data acquisition, cleaning, stratified sampling
│   ├── 02_ml_training.ipynb           XGBoost training, SHAP, MITRE mapping, ES ingestion
│   ├── 02_lstm_forecasting.ipynb      Metricbeat extraction, LSTM training, forecasting
│   ├── 03_baseline_comparison.ipynb   Rule-based baseline vs. AI model
│   └── 04_performance_analysis.ipynb  Throughput and resource utilisation measurement
├── dashboard/
│   ├── index.html                     Landing page and navigation hub
│   ├── ai_secopt_framework_dashboard.html      SOC operations dashboard
│   ├── ai_secopt_threat_intelligence_console.html   MITRE ATT&CK and SHAP feed
│   ├── ai_secopt_lstm_forecasting_engine.html  Infrastructure forecasting
│   ├── ai_secopt_ml_training_lab.html          Model evaluation interface
│   ├── ai_secopt_data_pipeline_engineer.html   Pipeline and index monitoring
│   └── ai_secopt_incident_correlation_console.html  Cross-signal correlation
├── models/
│   ├── xgb_model.pkl                  Trained XGBoost classifier
│   ├── scaler.pkl                     MinMaxScaler fitted on training partition only
│   ├── label_encoder.pkl              Class label mapping (15 classes)
│   ├── metricbeat_scaler.pkl          Scaler for infrastructure telemetry
│   └── lstm_model_v1_preliminary.keras  Trained LSTM forecaster
├── outputs/
│   ├── confusion_matrix.png
│   ├── shap_summary_global.png
│   ├── lstm_training_history.png
│   ├── baseline_vs_xgboost.csv
│   └── performance_analysis.csv
├── docs/
│   ├── SETUP.md                       Full reproduction instructions
│   ├── ARCHITECTURE.md                System design and component rationale
│   └── aisecopt_architecture.png      Architecture diagram
├── docker-compose.yml                 Elasticsearch and Kibana deployment
└── README.md                          This file
```

**Note on data files:** the `data/` directory is excluded from version control via `.gitignore`. The CSE-CIC-IDS2018 dataset is approximately 6 GB and is publicly available from the source cited in §6; `docs/SETUP.md` includes the exact download command.

---

## 4. Running the system

Full instructions are in `docs/SETUP.md`. The abbreviated path:

```bash
# 1. Start the Elastic Stack
docker compose up -d

# 2. Confirm Elasticsearch is responding
curl http://localhost:9200

# 3. Serve the dashboard (from the dashboard directory)
cd dashboard
python -m http.server 8080

# 4. Open in a browser
http://localhost:8080/index.html
```

The dashboards query Elasticsearch directly from the browser. This requires CORS to be enabled, which is pre-configured in `docker-compose.yml`. Serving the files over HTTP rather than opening them from the filesystem is necessary — browsers block cross-origin requests from `file://` origins regardless of server configuration.

**If Elasticsearch contains no data**, the dashboards will render with empty panels. Running `notebooks/02_ml_training.ipynb` through to the ingestion cells populates the security indices.

---

## 5. Known limitations

These are stated plainly rather than concealed; each is discussed in more depth in the dissertation.

**The system is not deployed in the cloud.** Elasticsearch, Kibana, and the dashboards run locally via Docker. Cloud capability was validated separately by deploying Metricbeat on an AWS EC2 instance streaming to a managed Elastic Cloud deployment, but the AISecOpt platform itself is not cloud-hosted.

**Security and infrastructure data are temporally disjoint.** The security dataset was captured in 2018; infrastructure telemetry was collected in 2026 from a different machine. The correlation mechanism is implemented and verifiably operational — the Incident Correlation Console computes the actual time delta between the most recent detection and the most recent telemetry reading — but it correctly reports no overlap, because none exists between these particular datasets. The limitation lies in the data pairing, not the mechanism.

**Infrastructure monitoring covers one host, not a datacentre.** Telemetry comes from a single development machine and, separately, one EC2 instance. Multi-node panels in the dashboards are explicitly labelled as illustrative.

**Detection operates on recorded flows, not live traffic.** There is no packet capture or live flow extraction. The classifier is evaluated against a benchmark dataset.

**Network throughput forecasting is unreliable.** The LSTM achieves approximately 1% error on memory forecasting but performs poorly on network throughput due to a documented distribution shift between the training and test periods.

**Power and thermal metrics were not collected.** These are not exposed by Metricbeat, nor accessible to a cloud tenant.

**Isolation Forest was not implemented.** Considered during design, scoped out to preserve focus on the two-model architecture. The dashboard tab is labelled accordingly.

---

## 6. Data sources and attribution

**CSE-CIC-IDS2018** — a joint project of the Communications Security Establishment (CSE) and the Canadian Institute for Cybersecurity (CIC), University of New Brunswick. Processed CSV flow records were obtained from the AWS Open Data Registry. Publicly available under the terms stated at:
https://www.unb.ca/cic/datasets/ids-2018.html

Citation: Sharafaldin, I., Habibi Lashkari, A. and Ghorbani, A.A. (2018) 'Toward generating a new intrusion detection dataset and intrusion traffic characterization', *4th International Conference on Information Systems Security and Privacy (ICISSP)*, Portugal.

**Infrastructure telemetry** — self-collected using Elastic Metricbeat 8.13.0 from the author's own development machine and an author-provisioned AWS EC2 instance. No third-party or personal data is involved.

---

## 7. Environment

- Python 3.11 (Anaconda)
- Elasticsearch 8.13.0, Kibana 8.13.0 (Docker)
- Metricbeat 8.13.0
- XGBoost, scikit-learn, imbalanced-learn, SHAP, TensorFlow/Keras, pandas, PyArrow

Exact versions and installation commands are in `docs/SETUP.md`.
