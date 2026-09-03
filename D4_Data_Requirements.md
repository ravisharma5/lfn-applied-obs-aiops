# D4 Data Requirements — Minimum required data per use case

This document breaks down the 5 identified telecom scenarios into the data each one needs. It was requested by Turkcell during the WG meeting and feeds into D4 (Use Case Catalog) and D5 (Simulated PoC).

Each use case lists three tiers:
- **Minimum viable** — the bare minimum to train and evaluate a baseline model
- **Recommended** — what you'd want for production-grade accuracy
- **Enrichment** — nice-to-have data that improves edge cases or adds explainability

After each use case, there's a self-assessment table. Each operator should fill in what they have, what they don't, and what they'd be willing to share (even in anonymized or synthetic form).

---

## 1. Fault prediction

**ROI target:** 40% fewer outages

### Data tiers

**Minimum viable:**
- Alarm and event logs (SNMP traps, syslog) with timestamps, severity, and affected network element IDs
- Performance counters: CPU utilization, memory, interface error rates, link utilization — sampled at 5-minute intervals or better
- Network topology: node inventory and adjacency (which nodes connect to which)
- Historical incident tickets with timestamps, fault type, root cause classification, affected elements, and time-to-resolution

**Recommended:**
- OTel-format traces and metrics from CNFs/VNFs (if cloud-native)
- Weather and environmental data correlated with outage events (fiber cuts, power failures)
- Maintenance window schedules (to separate planned from unplanned events)
- Vendor-specific alarm catalogs mapping alarm codes to plain-language descriptions

**Enrichment:**
- Change management logs (config changes preceding faults)
- Capacity planning forecasts (to distinguish overload faults from equipment faults)
- Cross-domain correlation: transport + RAN + core alarms for the same incident

### Labels

Historical incident labels are the critical bottleneck. You need:
- Fault type taxonomy (hardware failure, software bug, config error, capacity overload, external event)
- Root cause classification per incident
- ~1,000+ labeled examples per common fault type for supervised learning
- Rare fault types can use transfer learning or few-shot, but need at least 50-100 examples to validate

### Data format and refresh

| Field | Requirement |
|---|---|
| Format | CSV/JSON/Parquet; OTel OTLP preferred for new instrumentation |
| Refresh rate | Historical batch is fine for training; 5-min or better for inference |
| Time range | 12-24 months minimum to capture seasonal patterns |
| Anonymization | Replace subscriber IDs with hashes; network element names can be pseudonymized as long as topology adjacency is preserved |

### Operator self-assessment

| Data item | AT&T | Verizon | Turkcell |
|---|---|---|---|
| Alarm/event logs | | | |
| Performance counters (5-min) | | | |
| Network topology (node + adjacency) | | | |
| Historical incident tickets with root cause | | | |
| OTel traces from CNFs | | | |
| Maintenance schedules | | | |
| Change management logs | | | |

Fill in: **H** = have it, **D** = don't have it, **S** = can share (anonymized/synthetic), **N** = can't share

---

## 2. Churn prediction

**ROI target:** 15-25% churn reduction

### Data tiers

**Minimum viable:**
- Per-subscriber usage: data volume, voice minutes, SMS counts — monthly aggregates at minimum, weekly preferred
- Service quality per subscriber: average throughput, latency, dropped call/session rate, coverage gaps experienced
- Contract status: plan type, contract start/end date, remaining commitment
- Churn label: binary (churned / retained) per subscriber per period

**Recommended:**
- Handover failure rates and cell-level coverage quality per subscriber's primary cells
- Customer interaction history: service calls, complaints, store visits, chat transcripts
- Billing and payment history: late payments, bill shock events, plan changes
- NPS or CSAT scores if collected
- Competitor offers and porting requests

**Enrichment:**
- Demographics (age band, urban/rural, device type)
- Social network effects (did contacts of this subscriber churn recently?)
- Promotional offer history and redemption rates

### Labels

- Binary churn label per subscriber per observation period (typically monthly)
- Churn rate is typically 1.5-4% per month — heavy class imbalance requires oversampling, SMOTE, or focal loss during training
- Need 6-12 months of history minimum, ideally 18+ months to capture seasonal effects
- Distinguish voluntary churn (ported out) from involuntary (non-payment disconnection)

### Data format and refresh

