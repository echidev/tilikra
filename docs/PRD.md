# Tilikra — Product Requirements Document (PRD)

| Field | Value |
|---|---|
| Product | Tilikra — Digital Fraud Risk & Investigation Platform |
| Document | Product Requirements Document |
| Version | 1.0 |
| Status | Draft for implementation |
| Date | 19 October 2026 |
| Primary market context | Indonesia; demonstrated using appropriately licensed / permitted public datasets |
| Primary users | Fraud Analysts, Risk Data Analysts, Fraud Operations, Risk Modelling Analysts |
| Delivery target | End-to-end portfolio MVP in 8 weeks |

---

## 1. Executive Summary

Tilikra is a portfolio-grade, enterprise-style decision-support platform that demonstrates an end-to-end fraud analytics workflow: data intake, quality validation, feature generation, rule-based detection, machine-learning scoring, alert prioritisation, case investigation, auditability, and operational monitoring.

The product's core use case is **transaction-level e-commerce fraud analytics**. It uses the IEEE-CIS Fraud Detection competition dataset as an offline research and evaluation source. The data is historical, includes a fraud label for its training split, and contains anonymised or coded attributes. It is not Indonesian transaction data and is governed by the Kaggle competition rules rather than an unrestricted open-data licence. Tilikra must state these limitations prominently and must not redistribute the dataset.

An optional URL-intelligence module may accept a URL supplied by an analyst and evaluate it with a separately trained phishing classifier and/or a permitted threat-intelligence lookup. This module is a separate evidence source: it is **not** joined to the IEEE-CIS transaction model unless a real, defensible linkage key exists. The MVP must not imply that URL detections and transaction labels describe the same events.

Tilikra is a decision-support system for demonstrations and analyst workflows. It does not execute payments, block real accounts, or make production financial decisions. Any decision involving an actual customer or transaction would require institution-specific validation, controls, access management, regulatory assessment, and authorisation outside the scope of this project.

## 2. Problem Statement

Fraud teams need to identify suspicious activity while avoiding excessive friction for legitimate users. A raw probability score alone is insufficient for operational use. Analysts need to know why an alert was raised, how urgently it should be reviewed, what evidence is available, what the investigation concluded, and how system performance changes over time.

A portfolio that only trains a classifier fails to demonstrate several practical capabilities expected in risk analytics:

- translating a business risk into measurable outcomes;
- preparing and validating data while avoiding temporal leakage;
- comparing understandable rules with a supervised model;
- selecting thresholds against review capacity and error trade-offs;
- turning scores into a prioritised alert queue;
- documenting case outcomes and an auditable decision trail;
- monitoring data quality and model behaviour; and
- clearly communicating what a dataset does and does not prove.

Tilikra addresses these gaps in a single coherent product.

## 3. Product Vision and Positioning

**Vision:** help a fraud analyst move from a data signal to a documented, reviewable decision in one workflow.

**Positioning:** an offline-first fraud analytics workbench that joins a transparent rules layer, a validated risk-scoring model, and a case-management workflow.

**Portfolio proposition:** show practical competence across analytics, applied machine learning, data engineering fundamentals, business reasoning, and controls—not algorithm novelty alone.

## 4. Goals and Non-Goals

### 4.1 Product Goals

1. Provide a reproducible pipeline for preparing the permitted IEEE-CIS dataset and generating a leakage-aware evaluation split.
2. Establish a meaningful baseline using explainable rules and Logistic Regression, then compare it with a gradient-boosted tree model.
3. Return a score, model version, and human-readable reason codes for a transaction record available in the application.
4. Prioritise alerts based on a configurable score/rule policy and a simulated analyst review capacity.
5. Allow an analyst to create, assign, review, update, and close a case while retaining an append-only history of case events.
6. Provide metrics for model quality and operational workload, clearly separating measured metrics from simulated policy outputs.
7. Provide local-first deployment using open-source components and Docker Compose, with a constrained public demo option.
8. Explain relevance to Indonesian anti-fraud practice without claiming local model validation or regulatory compliance.

