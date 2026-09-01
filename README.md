# Sree

Senior software/AI engineer, 10+ years across backend systems, data
platforms, and cloud — currently focused on the infrastructure AI actually
runs on: correct feature pipelines, reproducible training data, and
bounded, auditable model behaviour rather than black boxes making binding
decisions on their own.

## Currently building: Almanac

An **ML platform for work-queue risk** — work items arrive in a queue,
some breach their service expectation, and a model predicts which ones
early enough for a human to intervene. Built on GitHub's public event
firehose, which is real, large, genuinely messy, and carries a real schema
break in 2015.

The domain is deliberately incidental. The same architecture serves a
support-ticket queue, a claims backlog, or a fraud review queue.

- **Point-in-time-correct feature store** — every feature computed for an
  item at time T uses only data that existed before T. This is where label
  leakage lives, and it's the reason the project exists
- **Data platform:** PySpark medallion (bronze/silver/gold) on Delta Lake,
  handling two schema eras through one config-driven framework; SCD Type 2
  dimensions and an accumulating-snapshot fact
- **ML lifecycle:** MLflow tracking and registry, a scale-to-zero serving
  endpoint, with drift and training/serving skew monitored rather than
  assumed absent
- **Streaming:** Spark Structured Streaming — watermarks, late arrival,
  exactly-once
- **Governance:** Unity Catalog, column-level lineage, and data contracts
  enforced as CI failures rather than written down as documents

Currently in early implementation, and private until a model serves —
design and phasing are complete, with every assumption measured against
real data rather than assumed. Public here when it's worth reading.

## Shipped: [Canopica](https://github.com/bsreecharanreddy/canopica)

A deterministic, auditable decision system — the kind where every dollar
amount has to be explainable and reproducible — with an AI capability
layer built on top of that core. Built solo, start to finish.

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

## Background

Production engineering across Java, Python, SQL, PySpark, data pipelines,
cloud infrastructure, message queues, microservices, and security —
currently aiming at senior/staff engineering and architecture roles in
AI/ML platform work.