| Field | Requirement |
|---|---|
| Format | Tabular (CSV/Parquet); one row per subscriber per period |
| Refresh rate | Monthly snapshots for training; weekly for early-warning inference |
| Time range | 12-18 months minimum |
| Anonymization | Subscriber IDs hashed; no PII (name, address, exact location). Aggregate usage and quality metrics are sufficient |

### Operator self-assessment

| Data item | AT&T | Verizon | Turkcell |
|---|---|---|---|
| Per-subscriber usage aggregates | | | |
| Per-subscriber service quality metrics | | | |
| Contract status and tenure | | | |
| Historical churn labels | | | |
| Customer interaction/complaint logs | | | |
| Billing and payment history | | | |
| NPS/CSAT scores | | | |

Fill in: **H** = have, **D** = don't have, **S** = can share, **N** = can't share

---

## 3. Revenue recovery

**ROI target:** 2-5% revenue recovery

### Data tiers

**Minimum viable:**
- CDR/xDR (call detail records, data records): raw usage events with timestamps, subscriber ID, service type, duration/volume, rating result
- Billing engine output: rated charges per CDR, applied tariff plan, discounts
- Invoice data: billed amount vs rated amount per subscriber per cycle

**Recommended:**
- Interconnect and roaming records (TAP/RAP files for international settlements)
- Product catalog: active plans, bundles, promotional offers with effective dates
- Mediation platform logs: records dropped, duplicated, or failed during mediation
- Revenue assurance audit samples: known leakage events with root cause (misrated CDR, unbilled usage, incorrect discount, duplicate charge)

**Enrichment:**
- Partner settlement records and reconciliation reports
- Fraud detection flags (to separate fraud from billing errors)
- Provisioning system logs (to catch order-to-activation gaps)

### Labels

This is the hardest use case to label. Revenue leakage is often discovered through manual audits rather than systematic tagging.
- Need a seed set of labeled leakage events from past revenue assurance audits
- Categories: misrated CDRs, unbilled usage, incorrect discounts, duplicate charges, interconnect settlement errors
- Even 200-500 labeled examples per category can bootstrap a model if combined with rule-based anomaly detection
- The model's primary job may be anomaly scoring rather than classification — flag CDRs that don't match expected patterns for human review

### Data format and refresh

| Field | Requirement |
|---|---|
| Format | CDR format varies by vendor; normalize to a common schema (timestamp, subscriber, service, duration/volume, rated amount, tariff) |
| Refresh rate | Daily CDR feeds for inference; historical batch for training |
| Time range | 6-12 months |
| Anonymization | Hash subscriber IDs; financial amounts can remain as-is since they're not PII by themselves. Tariff plan names may need pseudonymization if they reveal business strategy |

### Operator self-assessment

| Data item | AT&T | Verizon | Turkcell |
|---|---|---|---|
| CDR/xDR with rating results | | | |
| Billing engine output | | | |
| Invoice vs rated reconciliation | | | |
| Interconnect/roaming records | | | |
| Revenue assurance audit samples | | | |
| Mediation platform logs | | | |

Fill in: **H** = have, **D** = don't have, **S** = can share, **N** = can't share

---

## 4. Energy optimization

**ROI target:** 20-30% energy savings

### Data tiers

**Minimum viable:**
- Per-site power consumption: time series at 15-minute or hourly granularity, broken down by radio unit / baseband / cooling if possible
- Traffic load per cell: PRB (Physical Resource Block) utilization, number of connected users, data volume — at matching time granularity
- Cell configuration: carrier frequencies, antenna type, number of sectors, MIMO configuration

**Recommended:**
- Temperature and HVAC readings per site
- Hardware inventory: radio unit model, age, power specs
- Electricity tariff schedules (time-of-use pricing, demand charges)
- Sleep mode and carrier shutdown logs if already implemented (even partially)
- Coverage and service-level constraints: which cells must stay on for emergency services, minimum coverage thresholds

