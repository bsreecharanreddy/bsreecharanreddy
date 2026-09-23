# bsreecharanreddy

**Senior Inference & AI/ML Platform Engineer** · Atlanta, GA ·
[LinkedIn](https://www.linkedin.com/in/sreecharanreddybonthu)

11+ years building systems that can't afford to be wrong — now focused on
the infrastructure AI actually runs on, from the GPU kernel that serves a
model up through the feature pipelines that trained it: fast, correct
inference at the dispatch layer, reproducible training data, and bounded,
auditable model behaviour rather than black boxes making binding decisions
on their own.

## Day job

**AI/ML Platform Engineer at Deloitte** on Georgia Gateway, a regulated
eligibility and benefits system serving about 1 in 3 Georgians. I take
agentic AI from prototype to production: tool-calling LLM agents for
document classification and pre-screening, RAG over policy manuals behind a
caseworker copilot, LLM anomaly scoring in fraud/waste/abuse detection, and
the evaluation, guardrail and LLMOps layer that keeps all of it measurable —
golden test sets, eval regression checks in CI, PII redaction, tracing, and
cost and latency monitoring.

Before that: solution architect for the eligibility and benefits platform,
and nine years building it — Java, Spring, Oracle Policy Automation,
Azure and Databricks — including the Enterprise Master Person Index and the
P-EBT pandemic program that delivered benefits to 1.1M+ Georgians.

The projects below are built solo, on my own time, with public or synthetic
data.

## [Dispatch](https://github.com/bsreecharanreddy/dispatch) — inference engine for MoE token routing

Mixture-of-Experts models (DeepSeek, Llama, Mixtral, Grok, Qwen) get a
model's full capacity at a fraction of the compute by routing each token to
a handful of expert sub-networks out of many — the hard part is making
that routing fast under real, skewed per-expert load, and serving it like
production infrastructure once it works. A from-scratch Triton kernel, real
multi-GPU expert-parallel serving, a head-to-head benchmark against vLLM
and SGLang, and a Rust router in front of a real GPU-backed model server,
deployed to Kubernetes and demoed against a rented GPU — all measured, cost
included. **All 8 phases complete —
[`v1.0.0`](https://github.com/bsreecharanreddy/dispatch/releases/tag/v1.0.0),
with a git tag per phase** for anyone who wants the diff between any two
points in the build.

- **Custom Triton grouped-GEMM kernel**, proven numerically correct against
  a reference implementation on real hardware before any speed claim —
  then measured **~65-73% faster** decode throughput than DeepSeek's own
  stock MoE forward pass, at perfect logit agreement
- **The head-to-head this project was built toward:** raced dispatch's own
  kernel against vLLM 0.29.0's and SGLang 0.5.20's production fused-MoE on
  the same GPU, same model, same Triton compiler — **vLLM wins at every
  measured shape**, reported as the loss it is, not reframed. The at-scale
  correctness gate that came first (97.1-97.3% top-1 agreement at 1,036
  positions) is what makes that comparison trustworthy
- **A production serving path, not just kernels:** a Rust router (tonic
  gRPC, axum HTTP, Prometheus metrics) in front of the Python model
  server, containerized, deployed to a local Kubernetes cluster, and
  demonstrated against a real rented GPU — that session found and fixed a
  genuine defect in the model's own shipped tokenizer (wired to the wrong
  vocabulary format, silently dropping every space in generated text),
  root-caused to the exact missing data
- **Real multi-GPU expert-parallel serving** over DeepSeek's DeepEP
  library, byte-exact against a single-GPU reference — which also
  *disproved* the project's own kernel-crossover hypothesis at real
  scale, reported as the null result it was
- **Self-computed int8 weight-only quantization**, extending the kernel
  itself rather than calling a quantization library — **49.89%**
  expert-weight memory reduction at perfect model-level agreement. A real
  bug (quantizing without freeing the original weights, doubling memory
  instead of halving it) found live on rented hardware and fixed same-day
- **Open-source contribution, under review:** an opt-in skewed-load
  benchmark flag proposed to vLLM, after finding that neither vLLM's nor
  SGLang's official MoE benchmarks modeled skewed expert load —
  [vllm-project/vllm#57100](https://github.com/vllm-project/vllm/pull/57100)
- **Cost discipline:** eleven rented-GPU sessions, **$51.35 total**, every
  phase within its stated cap except two, both overruns disclosed rather
  than hidden
- **277 tests** (268 Python, 9 Rust), **86% coverage** on CPU-testable
  logic, `make check` green throughout — GPU-only kernel code is covered
  by its own paid, hardware-run correctness suite

Full write-ups, including every bug found on real hardware and every null
or mixed result:
[`docs/findings/`](https://github.com/bsreecharanreddy/dispatch/tree/main/docs/findings).

## [Almanac](https://github.com/bsreecharanreddy/almanac) — ML platform for work-queue risk

Work items arrive in a queue, some breach their service expectation, and a
model predicts which ones early enough for a human to intervene. Built on
GitHub's public event firehose — real, large, genuinely messy, and carrying
**three** schema eras, including a late-2025 payload reduction that removes
fields the model depends on. The domain is deliberately incidental — the
same architecture serves a support-ticket queue, a claims backlog, or a
fraud review queue.

**Live demo, no setup required: [almanac-live.streamlit.app](https://almanac-live.streamlit.app/).**
**341,060,851 events** ingested across a real schema break for a measured
**$11.96**. A point-in-time-correct feature store, a model at **0.4661
PR-AUC** against a 0.2650 baseline, a live serving endpoint measured at
**p50 263.5 ms** warm and **51.96 s** cold, and three AI/BI dashboards
defined as code. 88% coverage on transformation logic, gated at 85% in CI.
Clone to a green run in **4 m 28 s**.

- **Point-in-time correctness** — every feature computed for an item at
  time T uses only data that existed before T. This is where label leakage
  lives, and it is the reason the project exists
- **The project found a leakage bug in its own registered champion.** The
  train/test split was random where the design requires temporal. Fixed,
  re-scored, and the inflated number retracted in public: **0.612 →
  0.4661**, optimistic by 24%. The leakage suite was green throughout and
  was not wrong — it tested one axis, and the bug was on another
- **A baseline shipped before the model, and the first result was a null
  one** — LightGBM lost to a per-segment median by 51% on the regression
  target. Published, not buried, then reframed to classification
- **Data platform:** PySpark medallion on Delta Lake through one
  config-driven framework; SCD Type 2 dimensions, an accumulating-snapshot
  fact, and bad records quarantined by rule rather than dropped
- **ML lifecycle:** MLflow tracking and registry, scale-to-zero serving,
  drift measured and training/serving skew verified at **100.00%
  agreement** across 3,459 keys rather than assumed absent
- **Streaming:** Spark Structured Streaming — watermarks, late arrival,
  exactly-once, and a real watermark bug that silently dropped 18% of a
  live window, written up as a postmortem
- **Governance:** Unity Catalog, column-level lineage published with its
  own blind spots stated on it, and data contracts enforced as CI failures
  rather than written down as documents
- **Agent layer, read-only:** four MCP tools behind a tool gateway
  (allow-list, append-only audit log) and a model gateway (capability
  records, one explicit fallback), fronted by a bounded agent capped at
  six model requests per run
- **Grounding verification, no LLM judge:** every number in an agent's
  answer is traced to the tool call that produced it, and every claim
  *about* that number is traced to the field it actually means. Built
  after my own agent produced a real false claim under a green "every
  number came from a tool" rule; the fix was a schema rename plus a
  relationship check, not a stricter number check

What it does *not* do is written down as plainly as what it does —
including the numbers that were later found wrong and corrected in place.

## [Canopica](https://github.com/bsreecharanreddy/canopica) — deterministic, auditable decisions

The kind of system where every dollar amount has to be explainable and
reproducible, with an AI capability layer built on top of that core.

**Live demo, no setup required: [canopica-policy-demo.fly.dev](https://canopica-policy-demo.fly.dev/)**
(may take a moment to wake — it scales to zero between uses to control
compute cost).

- **Core system:** Spring Boot API + a Drools DMN rules engine, with a
  hash-chained, tamper-evident audit log behind every decision
- **UI:** React + TypeScript
- **Data platform:** Python-orchestrated bronze/silver/gold medallion
  pipeline — Airflow, dbt, Postgres
- **AI layer:** eight governed capabilities running in the demo — Policy
  Q&A, rule-authoring, Analytics Copilot, dashboard-authoring, an SOP
  copilot, risk triage, a QC/payment-error assistant, and an SLA/compliance
  monitor — each built as a fixed, evaluable pipeline (retrieval →
  generation → schema-validated output → eval-gated in CI), never an
  open-ended agent making the call itself
- **Real cloud deployments:** Databricks and Azure via Terraform, applied
  for real and verified live — not just `terraform validate` in CI

Uses a synthetic benefits-eligibility domain as the vehicle for exercising
that architecture end to end — the engineering pattern is the point, the
domain is the example.

## What I'm looking for

Senior/staff roles in either of two shapes: building the inference and
serving layer AI runs on, or working inside a customer's messy production
stack to make a model behave in it. Different titles, same underlying
skill.

**Stack:** Python, Java, Rust, SQL · PyTorch, Triton · PySpark, Databricks,
Delta Lake, MLflow · LLM agents, RAG, MCP, evals · Kubernetes, Docker,
Terraform · Azure, AWS · Kafka, Airflow, dbt