### 4.2 Non-Goals

- Live connection to a bank, payment gateway, e-wallet, card processor, or real customer account.
- Automated transaction decline, account blocking, payment cancellation, or fund recovery.
- A claim of fraud detection effectiveness on Indonesian transactions.
- Redistribution of IEEE-CIS files, copies, or artifacts that violate its current competition terms.
- Treating anonymised/coded IEEE-CIS features as identifiable Indonesian customer attributes.
- Using a URL phishing score as a transaction feature without a real join key and an appropriate point-in-time availability rule.
- Building a production-grade streaming platform, full feature store, graph database, or multi-tenant SaaS product for the MVP.
- Providing legal, regulatory, or AML determinations.

## 5. Target Users and Personas

### 5.1 Fraud Analyst

Needs to triage alerts quickly, understand which signals caused them, examine the available transaction context, record investigation notes, and reach a defensible case outcome.

### 5.2 Risk Data Analyst

Needs to query and aggregate data, compare fraud rates by available segment, analyse false positives, explain changes in performance, and communicate recommendations to the risk team.

### 5.3 Junior Data Scientist — Risk Analytics

Needs to establish baselines, engineer features, validate models, select operating thresholds, assess calibration where applicable, and document limitations and monitoring requirements.

### 5.4 Fraud Operations Lead / Risk Manager

Needs a compact view of alert volumes, analyst workload, case outcomes, model/rule performance, changes over time, and the limits of the available evidence.

## 6. Key User Journeys

### Journey A — Prepare and validate a dataset

1. The developer obtains the dataset directly through the Kaggle competition interface and accepts the applicable rules.
2. The ingestion command checks required files, columns, row counts, duplicate IDs, and join cardinality.
3. The pipeline writes validated columnar intermediates and a data-quality report.
4. A time-aware split manifest is generated reproducibly and recorded with the dataset run.

### Journey B — Score and prioritise transactions

1. A permitted transaction record enters the offline scoring interface or the batch-scoring job.
2. The feature pipeline validates inputs and generates only features available at scoring time.
3. The rules engine evaluates configured rules; the model generates a risk score.
4. The decision layer combines configured signals into a priority, returning reason codes and model/rule versions.
5. The record becomes an alert candidate. The system does not automatically block or decline the transaction.

### Journey C — Investigate an alert

1. An analyst opens an alert and views the score, triggered rules, available attributes, and data provenance.
2. The analyst creates or opens a case, records a hypothesis/evidence note, and updates the case status.
3. Each status change and note event is recorded with actor, timestamp, and before/after state where relevant.
4. The analyst closes the case with an outcome: `confirmed_fraud`, `not_fraud`, `inconclusive`, or `duplicate`.
5. Case labels are kept distinct from the original dataset's `isFraud` label.

### Journey D — Review performance

1. A user selects a dataset run, model version, evaluation window, and operating threshold.
2. The dashboard displays ML metrics and separate operational metrics.
3. The report distinguishes offline dataset results, application usage telemetry, and hypothetical policy simulations.
4. The user exports an aggregate report without exporting raw restricted data.

### Journey E — Optional URL intelligence check

1. An analyst manually submits a URL in a separate URL-intelligence screen.
2. Tilikra validates and parses the string; it does not automatically visit the URL.
3. The optional phishing model and/or permitted threat-intelligence API returns findings, source, timestamp, and limitations.
4. The result is stored as a separate indicator check. It is not attached as a transaction-model feature unless an independently justified linkage exists.

## 7. Scope and Priorities

Priority notation: **P0** = required for MVP; **P1** = useful extension after the end-to-end path works; **P2** = future capability.

### 7.1 Data and Pipeline

- **P0:** manual, documented acquisition path for IEEE-CIS files; raw files remain outside Git.
- **P0:** schema validation, required-column checks, duplicate transaction-key checks, and join-cardinality reporting.
- **P0:** data dictionary that marks each field as documented, coded/anonymised, or derived.
- **P0:** reproducible preparation and time-aware split manifests.
- **P0:** Parquet intermediates and a local analytical query layer.
- **P1:** scheduled refresh command for permitted external URL indicators.