**Enrichment:**
- Solar/renewable energy availability per site
- Predictive traffic models (event calendars, commuter patterns)
- Neighboring cell load (to validate that shutting down one cell doesn't overload neighbors)

### Labels

Energy optimization is one of the more label-friendly use cases because it can use reinforcement learning, where the reward signal is measured power savings vs. maintained service quality. If going supervised:
- Optimal on/off decision per carrier/cell per time slot, validated against traffic thresholds
- Expert-labeled "this cell could have been turned off during this period without SLA impact" from retrospective analysis
- The RL approach needs: action space (carrier on/off, power level, tilt), state (traffic load, time, neighbor load), reward (power saved minus SLA violations)

### Data format and refresh

| Field | Requirement |
|---|---|
| Format | Time series (CSV/Parquet); one row per site per time slot |
| Refresh rate | 15-minute for inference; hourly acceptable for training |
| Time range | 12 months to capture seasonal and day/night patterns |
| Anonymization | Site IDs can be pseudonymized; geographic coordinates can be rounded to 1km grid. Power and traffic data are not PII |

### Operator self-assessment

| Data item | AT&T | Verizon | Turkcell |
|---|---|---|---|
| Per-site power consumption (time series) | | | |
| Traffic load per cell (PRB utilization) | | | |
| Cell configuration and hardware inventory | | | |
| Electricity tariff schedules | | | |
| Existing sleep mode / shutdown logs | | | |
| Coverage constraint definitions | | | |

Fill in: **H** = have, **D** = don't have, **S** = can share, **N** = can't share

---

## 5. Closed-loop autonomous operations (sub-200ms demo)

**ROI target:** TM Forum Autonomy Level 3 (Conditional Autonomy) demonstrated on one use case

### Data tiers

**Minimum viable:**
- Real-time telemetry: OTel-format metrics (latency, error rate, saturation, request rate) from at least one service chain at sub-second resolution
- Kubernetes events and pod health for the CNFs in the chain
- Intent definitions: what "healthy" looks like for this service (SLA thresholds for latency, availability, throughput)
- Action catalog: what remediation actions the system can take (restart pod, scale replicas, reroute traffic, adjust resource limits)

**Recommended:**
- Service mesh telemetry (Istio/Envoy sidecar metrics) for inter-service latency breakdown
- VNF/CNF resource utilization: CPU, memory, network I/O per container
- Historical incident-to-remediation mappings: for a given symptom pattern, what action was taken, did it work, how long did it take
- Escalation policies: when to act autonomously vs when to alert a human

**Enrichment:**
- Distributed traces (full request path through the service chain) for root cause localization
- Chaos engineering results: known failure modes and their signatures from past fault injection
- Dependency graphs: which services depend on which, and failure blast radius

### Labels

- Action labels: for each detected anomaly, what remediation was taken, whether it succeeded, time from detection to resolution
- Need sub-second telemetry resolution to demonstrate the 200ms closed-loop target
- The closed-loop demo is likely built on synthetic telemetry (from the GSMA 5G Sandbox or the Telco-AIX repo) rather than production data, since no operator will put a live network under autonomous control for a demo
- Label generation can be partially automated: inject faults into the sandbox, record the correct remediation, use that as ground truth

### Data format and refresh

| Field | Requirement |
|---|---|
| Format | OTel OTLP (metrics, traces, logs); Kubernetes events as JSON |
| Refresh rate | Sub-second for the demo; 1-second minimum |
| Time range | Can be generated synthetically; 1-2 weeks of simulated operation is enough |
| Anonymization | Synthetic data, so not an issue |

### Operator self-assessment

| Data item | AT&T | Verizon | Turkcell |
|---|---|---|---|
| OTel-format metrics from a service chain | | | |
| Kubernetes events and pod health | | | |
| Intent/SLA definitions for a service | | | |
| Action catalog (what remediation is possible) | | | |
| Historical incident-to-remediation mappings | | | |
| Service dependency graph | | | |

Fill in: **H** = have, **D** = don't have, **S** = can share, **N** = can't share

---

## Cross-cutting notes

**Privacy and data sharing.** Most operators will not share raw production data outside their organization. The self-assessment tables are designed to surface what can be shared in anonymized or synthetic form. For the D5 PoC, the realistic path is:
- Operators fill in the self-assessment to establish what exists
- Shared data is either anonymized, aggregated, or synthetically generated from real distributions
- The GSMA 5G Sandbox and Telco-AIX repo provide a synthetic baseline for use cases 1 and 5

**Format standardization.** Where possible, converge on OpenTelemetry (OTLP) for new instrumentation and Parquet for historical batch data. This aligns with the WG's scope (OpenTelemetry is a cross-cutting technology) and avoids vendor-specific format lock-in.

**Label quality vs. quantity.** For most of these use cases, the bottleneck is labeled data, not raw telemetry. Operators typically have plenty of raw logs and metrics but few systematically labeled incidents. The data requirements document should prompt operators to assess their labeling maturity, not just their data availability.
