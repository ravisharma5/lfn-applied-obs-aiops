# LFN Applied Observability for AIOps (A-O/AI) Working Group

A Linux Foundation Networking (LFN) working group tackling fragmented telco observability by building shared best practices for AI-driven operations, moving networks from static silos toward a living, self-healing state. The guide covers the full telco stack (RAN through BSS, edge, NTN) and gets validated through a simulated proof of concept by Q4 2026.

Working group wiki: [Applied Observability for AIOps](https://lf-networking.atlassian.net/wiki/spaces/LN/pages/1181417473/Applied+Observability+for+AIOps)

## Participants

People from AT&T, Verizon, Turkcell, AWS, Red Hat, and LFN.

| Name | Org | Role |
|---|---|---|
| Ravi Sharma | Red Hat | Technical Lead, PoC & guide author |
| Fatih E. Nar | Red Hat | Vice Chair |
| Tony Hansen | AT&T | Security & Observability TAC Lead |
| Murat Parlakisik | AWS | AI/ML Architecture Lead |
| Ranny Haiby | LFN | CTO / Governance Sponsor |
| TBD | Turkcell | Operator Rep |
| Sunny Singh | Verizon | Operator Rep |

## Four-step AIOps method

1. **Find the data** - map silos using OpenTelemetry/eBPF instrumentation
2. **Apply domain expertise** - CRISP-DM-style feature selection for use cases
3. **Place models on the grid** - size and place models (4B-405B) by latency tier
4. **Govern the agents** - use MCP/A2A protocols for agent oversight

## North-star KPIs

- TM Forum Autonomy L3 (Conditional Autonomy) demonstrated on one use case
- Sub-200ms closed-loop demo
- Reference ROI targets: 40% fewer outages (fault prediction), 15-25% churn reduction, 2-5% revenue recovery, 20-30% energy savings

## Scope

**In scope:** RAN, Transport, 5G/6G Core, OSS, BSS, Edge/MEC, NTN, Cloud Infra. Cross-cutting tech includes OpenTelemetry, eBPF, vLLM, MCP/A2A, Kubernetes platforms, and zero-trust security.

**Out of scope:** vendor-specific tools, production deployments, formal standardization. Voice services (IMS/VoLTE/VoNR) are deferred to a future phase.

## Deliverables

| # | Deliverable | Timeline | Status |
|---|---|---|---|
| D1 | Charter & Governance | May 2026 | Done: [`LFN_AO-AI_WG_Charter_v1_1.docx`](./LFN_AO-AI_WG_Charter_v1_1.docx) |
| D2 | Landscape Assessment - per-operator workbooks from AT&T, Verizon, Turkcell | June 2026 | In progress: [`D2_Operator_Template.md`](./D2_Operator_Template.md) |
| D3 | Reference Architecture - 5-layer stack, 4-tier AI Grid, governance guardrails, sub-200ms closed-loop target | July 2026 | Not started |
| D4 | Use Case Catalog - five anchor use cases with ROI targets | July-Aug 2026 | Not started |
| D5 | Simulated PoC - built on the AWS/LFN AI TAC open-source 6G sandbox, using the Telco-AIX repo's pre-built AI experiments | Aug-Sep 2026 | Not started |
| D6 | Best Practice Guide - published under Apache 2.0 | Sep-Oct 2026 | Not started |
| D7 | Final Demo & LFN Presentation | Nov-Dec 2026 | Not started |

Five phases run May-December 2026: Foundation, Architecture/Design, Validation, Publication, Delivery.

## Related resources

- [Working group wiki](https://lf-networking.atlassian.net/wiki/spaces/LN/pages/1181417473/Applied+Observability+for+AIOps) - includes linked "AI Observability Minutes" and "AI Observability Members" subpages
- [github.com/Open-Experiments](https://github.com/Open-Experiments)
- [github.com/open-experiments/Telco-AIX](https://github.com/open-experiments/Telco-AIX): 26+ pre-built AI experiments