### 7.2 Detection and Scoring

- **P0:** configurable rule engine with rule ID, version, rationale, input fields, and trigger output.
- **P0:** Logistic Regression baseline and LightGBM challenger.
- **P0:** probability/risk score, version information, and reason-code output.
- **P0:** threshold selection with PR-based metrics and top-K review-capacity analysis.
- **P0:** rule-only, model-only, and combined strategy comparison.
- **P1:** Isolation Forest anomaly ranking as a challenger; must not label anomalies as confirmed fraud.
- **P1:** SHAP explanations for the tabular model where computationally practical.

### 7.3 Alerts and Case Workflow

- **P0:** alert list with filter/sort by priority, score, rule, status, and time available in the dataset/application.
- **P0:** case create/assign/status update/close workflow.
- **P0:** event history, notes, and source evidence references.
- **P0:** outcome taxonomy and distinction between dataset labels and analyst review outcomes.
- **P1:** simple local roles (viewer, analyst, admin) for the demo environment.

### 7.4 Reporting and Monitoring

- **P0:** model evaluation report with PR-AUC / Average Precision, precision, recall, confusion matrix at selected thresholds, and top-K metrics.
- **P0:** operational dashboard with alert volume, case statuses, review workload, and outcome rates when application data exists.
- **P0:** data-quality and model-version visibility.
- **P0:** explicit labelling of simulated metrics and unavailable measures.
- **P1:** data drift summaries and scheduled evaluation report.

### 7.5 API and Demo

- **P0:** FastAPI scoring and alert/case endpoints.
- **P0:** Streamlit analyst workbench.
- **P0:** Docker Compose local stack and one-command setup guidance.
- **P0:** automated tests and CI checks.
- **P1:** constrained, read-only public demo using only data that can legally be published.

## 8. Business and Product Metrics

Tilikra will define metrics before experimenting. Every metric must specify its numerator, denominator, evaluation period, label source, and any known selection bias.

### 8.1 Offline Model Metrics

- **Average Precision / PR-AUC:** primary discrimination summary for the imbalanced classification setting.
- **Precision and recall:** reported at explicitly named thresholds.
- **Precision@K / Recall@K:** measures the usefulness of the top K ranked records under a simulated review capacity.
- **ROC-AUC:** supplementary only; not sufficient on its own.
- **Calibration metrics:** Brier score/reliability plot only if the score is interpreted as a probability and calibration is evaluated.
- **Error analysis:** false-positive and false-negative slices by available fields and time windows.

### 8.2 Operational Metrics

- Alert count and alert-rate by run/window.
- Cases opened, closed, and outstanding by status.
- Alert-to-case conversion, only when alert and case linkage is available and the definition is stated.
- Review workload and time-to-action, based on application event timestamps—not inferred from the historical dataset.
- False-positive workload, only when a sufficiently mature reviewed outcome exists.

### 8.3 Metrics Not Claimed by the MVP

- Actual loss prevented, money recovered, or fraud loss avoided.
- Production false-positive rate in Indonesia.
- Live detection latency under bank/payment-network throughput.
- Regulatory compliance certification.

## 9. Business Rules and Decision Principles

1. A model score is a prioritisation signal, not a final legal or fraud determination.
2. A triggered rule is an explainable observation, not proof of fraud.
3. An anomaly is not automatically fraud.
4. Dataset `isFraud` labels and analyst case outcomes must remain separately stored and separately named.
5. Case closure must not silently rewrite historical model evaluation labels.
6. No automatic customer-impacting action is performed by the MVP.
7. Any metric based on hypothetical costs, analyst capacity, or policy choices must be labelled as a scenario/simulation.
8. Model and rule versions must be visible for every scoring decision.

## 10. Indonesian Context and Regulatory Alignment

