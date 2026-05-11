# 🔍 Junior Job Market Intelligence — Data Warehouse & BI System

> **Which fields actually hire junior talent?**  
> An end-to-end data engineering project analyzing the junior job market across 3 heterogeneous sources, powered by LLM enrichment and Elasticsearch analytics.

**Academic project — ISIMS Sfax | 2025–2026**  
Team: Salma Bensaida · Hadir Mejri · Yosr Allouch  
Supervised by: Mme Feiza Ghozzi

---

## 📌 Problem Statement

New graduates face a structural information gap: they don't know which domains genuinely hire junior profiles vs. which ones only post senior roles. This project builds a decisional data warehouse to answer that question with real data.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Data Sources                         │
│   Adzuna API (real-time)  │  Kaggle CSV  │  Hugging Face    │
└──────────────┬────────────┴──────┬───────┴────────┬─────────┘
               │                   │                │
        ┌──────▼──────┐    ┌───────▼──────┐         │
        │  n8n Pipeline│    │  Apache Hop  │◄────────┘
        │  (API source)│    │ (static files│
        └──────┬───────┘    └──────┬───────┘
               │                   │
        ┌──────▼───────────────────▼───────┐
        │     LLM Enrichment Layer          │
        │  Ollama (local) · Groq Cloud      │
        │  → experience_level, skills,      │
        │    certifications, company_size   │
        └──────────────┬────────────────────┘
                       │
        ┌──────────────▼────────────────────┐
        │        Elasticsearch              │
        │   Denormalized document model     │
        │   Dual text/keyword mapping       │
        │   geo_point · nested aggregations │
        └──────────────┬────────────────────┘
                       │
        ┌──────────────▼────────────────────┐
        │         Kibana Dashboards         │
        │   8 analytical views (B01–B08)    │
        └───────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Tools |
|-------|-------|
| ETL — API sources | n8n |
| ETL — Static files(csv)| Apache Hop |
| LLM Enrichment | Ollama (local) · Groq Cloud |
| Data Warehouse | Elasticsearch |
| Visualization | Kibana |
| Data Sources | Adzuna API · Kaggle · Hugging Face |

---

## 📊 Data Sources

| Source | Type | Volume | Key fields |
|--------|------|--------|-----------|
| Adzuna API | Real-time REST | Dynamic | title, salary, location, category |
| Kaggle dataset | Static CSV | ~350k rows | title, description, company, location |
| Hugging Face dataset | Static JSON | ~17k rows | title, description, skills, category |

---

## ⚙️ ETL Pipelines

### n8n — Adzuna API Pipeline
- Paginated API calls with rate-limit handling
- 3 parallel LLM flows (Ollama) for experience level, skills, and company size extraction
- Merge node (Combine by Position) to reunify enriched fields before indexing
- Direct push to Elasticsearch index

→ Export: [`n8n/adzuna_pipeline.json`](n8n/adzuna_pipeline.json)

### Apache Hop — Kaggle & Hugging Face Pipelines
- CSV/JSON ingestion with field normalization
- Semantic category unification across sources → 11-category unified taxonomy
- LLM enrichment via Groq Cloud for missing structured fields
- Traceable extraction method field (`*_extraction_method`) to distinguish native vs. inferred values

→ Pipelines: [`hop/kaggle_pipeline.hwf`](hop/kaggle_pipeline.hwf) · [`hop/huggingface_pipeline.hwf`](hop/huggingface_pipeline.hwf)

---

## 🧠 LLM Enrichment Strategy

Raw job postings from all three sources lack structured fields for `experience_level`, `skills`, and `certifications`. We extract these from free-text descriptions using LLMs:

| Source | Model | Reason |
|--------|-------|--------|
| Adzuna | Ollama (local) | High call frequency, data privacy |
| Hugging Face | Groq Cloud | Lower volume, faster inference |

A fallback mechanism handles malformed LLM responses to ensure pipeline stability.

---

## 🗂️ Elasticsearch Schema

- **Model**: Fully denormalized documents (no joins needed at query time)
- **Mapping strategy**: Dual `text`/`keyword` for aggregatable string fields, `geo_point` for geographic mapping, numeric types for salary measures
- **Query optimization**: `filter context` + `term` on keyword fields enables automatic filter caching, dramatically reducing Kibana dashboard refresh latency

→ Schema: [`elasticsearch/index_mapping.json`](elasticsearch/index_mapping.json)

---

## 📈 Kibana Dashboards

Eight analytical views built to answer the core business needs:

| ID | View | Description |
|----|------|-------------|
| B01 | Junior offer volume by domain | Top fields hiring junior profiles |
| B02 | Junior/total ratio by domain | Which fields are genuinely junior-friendly |
| B03 | Average junior salary by domain | Compensation benchmarks |
| B04 | Geographic distribution | Where junior jobs are concentrated |
| B05 | Temporal evolution | Hiring trends over time |
| B06 | Most demanded qualifications | Skills employers actually ask for |
| B07 | Experience level breakdown | Junior vs mid vs senior share per domain |
| B08 | Company size distribution | Startup vs. corporate hiring patterns |

→ Export: [`kibana/dashboards.ndjson`](kibana/dashboards.ndjson)

---

## 🔑 Key Findings

- **IT, Engineering, and Healthcare** consistently show the highest volume of junior-tagged offers across all three sources
- **LLM enrichment** recovered structured fields for ~85% of offers that had no explicit experience level in the raw data
- **Semantic normalization** across 3 incompatible taxonomies was the most complex challenge — solved via LLM-assisted mapping to a unified 11-category system

---

## 📁 Repository Structure

```
├── n8n/
│   └── adzuna_pipeline.json          # n8n workflow export
├── hop/
│   ├── kaggle_pipeline.hwf           # Apache Hop pipeline
│   └── huggingface_pipeline.hwf      # Apache Hop pipeline
├── elasticsearch/
│   └── index_mapping.json            # Index mapping + settings
├── kibana/
│   └── dashboards.ndjson             # Dashboard export
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- Elasticsearch 8.x + Kibana (local or Elastic Cloud)
- n8n (self-hosted recommended)
- Apache Hop
- Ollama with a local model (e.g. `llama3`)
- Groq Cloud API key (free tier sufficient)

### Setup
```bash
# 1. Create Elasticsearch index
curl -X PUT "localhost:9200/job_offers" \
  -H "Content-Type: application/json" \
  -d @elasticsearch/index_mapping.json

# 2. Import Kibana dashboards
# Kibana → Stack Management → Saved Objects → Import dashboards.ndjson

# 3. Import n8n workflow
# n8n → Workflows → Import from file → adzuna_pipeline.json

# 4. Open Apache Hop and load pipelines from /hop directory
```

---

## 📄 License

Academic project — ISIMS Sfax. Not licensed for commercial use.
