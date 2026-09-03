# AISecOpt — AI-Enhanced Cloud SIEM for Threat Detection and Infrastructure Optimisation

**MSc Dissertation Artefact**
Author: Tijani Tobiloba
Submission: September 2026

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

## 2. Live links

| What | Link | Notes |
|---|---|---|
| Repository | `https://github.com/teebee09/aisecopt-project` | Full code, notebooks (with saved output), models, documentation |
| Hosted dashboard | `https://teebee09.github.io/aisecopt-project/dashboard/index.html` | Structure, navigation and design only — see Section 2.1 |
|---|---|---|
| CSE-CIC-IDS2018 Dataset | `https://www.unb.ca/cic/datasets/ids-2018.html` | The location of the dataset on Canada Institute of Cybersecurity |
| AWS CSE-CIC-IDS2018 Utilised | `https://registry.opendata.aws/cse-cic-ids2018/` | AWS host dataset of collabroative effort between CSE and CIC |

### 2.1 Important: the hosted dashboard shows no live data

The dashboard modules query a local Elasticsearch instance (`http://localhost:9200`) directly from the browser. This works when the dashboard is served from the same machine running the project's Docker stack (see Section 4), but a visitor's browser on the hosted GitHub Pages link has no such instance to reach, so live panels appear empty or static.

**This is a stated scope boundary, not a fault.** The hosted link demonstrates the interface design, navigation model, and six-module architecture described in Chapter 3 of the dissertation. To see the system operating on live data, either run it locally (Section 4) or refer to the notebooks below, which contain real, saved results from actual runs.

---

## 3. Quick evaluation guide for examiners

If you have limited time, these artefacts demonstrate the core contribution without requiring any setup — all output cells are saved and viewable directly on GitHub:

| What to look at | Where | What it shows |
|---|---|---|
| Data cleaning, ML training, SHAP, MITRE mapping, Elasticsearch ingestion | `notebooks/01_eda_and_cleaning.ipynb` | Full pipeline from raw data through XGBoost training (0.936 weighted F1), SHAP explainability, and MITRE ATT&CK mapping — all in one notebook |
| LSTM forecasting | `notebooks/02_lstm_forecasting.ipynb` | Metricbeat extraction, LSTM training, forecast evaluation |
| Baseline comparison | `notebooks/03_baseline_comparison.ipynb` | Rule-based SIEM baseline (0.033 F1) vs. AI model (0.936 F1) on identical test data |
| Performance analysis | `notebooks/04_performance_analysis.ipynb` | Ingestion throughput and host resource utilisation |
| Working dashboard | `dashboard/index.html` (see Section 4 to run locally with live data) | Six linked modules |

---

## 4. Running the system locally (for live data)

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

The dashboards query Elasticsearch directly from the browser, which requires CORS enabled (pre-configured in `docker-compose.yml`) and the files served over HTTP rather than opened directly from the filesystem.

**If Elasticsearch contains no data**, the dashboards will render with empty panels. Running `notebooks/01_eda_and_cleaning.ipynb` through to its ingestion cells populates the security indices; `notebooks/02_lstm_forecasting.ipynb` populates the infrastructure indices.

---

## 5. Repository structure