Tilikra is intended as a learning and portfolio demonstration informed by Indonesian financial-services practice. POJK No. 12 Tahun 2024 on Anti-Fraud Strategy for Financial Services Institutions describes four pillars: prevention; detection; investigation, reporting and sanctions; and monitoring, evaluation and follow-up. Tilikra maps product capabilities to these concepts as an educational control map, not a claim of compliance or a substitute for an institution's own governance framework.

| Anti-fraud concept | Tilikra capability | Limitation |
|---|---|---|
| Prevention | Rule catalogue and documented risk scenarios | Does not implement institutional preventive controls |
| Detection | Rules, scoring, alert priority | Evaluated on a non-Indonesian research dataset |
| Investigation/reporting | Case workflow, evidence references, event history | Does not submit regulatory reports |
| Monitoring/evaluation | Metrics, model versioning, drift summaries | No live institution data or production SLA |

Indonesia-specific scam context can inform the typology catalogue and analyst guidance. Aggregated official statistics must not be misused as transaction-level training labels.

## 11. Data Access, Licensing, and Governance

### 11.1 IEEE-CIS Fraud Detection

- Official access: https://www.kaggle.com/competitions/ieee-fraud-detection/data
- Access requires a Kaggle account and acceptance of the current competition rules.
- The dataset is subject to competition rules; it must not be described as an unrestricted open-licensed dataset.
- Raw source files, extracted copies, and restricted dataset derivatives must not be committed or redistributed unless current terms explicitly permit it.
- Some features are anonymised/coded. The model cannot claim interpretable meaning for unknown fields without defensible evidence.
- `TransactionDT` is a relative time delta, not a calendar timestamp.
- The dataset is not Indonesian transaction data.

### 11.2 Optional PhiUSIIL URL Dataset

- Official source: https://archive.ics.uci.edu/dataset/967/phiusil-phishing-url-dataset
- License listed by UCI: Creative Commons Attribution 4.0 International (CC BY 4.0).
- Preserve attribution and record the version/source.
- The dataset includes URL and webpage-derived features. The URL-only MVP model must not depend on features that require fetching the webpage unless a separate, safe acquisition process is approved.

### 11.3 Optional URLhaus Enrichment

- API documentation: https://urlhaus.abuse.ch/api/
- Treat API access, key requirements, fair-use restrictions, freshness, and redistribution rights according to current service terms.
- Store source and check timestamp. Avoid bulk replication of third-party feeds.
- Submitted URLs must not be fetched by the app. The enrichment adapter must call only the documented provider endpoint.

### 11.4 Data Retention

- Raw data: local ignored path; source-controlled code only.
- Processed files: local ignored path or explicitly permitted storage; do not publish restricted rows.
- Public demo: use a tiny approved sample, aggregate outputs, or a separate CC BY 4.0 URL dataset; do not upload raw IEEE-CIS data.
- Logs: avoid storing full URL query strings, credentials, or unnecessary personal data.

## 12. Non-Functional Requirements

### Performance

- The preparation workflow should use memory-conscious columnar processing (DuckDB/Polars) and should not require a GPU.
- The web UI should paginate alert tables and avoid loading all records into application memory.
- Initial scoring latency target is a local development target, measured and reported after implementation; it is not a production SLA.

### Reliability

- Pipeline runs must be reproducible with recorded parameters, split manifest, code revision, and model version.
- Failed ingestion or model loading must return an actionable error, not silently continue with stale data.

### Security and Privacy

- No real customer PII or credentials are required.
- API secrets must be server-side environment variables and excluded from source control.
- URL input is treated as untrusted text and is never automatically fetched by Tilikra.
- The demo should be read-only or rate-limited; no public write access to cases without authentication or controlled demo mode.

### Accessibility and Usability

- Risk decisions must have readable reason codes and definitions.
- Tables and charts should use explicit labels, units, time windows, and dataset/run identifiers.
- The UI must visibly state that outputs are research/demo results, not production financial advice or automatic blocking decisions.

## 13. MVP Acceptance Criteria

