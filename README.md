# Sree

Senior software/AI engineer, 11+ years across backend systems, data
platforms, and cloud — currently focused on the infrastructure AI actually
runs on: correct feature pipelines, reproducible training data, and
bounded, auditable model behaviour rather than black boxes making binding
decisions on their own.

## [Almanac](https://github.com/bsreecharanreddy/almanac) — ML platform for work-queue risk

Work items arrive in a queue, some breach their service expectation, and a
model predicts which ones early enough for a human to intervene. Built solo
on GitHub's public event firehose — real, large, genuinely messy, and
carrying **three** schema eras, including a payload reduction in late 2025
that removes fields the model depends on.

The domain is deliberately incidental. The same architecture serves a
support-ticket queue, a claims backlog, or a fraud review queue.

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
  answer is traced to the tool call that produced it, and — the harder
  check — every claim *about* that number is traced to the specific field
  it actually means. Built after my own agent produced a real false claim
  under a green "every number came from a tool" rule; the fix was a
  schema rename plus a relationship check, not a stricter number check

What it does *not* do is written down as plainly as what it does —
including the numbers that were later found wrong and corrected in place.

## [Canopica](https://github.com/bsreecharanreddy/canopica) — deterministic, auditable decisions

The kind of system where every dollar amount has to be explainable and
reproducible, with an AI capability layer built on top of that core. Built
solo, start to finish.

- **Core system:** Spring Boot API + a Drools DMN rules engine, with a
  hash-chained, tamper-evident audit log behind every decision
- **UI:** React + TypeScript
- **Data platform:** Python-orchestrated bronze/silver/gold medallion
  pipeline — Airflow, dbt, Postgres
- **AI layer:** RAG-based copilots built as fixed, evaluable pipelines
  (retrieval → generation → schema-validated output → eval-gated in CI),
  never an open-ended agent making the call itself
- **Real cloud deployments:** Databricks and Azure via Terraform, applied
  for real and verified live — not just `terraform validate` in CI

Applied to benefits eligibility as the vehicle for exercising that
architecture end to end — the engineering pattern is the point, the
domain is the example.

## [Dispatch](https://github.com/bsreecharanreddy/dispatch) — inference engine for MoE token routing

Mixture-of-Experts models (DeepSeek, Llama, Mixtral, Grok, Qwen) get a
model's full capacity at a fraction of the compute by routing each token to
a handful of expert sub-networks out of many — the hard part is making
that routing fast under real, skewed per-expert load. Built solo: a
from-scratch Triton kernel, real multi-GPU expert-parallel serving, a
disaggregated prefill/decode scheduler, and an upstreamed open-source
benchmark contribution, all measured on rented GPU hardware, cost included.

- **Custom Triton grouped-GEMM kernel**, proven numerically correct against
  a reference implementation on real hardware before any speed claim —
  then measured **~65-67% faster** decode throughput than DeepSeek's own
  stock MoE forward pass, at perfect logit agreement
- **Real multi-GPU expert-parallel serving** over DeepSeek's own DeepEP
  library, byte-exact against a single-GPU reference — which also
  *disproved* this project's own kernel-crossover hypothesis at real
  scale, reported as the null result it was rather than smoothed over
- **Disaggregated prefill/decode** across 4 real GPUs, correctness-gated on
  both topologies, with a genuinely mixed throughput result reported
  honestly instead of forced into a win
- **An open-source contribution:** an opt-in skewed-load benchmark flag
  upstreamed to vLLM after finding neither vLLM's nor SGLang's official MoE
  benchmarks modeled it —
  [vllm-project/vllm#57100](https://github.com/vllm-project/vllm/pull/57100)
- **Cost discipline:** five rented-GPU sessions, **$25.74 total**, every
  one under its stated cap, every number measured rather than estimated

Full write-ups, including every bug found on real hardware and every null
or mixed result:
[`docs/findings/`](https://github.com/bsreecharanreddy/dispatch/tree/main/docs/findings).

## Background

Production engineering across Java, Python, SQL, PySpark, data pipelines,
cloud infrastructure, message queues, microservices, and security —
currently aiming at senior/staff engineering and architecture roles in
AI/ML platform work.