```
aisecopt-project/
├── notebooks/
│   ├── 01_eda_and_cleaning.ipynb      Data acquisition, cleaning, sampling, XGBoost training,
│   │                                   SHAP explainability, MITRE mapping, Elasticsearch ingestion
│   ├── 02_lstm_forecasting.ipynb      Metricbeat extraction, LSTM training, forecasting
│   ├── 03_baseline_comparison.ipynb   Rule-based baseline vs. AI model
│   └── 04_performance_analysis.ipynb  Throughput and resource utilisation measurement
├── dashboard/
│   ├── index.html                     Landing page and navigation hub
│   ├── ai_secopt_framework_dashboard.html          SOC operations dashboard
│   ├── ai_secopt_threat_intelligence_console.html  MITRE ATT&CK and SHAP feed
│   ├── ai_secopt_lstm_forecasting_engine.html      Infrastructure forecasting
│   ├── ai_secopt_ml_training_lab.html              Model evaluation interface
│   ├── ai_secopt_data_pipeline_engineer.html       Pipeline and index monitoring
│   └── ai_secopt_incident_correlation_console.html Cross-signal correlation
├── models/
│   ├── xgb_model.pkl                  Trained XGBoost classifier
│   ├── scaler.pkl                     MinMaxScaler fitted on training partition only
│   ├── label_encoder.pkl              Class label mapping (15 classes)
│   ├── metricbeat_scaler.pkl          Scaler for infrastructure telemetry
│   └── lstm_model_v1_preliminary.keras  Trained LSTM forecaster
├── outputs/
│   ├── shap_summary_global.png        Global SHAP feature importance
│   ├── baseline_vs_xgboost.csv        Baseline comparison results
│   └── performance_analysis.csv       Throughput and resource utilisation results
├── docs/
│   ├── SETUP.md                       Full reproduction instructions
│   ├── ARCHITECTURE.md                System design and component rationale
│   └── aisecopt_architecture.png      Architecture diagram
├── docker-compose.yml                 Elasticsearch and Kibana deployment
└── README.md                          This file
```

**Note on data files:** the `data/` directory is excluded from version control via `.gitignore`. The CSE-CIC-IDS2018 dataset is approximately 6 GB and is publicly available from the source cited in Section 7; `docs/SETUP.md` includes the exact download command.

---

## 6. Known limitations

Stated plainly rather than concealed; each is discussed in more depth in the dissertation.

**The system is not deployed in the cloud.** Elasticsearch, Kibana, and the dashboards run locally via Docker. Cloud capability was validated separately by deploying Metricbeat on an AWS EC2 instance streaming to a managed Elastic Cloud deployment, but the AISecOpt platform itself is not cloud-hosted. The GitHub Pages link (Section 2) hosts the static dashboard files only.

**Security and infrastructure data are temporally disjoint.** The security dataset was captured in 2018; infrastructure telemetry was collected in 2026 from a different machine. The correlation mechanism is implemented and operational — it computes the real time delta between the most recent detection and telemetry reading — but correctly reports no overlap, because none exists between these particular datasets.

**Infrastructure monitoring covers one host, not a datacentre.** Telemetry comes from a single development machine and, separately, one EC2 instance. Multi-node panels in the dashboards are explicitly labelled as illustrative.

**Detection operates on recorded flows, not live traffic.** There is no packet capture or live flow extraction. The classifier is evaluated against a benchmark dataset.

**Network throughput forecasting is unreliable.** The LSTM achieves approximately 1% error on memory forecasting but performs poorly on network throughput due to a documented distribution shift.

**Power and thermal metrics were not collected.** These are not exposed by Metricbeat, nor accessible to a cloud tenant.

**Isolation Forest was not implemented.** Considered during design, scoped out to preserve focus on the two-model architecture.

**"Optimisation" denotes forecast-informed visibility, not an autonomous decision mechanism.** The LSTM's forecasts support a human decision; the system does not currently act on them.

---

## 7. Data sources and attribution

**CSE-CIC-IDS2018** — a joint project of the Communications Security Establishment (CSE) and the Canadian Institute for Cybersecurity (CIC), University of New Brunswick. Processed CSV flow records were obtained from the AWS Open Data Registry.
https://www.unb.ca/cic/datasets/ids-2018.html

Citation: Sharafaldin, I., Habibi Lashkari, A. and Ghorbani, A.A. (2018) 'Toward generating a new intrusion detection dataset and intrusion traffic characterization', *4th International Conference on Information Systems Security and Privacy (ICISSP)*, Portugal.

**Infrastructure telemetry** — self-collected using Elastic Metricbeat 8.13.0 from the author's own development machine and an author-provisioned AWS EC2 instance. No third-party or personal data is involved.

---

## 8. Environment

- Python 3.11 (Anaconda)
- Elasticsearch 8.13.0, Kibana 8.13.0 (Docker)
- Metricbeat 8.13.0
- XGBoost, scikit-learn, imbalanced-learn, SHAP, TensorFlow/Keras, pandas, PyArrow

Exact versions and installation commands are in `docs/SETUP.md`.