The MVP is accepted when all P0 criteria below are met.

1. A developer can follow the README to obtain permitted data, configure the local environment, and reproduce the preparation pipeline.
2. Data validation produces a clear pass/fail report, including expected columns, duplicates, nulls, and join coverage.
3. A reproducible time-aware split is generated and documented.
4. A baseline and challenger model are trained and evaluated on a holdout that was not used for model/threshold selection.
5. The evaluation report includes class imbalance, PR-based metrics, selected threshold, top-K metrics, and error analysis.
6. Rule-only, model-only, and combined approaches can be compared.
7. The scoring API returns a score, version information, and reason codes with schema validation.
8. An alert can be created, reviewed, assigned (or left unassigned), and closed with a defined outcome.
9. Case event history is persisted and can be inspected.
10. Dashboard distinguishes offline model metrics from application workflow metrics and simulations.
11. All P0 automated tests pass in CI.
12. Docker Compose starts the supported local application stack.
13. README documents dataset access rules, source attribution, limitations, and the absence of Indonesian transaction-level validation.
14. Raw restricted datasets and secrets are absent from public Git history.

## 14. Delivery Plan

| Phase | Weeks | Deliverables |
|---|---:|---|
| Foundation | 1–2 | Dataset audit, data model, pipeline, data-quality reporting |
| Detection | 3–4 | Rules, baseline/challenger, validation, threshold analysis |
| Product workflow | 5–6 | API, database persistence, case workflow, analyst UI |
| Operationalisation | 7–8 | Monitoring, Indonesian context map, tests, Docker, documentation, demo |

## 15. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Competition-data restrictions misunderstood | Licence / distribution risk | Direct acquisition from Kaggle; record current terms; exclude raw and derived restricted files from Git |
| Coded/anonymised columns over-interpreted | False business conclusions | Mark semantics as unknown unless documented; use defensible features first |
| Leakage inflates performance | Unreliable model claim | Time-aware split; point-in-time feature rules; separate threshold-selection and test data |
| Non-Indonesian data presented as local proof | Misleading portfolio claims | Prominent external-validity limitations and Indonesia control mapping separate from model validation |
| False positives create excessive review load | Poor operational utility | Top-K analysis, threshold scenarios, rule/model comparison |
| URL enrichment creates SSRF or unsafe browsing | Security risk | Never visit user-provided URLs; only send validated requests to configured API provider endpoints |
| Scope creep | MVP remains unfinished | Core scope stays transaction analytics and investigation; optional URL checks are kept separate |
| Public demo exposes restricted data | Compliance and reputational risk | Read-only demo using approved outputs and no raw IEEE-CIS rows |

## 16. Open Decisions

These decisions should be recorded before implementation or resolved through explicit Architecture Decision Records (ADRs):

1. Exact time split boundaries and test duration, selected after an initial audit of `TransactionDT` and label prevalence.
2. Whether to make URL intelligence part of the MVP or leave it as a clearly isolated P1 module.
3. Whether Postgres is mandatory for all local runs or whether a minimal offline mode should use DuckDB only.
4. Hosting choice for a public demo, based on current free-tier constraints and data-permission review.
5. Final operating threshold(s), selected from validation data and stated as demonstration policy rather than a production recommendation.

## 17. References

- Kaggle, IEEE-CIS Fraud Detection competition data and rules: https://www.kaggle.com/competitions/ieee-fraud-detection/data
- UCI Machine Learning Repository, PhiUSIIL Phishing URL Dataset (CC BY 4.0): https://archive.ics.uci.edu/dataset/967/phiusil-phishing-url-dataset
- OJK, POJK No. 12 Tahun 2024 on Anti-Fraud Strategy for Financial Services Institutions: https://www.ojk.go.id/en/regulasi/Pages/12-Tahun-2024-Penerapan-Strategi-Anti-Fraud-bagi-Lembaga-Jasa-Keuangan.aspx
- URLhaus API documentation and terms: https://urlhaus.abuse.ch/api/
