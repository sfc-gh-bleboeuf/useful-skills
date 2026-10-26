---
name: cost-estimator
description: Estimate Snowflake costs for customer use cases. Use when: sizing, pricing estimate, cost calculator, estimate credits, TCO, consumption forecast, how much will it cost. Triggers: estimate cost, cost estimate, sizing exercise, credit estimate, consumption estimate, pricing calculator.
---

# Snowflake Cost Estimator

Estimate Snowflake credit and dollar costs for customer use cases. Collects use case details, maps to cost categories, calculates credits, and produces a summary table.

> **Source:** [Snowflake Service Consumption Table](https://www.snowflake.com/legal-files/CreditConsumptionTable.pdf) — **effective August 26, 2026**. Always recommend customers verify against the latest published table; rates change roughly monthly.

---

## Two Credit Currencies — Read This First

Snowflake bills in **two separate purchasable currencies**. Getting this wrong is the single largest source of estimation error.

| Currency | Rate | What it covers |
|----------|------|----------------|
| **Platform Credits** | $2.00–$9.75/credit (varies by edition + cloud + region) | Virtual warehouses, SPCS Compute, Adaptive Compute, Openflow Compute, Postgres Compute, Cloud Services, all Serverless Features (Snowpipe, tasks, clustering, SOS, MVs, replication), Cortex Analyst API, Cortex Fine-Tuning *training*, Provisioned Throughput, Organization Usage views |
| **AI Credits** | **$2.00/credit Global, $2.20/credit Regional** (On Demand) — flat, **edition-independent** | Essentially **all** Cortex AI inference: AI_COMPLETE and every other `AI_*` function, AI_EMBED, AI_EXTRACT, AI_PARSE_DOCUMENT, AI_CLASSIFY, Cortex Search, Cortex Agents, Snowflake CoWork (Snowflake Intelligence), Cortex Code (CoCo), fine-tuned model *inference* |

**Critical notes:**
- The term "Snowflake Credits" in older contracts/docs now means **Platform Credits**.
- Because AI Credits are edition-independent, Enterprise/BC/VPS customers effectively save 33–67% on **all** AI workloads vs. the old edition-priced model. Standard-edition US customers see no change.
- Cortex Analyst is the notable exception: **still Platform Credits** when called via the Analyst API.

### AI Credit Pricing — On Demand

| Type | On Demand |
|------|-----------|
| Global | $2.00 |
| Regional | $2.20 |

These are published On Demand rates. For contracted pricing, use the rates on the customer's Order Form. **Default to $2.00/AI Credit** unless told otherwise.

------|-----------|--------|--------|--------|--------|--------|--------|--------|
| Global | $2.00 | $2.00 | $1.96 | $1.96 | $1.94 | $1.92 | $1.90 | $1.88 |
| Regional | $2.20 | $2.20 | $2.16 | $2.16 | $2.13 | $2.11 | $2.09 | $2.07 |

Tiers are set by annualized contract value on the Order Form. Cortex Inference may not be eligible for Capacity AI Credit pricing at all — check the Order Form. **Default to $2.00/AI Credit** unless told otherwise.

---

## Platform Credit Pricing (On Demand, per credit)

| Edition | US (AWS/Azure/GCP core regions) | Typical EU | Typical APAC | Highest (Dammam / Zurich / São Paulo) |
|---------|------|------|------|------|
| Standard | $2.00 | $2.60–2.70 | $2.40–2.85 | $3.10–3.25 |
| Enterprise | $3.00 | $3.90–4.00 | $3.60–4.30 | $4.65–4.90 |
| Business Critical | $4.00 | $5.20–5.40 | $4.80–5.70 | $6.20–6.50 |
| VPS | $6.00 | $7.80–8.10 | $7.20–8.55 | $9.30–9.75 |

US Gov / FedRAMP regions: Business Critical $4.80–5.60, VPS $7.20–8.40 (no Standard/Enterprise).

**Default for estimates: $3.00/Platform Credit (Enterprise, US AWS).** For contracted pricing, use the rates on the customer's Order Form.

---

## Compute Pricing

### Table 1(a): Standard Virtual Warehouse — Platform Credits/hr

| XS | S | M | L | XL | 2XL | 3XL | 4XL | 5XL | 6XL |
|----|---|---|---|----|-----|-----|-----|-----|-----|
| 1 | 2 | 4 | 8 | 16 | 32 | 64 | 128 | 256 | 512 |

5XL/6XL may not be GA in all regions. Minimum 1 minute on resume, then per-second billing.

### Table 1(b): Gen 2 Warehouse — Platform Credits/hr

| Cloud | XS | S | M | L | XL | 2XL | 3XL | 4XL |
|-------|----|---|---|---|----|-----|-----|-----|
| AWS | 1.35 | 2.7 | 5.4 | 10.8 | 21.6 | 43.2 | 86.4 | 172.8 |
| Azure | 1.25 | 2.5 | 5 | 10 | 20 | 40 | 80 | 160 |
| GCP | 1.35 | 2.7 | 5.4 | 10.8 | 21.6 | 43.2 | 86.4 | 172.8 |

1.35× (AWS/GCP) or 1.25× (Azure) the Gen1 rate; runs ~28–47% faster on benchmarks. Net win only if the speedup exceeds the premium — test on the real workload. Max size 4XL.

### Table 1(d): Interactive Warehouse — Platform Credits/hr

| XS | S | M | L | XL | 2XL | 3XL | 4XL |
|----|---|---|---|---|-----|-----|-----|
| 0.6 | 1.2 | 2.4 | 4.8 | 9.6 | 19.2 | 38.4 | 76.8 |

**0.6× the standard rate, but a 60-minute minimum charge on every start/resume.** Cheap for sustained high-concurrency serving; expensive for spiky, intermittent use.

### Table 1(c): Snowpark-Optimized Warehouse — Platform Credits/hr

| Constraint | XS | S | M | L | XL | 2XL | 3XL | 4XL | 5XL | 6XL |
|-----------|----|---|---|---|----|-----|-----|-----|-----|-----|
| MEMORY_1X | 1.00 | 2.00 | 4.00 | 8.00 | 16.00 | 32.00 | 64.00 | 128.00 | – | – |
| MEMORY_1X_x86 | 1.10 | 2.20 | 4.40 | 8.80 | 17.60 | 35.20 | 70.40 | 140.80 | – | – |
| MEMORY_16X | – | – | 6.00 | 12.00 | 24.00 | 48.00 | 96.00 | 192.00 | 384.00 | 768.00 |
| MEMORY_16X_x86 | – | – | 6.25 | 12.50 | 25.00 | 50.00 | 100.00 | 200.00 | – | – |
| MEMORY_64X | – | – | – | 15.00 | 30.00 | 60.00 | 120.00 | 240.00 | – | – |
| MEMORY_64X_x86 | – | – | – | 16.00 | 32.00 | 64.00 | 128.00 | 256.00 | – | – |

### Adaptive Warehouse (Table 1(e), Preview)

No fixed rate. Platform Credit consumption scales with compute usage, software optimizations, and active queries, governed by customer-provided settings. **Do not quote a hard number** — model as an equivalent standard warehouse and flag the uncertainty.

### SPCS Compute (Table 1(g)) — Platform Credits/hr, selected

| Instance Family | AWS | Azure | GCP |
|----------------|-----|-------|-----|
| GEN_ARM_G1_2 / _4 / _8 | 0.057 / 0.114 / 0.228 | N/A | N/A |
| GEN_ARM_G1_16 / _32 | 0.456 / 0.912 | N/A | N/A |
| GEN_X64_G2_2 / _4 / _8 | 0.064 / 0.128 / 0.256 | 0.064 / 0.128 / 0.256 | N/A |
| GEN_X64_G2_32 | 1.024 | 1.024 | N/A |
| MEM_X64_G2_8 / _32 / _64 | 0.336 / 1.344 / 2.688 | 0.287 / 1.148 / 2.296 | N/A |
| GPU_L4_G1_4 / _16 | 0.511 / 0.840 | N/A | N/A |
| GPU_L40S_G1_8 / _48 | 1.424 / 6.663 | N/A | N/A |
| GPU_R6K_G1_8 / _48 | 2.136 / 5.262 | N/A | 2.857 (48) |
| GPU_A100_G1_12 / _48 | N/A | N/A | 2.333 / 9.330 |

Compute Nodes bill a **5-minute minimum** on start/resume (Postgres Compute: 1 minute), then per-second.

### Openflow (Table 1(h))
- **BYOC deployment:** 0.0225 Platform Credits per vCPU per hour
- **Snowflake (SPCS) deployment:** uses First-Gen SPCS instance rates (CPU_X64_S 0.11, CPU_X64_SL 0.41, CPU_X64_L 0.83 credits/hr), 60-second minimum
- **Openflow Connector for Oracle** is separately licensed (Table 7): $70/month per Licensed Core (36-mo) or $2,520 (12-mo), plus S&M $40/month or $600/year. Licensed Core = core × Oracle Core Processor Licensing Factor. **This is a real line item — don't forget it on Oracle deals.**

### Postgres Compute (Table 1(i), Preview) — Platform Credits/hr, selected

| Instance | AWS | Azure | AWS HA | Azure HA |
|----------|-----|-------|--------|----------|
| BURST_XS / BURST_S / BURST_M | 0.0068 / 0.0136 / 0.0272 | – / 0.0144 / 0.0288 | 0.0136 / 0.0272 / 0.0544 | – / 0.0288 / 0.0576 |
| STANDARD_M / L / XL / 2X | 0.0356 / 0.0712 / 0.1424 / 0.2848 | 0.0376 / 0.0752 / 0.1504 / 0.3008 | 2× the non-HA rate | 2× the non-HA rate |
| STANDARD_8XL / 24XL | 1.1392 / 3.4176 | 1.2032 / 3.6096 | 2.2784 / 6.8352 | 2.4064 / 7.2192 |
| HIGHMEM_L / 4XL / 48XL | 0.1024 / 0.8192 / 9.8304 | 0.1088 / 0.8704 / 10.4448 | 0.2048 / 1.6384 / 19.6608 | 0.2176 / 1.7408 / 20.8896 |

High Availability is exactly 2× the standard rate.

### Cloud Services
- **4.4 Platform Credits per hour** of Cloud Services use.
- **Cloud Services Adjustment:** free if daily Cloud Services credits ≤ 10% of daily Virtual Warehouse credits. Most well-behaved workloads pay $0.
- Cloud Services from Serverless Features and SPCS Compute **do not** count toward the 10% adjustment — they are always billed.
- Typical modeling: assume $0, but flag risk for metadata-heavy patterns (thousands of tiny queries, heavy `SHOW`/`INFORMATION_SCHEMA` use, many small DDL ops).

---

## Serverless Features (Table 5) — Platform Credits

Serverless compute bills at **1 Platform Credit per Compute-Hour × feature multiplier**, where 1 Compute-Hour = one XS warehouse for one hour.

| Feature | Compute Multiplier | Cloud Svcs Multiplier | Unit Charges |
|---------|-------------------|----------------------|--------------|
| Snowpipe | – | – | **0.0037 Platform Credits/GB** |
| Snowpipe Streaming (high-perf arch) | – | – | **0.0037 Platform Credits per uncompressed GB** |
| Snowpipe Streaming Classic | 1 | – | 0.01 Platform Credits per client instance per hour |
| Serverless Tasks | 0.9 | 1 | – |
| Serverless Tasks Flex | **0.5** | 1 | – |
| Serverless Alerts | 0.9 | 1 | – |
| Materialized Views maintenance | 2 | 1 | – |
| Clustering Classic | 2 | 1 | – |
| Optima Clustering | – | – | 0.007 Platform Credits per uncompressed GB |
| Search Optimization Service | 2 | 1 | – |
| Query Acceleration | 1 | – | – |
| Data Quality Monitoring | 2 | 1 | – (ROW_COUNT DMF uses 0.9) |
| Sensitive Data Classification | 0.9 | 1 | – |
| AI Sensitive Data Classification | – | – | 0.15 AI Credits/1M input + 1.2 AI Credits/1M output tokens |
| Replication Classic | 2 | 0.35 | – |
| Replication – Optimized Refresh | – | – | 5 Platform Credits/TB replicated; 0.2 per 10K changed objects >25M |
| Backup | 2 | 1 | – |
| Failsafe Recovery | 0.9 | 1 | – |
| Table Optimization | 0.75 | 1 | – |
| Storage Lifecycle Policy Execution | 0.50 | 1 | – |
| Automated Refresh & Data Registration | 1.25 | – | 0.06 Platform Credits/1000 files (External/Directory tables only, not Iceberg) |
| Archive Storage Retrieval / Write | – | – | 0.05 Platform Credits/1000 files each |
| Telemetry Data Ingest | – | – | 0.0212 Platform Credits/GB |
| Logging (being replaced by Telemetry) | 1.25 | – | 0.28 Platform Credits/1000 file batches |
| Copy Files | 2 | – | – |
| Snowflake App Runtime | 0.27 | – | – |
| Open Catalog | – | – | 0.5 Platform Credits/1M requests (free for a limited period) |
| Trust Center | 1 | 1 | – |

**Snowpipe billing basis:** text files (CSV, JSON, XML) are billed on **uncompressed** size; binary files (Parquet, Avro, ORC) on **observed** size regardless of compression. This makes Parquet meaningfully cheaper to ingest.

---

## Cortex AI Pricing — All AI Credits Unless Noted

There are **five separate rate tables** for the same underlying models. Using the wrong one is a common error. Pick by *how the model is invoked*:

| Invocation surface | Table | Relative cost |
|--------------------|-------|---------------|
| `AI_COMPLETE` / AI SQL functions, no caching | 6(a) | baseline |
| Cortex Inference **with prompt caching** | 6(b) | ~17% cheaper on input/output |
| Cortex Inference (open-source models) | 6(c) | cheapest |
| **Cortex Agents / Snowflake CoWork** (Snowflake Intelligence) | 6(d) | ~8% premium over 6(a) |
| **Cortex Code (CoCo)** | 6(e) | ~8% below 6(d) |

### Table 6(a): Cortex AI Functions — AI Credits per 1M tokens

| Model | Input | Output |
|-------|-------|--------|
| claude-fable-5 | 6.00 | 30.00 |
| claude-opus-4-5 / 4-6 / 4-7 / 4-8 / claude-opus-5 | 3.00 | 15.00 |
| claude-4-sonnet / claude-sonnet-4-5 / claude-sonnet-4-6 | 1.80 | 9.00 |
| claude-sonnet-5 | 1.20 | 6.00 |
| claude-haiku-4-5 | 0.60 | 3.00 |
| openai-gpt-5.5 | 3.00 | 18.00 |
| openai-gpt-5.5-long-context | 6.00 | 27.00 |
| openai-gpt-5.6-sol | 3.00 | 18.00 |
| openai-gpt-5.6-terra | 1.20 | 7.20 |
| openai-gpt-5.6-luna | 0.12 | 0.72 |
| openai-gpt-5.4 | 1.50 | 9.00 |
| openai-gpt-5.4-long-context | 3.00 | 13.50 |
| openai-gpt-5.4-mini / -nano | 0.45 / 0.12 | 2.70 / 0.75 |
| openai-gpt-5.2 | 1.05 | 8.40 |
| openai-gpt-5 / 5.1 | 0.75 | 6.00 |
| openai-gpt-5-mini / -nano | 0.15 / 0.033 | 1.20 / 0.24 |
| openai-gpt-4.1 | 1.20 | 4.80 |
| gemini-3.1-pro | 1.20 | 7.20 |
| gemini-3.1-pro-long-context | 2.40 | 10.80 |
| gemini-3.5-flash | 0.90 | 5.40 |
| gemini-3.7-flash | 0.45 | 2.25 |
| gemini-3.1-flash-lite | 0.15 | 0.90 |
| gemini-2-5-flash / -flash-lite | 0.18 / 0.06 | 1.50 / 0.24 |
| grok-4.6 | 1.20 | 3.60 |
| grok-4.2 / grok-4.3 | 0.75 | 1.50 |
| mistral-large2 / pixtral-large | 1.20 | 3.60 |
| mistral-large3 | 0.30 | 0.90 |
| mistral-7b / ministral-3-8b | 0.09 | 0.12 / 0.09 |
| mixtral-8x7b | 0.27 | 0.42 |
| llama3.1-70b / llama3.3-70b | 0.432 | 0.432 |
| llama3.1-8b | 0.132 | 0.132 |
| llama4-maverick | 0.144 | 0.582 |
| gemma-4-31b / -26b-a4b / -e2b | 0.084 / 0.078 / 0.024 | 0.24 / 0.24 / 0.048 |
| qwen3-32b / qwen3-next-80b-a3b | 0.09 | 0.36 / 0.72 |
| qwen3-vl-235b-a22b | 0.32 | 1.60 |

Legacy: `llama3-70b` 1.21, `llama3-8b` 0.19 (single blended rate).

**Promotional pricing caveats:** `claude-sonnet-5` rises 50% on Sept 1, 2026. `gemini-3.7-flash` rises 100% on Jan 1, 2027. Do not build a multi-year estimate on either without flagging this.

### Flat-Rate AI Functions — AI Credits per 1M tokens (single blended rate)

| Function | Rate | Function | Rate |
|----------|------|----------|------|
| **AI_EXTRACT – arctic-extract** | **5.55** | AI_AGG | 1.85 |
| AI_SUMMARIZE_AGG | 1.85 | AI_TRANSLATE | 1.63 |
| AI_CLASSIFY | 1.62 | AI_FILTER | 1.62 |
| AI_SENTIMENT | 1.60 | AI_SUMMARIZE | 1.60 |
| AI_TRANSCRIBE | 0.97 | AI_REDACT | 0.69 |
| AI_GUARDRAILS | 0.35 | Guard (legacy) | 0.25 |
| Summarize (legacy) | 0.13 | Extract Answer (legacy) | 0.11 |
| Sentiment (legacy) | 0.09 | | |

Legacy `AI_EXTRACT`: `arctic-tilt-entity` **9.50**, `arctic-tilt-table` **28.40** — 1.7× and 5.1× the cost of `arctic-extract`. Always steer customers to `arctic-extract`.

### Document, Video, Audio & Image — Unit-Priced (Table 6(g))

| Feature | Rate |
|---------|------|
| **AI_PARSE_DOCUMENT – OCR** | **0.68 AI Credits per 1,000 pages** |
| **AI_PARSE_DOCUMENT – Layout** | **3.66 AI Credits per 1,000 pages** |
| AI_COMPLETE – twelvelabs-pegasus-1-2 | 0.294 AI Credits/1,000 video seconds + 4.5 AI Credits/1M output tokens |
| AI_MULTI_EMBED – twelvelabs-marengo-embed-3-0 | 0.420/1,000 video sec; 0.084/1,000 audio sec; 0.060/1,000 image requests; 0.042/1,000 text requests |

**Layout mode costs 5.4× OCR mode.** Only use Layout when table structure, reading order, or form geometry actually matters. At $2.00/AI Credit: OCR = **$0.00136/page**, Layout = **$0.00732/page**.

### Embeddings — AI_EMBED, AI Credits per 1M tokens (input only)

| Model | Rate |
|-------|------|
| voyage-multilingual-2 | 0.07 |
| voyage-multimodal-3 | 0.06 |
| snowflake-arctic-embed-l-v2.0 / nv-embed-qa-4 / multilingual-e5-large | 0.05 |
| e5-base-v2 / snowflake-arctic-embed-m / -m-v1.5 | 0.03 |

### Table 6(b): Cortex Inference with Prompt Caching — AI Credits/1M tokens

| Model | Input | Output | Cache Write | Cache Read |
|-------|-------|--------|-------------|------------|
| claude-opus-4-5 / 4-6 / 4-7 / 4-8 / opus-5 | 2.50 | 12.50 | 3.125 | 0.25 |
| claude-4-sonnet / sonnet-4-5 / sonnet-4-6 | 1.50 | 7.50 | 1.875 | 0.15 |
| claude-sonnet-5 | 1.00 | 5.00 | 1.25 | 0.10 |
| claude-haiku-4-5 | 0.50 | 2.50 | 0.625 | 0.05 |
| claude-fable-5 | 5.00 | 25.00 | 6.25 | 0.50 |
| openai-gpt-5.5 / 5.6-sol | 2.50 | 15.00 | – | 0.25 |
| openai-gpt-5.4 | 1.25 | 7.50 | – | 0.125 |
| openai-gpt-5.2 | 0.875 | 7.00 | – | 0.09 |
| openai-gpt-5 / 5.1 | 0.625 | 5.00 | – | 0.065 |
| openai-gpt-5.6-terra | 1.00 | 6.00 | 1.25 | 0.10 |
| openai-gpt-5.6-luna | 0.10 | 0.60 | 0.125 | 0.01 |
| openai-gpt-5-mini / -nano | 0.125 / 0.03 | 1.00 / 0.22 | – | 0.015 / 0.005 |
| openai-gpt-4.1 | 1.00 | 4.00 | – | 0.25 |
| grok-4.6 | 1.0 | 3.0 | – | 0.25 |
| grok-4.2 / 4.3 | 0.625 | 1.25 | – | 0.1 |

**Cache read is ~90% cheaper than input; cache write is ~25% more.** Break-even after ~2 hits on the same cached prefix. For repeated large system prompts / fixed document context, this is the single biggest AI cost lever.

### Table 6(c): Cortex Inference, Open-Source — AI Credits/1M tokens

| Model | Input | Output |
|-------|-------|--------|
| mistral-large2 | 1.00 | 3.00 |
| llama3.1-70b / llama3.3-70b | 0.36 | 0.36 |
| llama4-maverick | 0.12 | 0.485 |
| llama3.1-8b | 0.11 | 0.11 |
| mistral-7b | 0.075 | 0.10 |
| llama3.2-3b | 0.075 | 0.075 |
| llama3.2-1b | 0.05 | 0.05 |

### Table 6(d): Cortex Agents / Snowflake CoWork / Cortex Analyst-via-Agents — AI Credits/1M tokens

| Model | Input | Output | Cache Write | Cache Read |
|-------|-------|--------|-------------|------------|
| claude-opus-4-5 / 4-6 / 4-7 / 4-8 / opus-5 | 3.25 | 16.26 | 4.07 | 0.33 |
| claude-sonnet-4-5 / sonnet-4-6 | 1.95 | 9.76 | 2.44 | 0.20 |
| claude-sonnet-5 | 1.30 | 6.50 | 1.625 | 0.13 |
| claude-haiku-4-5 | 0.65 | 3.25 | 0.81 | 0.06 |
| claude-fable-5 | 6.50 | 32.50 | 8.13 | 0.65 |
| openai-gpt-5.5 / 5.6-sol | 3.25 | 19.50 | – | 0.33 |
| openai-gpt-5.4 | 1.63 | 9.76 | – | 0.16 |
| openai-gpt-5.2 | 1.14 | 9.11 | – | 0.11 |
| openai-gpt-5 / 5.1 | 0.81 | 6.51 | – | 0.08 |
| openai-gpt-5.6-terra | 1.30 | 7.80 | 1.625 | 0.13 |
| openai-gpt-5.6-luna | 0.13 | 0.78 | 0.163 | 0.013 |
| openai-gpt-4.1 | 1.30 | 5.20 | – | 0.33 |
| gemini-3.1-pro | 1.30 | 7.81 | – | 0.13 |
| gemini-3.5-flash | 0.98 | 5.85 | – | 0.10 |
| gemini-3.7-flash | 0.488 | 2.438 | – | 0.049 |
| gemini-3.1-flash-lite | 0.163 | 0.975 | – | 0.017 |
| grok-4.6 | 1.3 | 3.9 | – | 0.325 |

**Default for Agents/Intelligence estimates: claude-sonnet-4-6 at 1.95 / 9.76.**

### Table 6(e): Cortex Code (CoCo) — AI Credits/1M tokens

| Model | Input | Output | Cache Write | Cache Read |
|-------|-------|--------|-------------|------------|
| claude-opus-4-5 / 4-6 / 4-7 / 4-8 / opus-5 | 2.75 | 13.75 | 3.44 | 0.28 |
| claude-sonnet-4-5 / sonnet-4-6 | 1.65 | 8.25 | 2.07 | 0.17 |
| claude-sonnet-5 | 1.10 | 5.50 | 1.375 | 0.114 |
| claude-haiku-4-5 | 0.55 | 2.75 | 0.688 | 0.055 |
| openai-gpt-5.5 / 5.6-sol | 2.75 | 16.50 | – | 0.28 |
| openai-gpt-5.4 | 1.38 | 8.25 | – | 0.14 |
| openai-gpt-5.2 | 0.97 | 7.70 | – | 0.10 |
| openai-gpt-5.6-terra | 1.10 | 6.60 | 1.375 | 0.11 |
| openai-gpt-5.6-luna | 0.11 | 0.66 | 0.138 | 0.011 |
| gemini-3.7-flash | 0.413 | 2.063 | – | 0.041 |
| grok-4.6 | 1.1 | 3.3 | – | 0.275 |

CoCo sessions are cache-heavy (large repeated codebase/system context) — always model cache reads or you will overstate by 2–5×.

### Cortex Search

| Mode | Rate | Currency |
|------|------|----------|
| **Cortex Search (serving)** | **6.3 AI Credits per GB/month of indexed data** | AI Credits |
| **Batch Cortex Search** | **0.12 AI Credits per GB/hr of indexed data** | AI Credits |
| Embedding at index time | AI_EMBED rates above (0.03–0.07 AI Credits/1M tokens) | AI Credits |

Serving is a **continuous** charge on index size — it accrues whether or not anyone queries. This is the most commonly under-estimated AI line item. At $2.00/AI Credit, 10 GB indexed = **$126/month** standing cost.

### Cortex Analyst — the Platform Credit exception

| Access method | Pricing | Currency |
|---------------|---------|----------|
| **Cortex Analyst API** | **67 Platform Credits per 1,000 messages** (0.067/message) | **Platform Credits** |
| Via Cortex Agents / Snowflake CoWork | Token-based, Table 6(d) | AI Credits |

Plus warehouse compute to run the generated SQL (assume XS, ~5 sec/query).

At Enterprise ($3.00), the API is **$0.201/message**. Via Agents, a typical 4K-in/600-out message on claude-sonnet-4-6 is ~$0.023 — roughly **9× cheaper**. Route high-volume Analyst traffic through Agents unless the API is required.

### Cortex Fine-Tuning (Table 6(f)) — split currency

| Feature | Training (**Platform** Credits/1M tokens) | Inference (**AI** Credits/1M tokens) |
|---------|------------------------------------------|--------------------------------------|
| AI_EXTRACT – arctic-extract – finetuned | 0 | 9.140 |
| llama3.1-70b | 3.40 | 2.42 |
| llama3.1-8b | 0.64 | 0.38 |
| mistral-7b | 0.64 | 0.24 |
| mixtral-8x7b | 3.40 | 0.44 |

Fine-tuned `arctic-extract` inference is **1.65× base** `arctic-extract` (9.140 vs 5.55) — the accuracy gain must justify a permanent 65% inference premium. Training is free for it.

### Provisioned Throughput (Table 6(h)) — Platform Credits

| Cloud | Platform Credits per PTU per hour | Min term |
|-------|----------------------------------|----------|
| AWS | 0.08 | 1 month |
| Azure | 0.10 | 1 month |

**You pay for the full term regardless of actual usage.** Non-cancellable, non-refundable, no auto-renew. Only recommend when sustained throughput is proven. 1 PTU on AWS for a 30-day month = 0.08 × 720 = **57.6 Platform Credits** (~$173 at Enterprise).

---

## Storage

### Table 3(a): Standard Storage — $/TB/month

| Region | On Demand |
|--------|-----------|
| AWS US East/West, EU Dublin, Stockholm, Mumbai; Azure US, W. Europe, Ireland | **$23.00** |
| AWS Asia Pacific (Malaysia, Thailand) | $22.50 |
| AWS Europe (London), EU Paris | $24.00 |
| AWS EU Frankfurt | $24.50 |
| AWS Sydney, Singapore, Canada, Tokyo, Seoul, Osaka, Jakarta | $25.00 |
| AWS Asia Pacific (New Zealand) | $26.25 |
| AWS EU (Zurich) | $26.95 |
| AWS Africa (Cape Town) | $27.40 |
| AWS US Gov / FedRAMP / DoD | $39.00 |
| AWS South America East 1 (São Paulo) | **$40.50** |

**Default: $23.00/TB/month (US, On Demand).**

### Storage modeling
- Snowflake compresses ~3:1 typically (range 2:1–10:1). `compressed_TB = raw_TB / 3` is conservative.
- **Time Travel** (default 1 day) + **Fail-safe** (7 days, non-configurable, Permanent tables only) add ~10–15% overhead. Longer Time Travel (up to 90 days on Enterprise+) can add far more on high-churn tables.
- Transient/temporary tables have **no Fail-safe** — a real lever for large staging data.
- **Storage Lifecycle Policies** move cold data to cheaper tiers; execution is serverless at a 0.50 multiplier. Archive retrieval/write is 0.05 Platform Credits per 1,000 files.
- **Cloud Storage Requests** (Table 3(g)) apply to some features: Class 1 ~$4.50–10.20 per 1M requests, Class 2 ~$0.36–0.64 per 1M. Usually negligible; can matter for many-small-file Iceberg/external patterns.

---

## Data Transfer

Ingress is free. Egress is the risk.

| Path | Typical rate |
|------|--------------|
| Same cloud, same region (non-SPCS) | **$0.00/TB** |
| SPCS / Postgres, same region (AWS) | $3.07/TB (GCP $6.22, Azure $0.00) |
| Same cloud, different region (AWS US/EU) | $20.00/TB |
| Same cloud, cross-region APAC/LATAM/Africa | $60–147/TB |
| **Different cloud provider or internet (AWS US/EU)** | **$90.00/TB** |
| Different cloud / internet, APAC / São Paulo / Africa | $108–155/TB |
| GCP to internet | $120–230/TB |
| Azure to internet | $87.00–120.00/TB |

SPCS Data Transfer gets a daily credit equal to the lower of actual SPCS transfer cost or 10% of that day's SPCS Compute cost.

**Other endpoint charges:** AWS API Gateway / Private Endpoints $10/TB processed. Outbound PrivateLink: $10–21 per endpoint per 1,000 hours, plus tiered data processing ($10.24/TB first PB, $6.14 next 4 PB, $4.09 beyond — GCP $30.72/$26.62/$24.57).

**Always ask where results land.** A 5 TB/month BI extract to a non-Snowflake cloud = $450/month in egress alone, invisible in credit-only estimates.

---

## Organization Usage (Table 8) — Platform Credits/month

| Records powering views/month | Credits/mo |
|------------------------------|-----------|
| <1M | 0 |
| 1M–10M | 2 |
| 10M–50M | 11 |
| 50M–250M | 50 |
| 250M–500M | 115 |
| 500M–1B | 230 |
| 1B–2.5B | 290 |
| 2.5B–5B | 575 |
| 5B–10B | 1,150 |
| 10B–20B | 2,300 |
| ≥20B | 3,500 |

---

## Workflow

### Step 1: Gather Use Case Details

**Ask** the user to describe each use case. Prompt with this template:

```
To estimate costs, describe your use cases. For each one, provide what you can:

**Account basics:**
- Edition: Standard / Enterprise / Business Critical / VPS
- Cloud + region (drives both credit rate and storage rate)
- On Demand or Capacity? If Capacity: contracted rates from the Order Form
- Estimation period: monthly, annual, or one-time

**Data Ingestion:**
- Source(s): S3/ADLS/GCS, Oracle, Kafka, API, SaaS
- One-time load volume (e.g., 100 GB)
- Ongoing volume (e.g., 5 GB/day)
- File formats (Parquet vs CSV/JSON changes Snowpipe billing basis)
- Method: Snowpipe, Snowpipe Streaming, COPY INTO, Openflow, or unsure

**Warehouse Compute:**
- Number of warehouses, size each (XS–6XL)
- Generation: Gen1, Gen2, Interactive, Snowpark-optimized, Adaptive
- Hours/day active, days/month active
- Workload type: ETL, BI, data science, serving
- Multi-cluster? (max clusters multiplies the rate)

**Cortex AI:**
- Which surface: AI SQL functions / Cortex Search / Cortex Analyst / Cortex Agents /
  Snowflake CoWork (Intelligence) / Cortex Code  — this picks the rate table
- Model preference or tier (premium reasoning vs bulk/cheap)
- Volume: rows processed, or users × interactions/day
- Average input and output tokens per call (or let me assume)
- Documents? pages/month and whether layout structure matters (OCR vs Layout)
- Prompt caching applicable? (repeated system prompt or fixed document context)
- Cortex Search: GB of indexed data (this is a continuous monthly charge)

**Storage:**
- Total raw data volume and growth rate
- Time Travel retention needed; permanent vs transient tables

**Data Transfer:**
- Does data leave Snowflake? How much/month, to where (same region, cross-region,
  other cloud, internet)?
```

If the user provides partial info, fill reasonable defaults and **call them out explicitly**.

**STOP**: Confirm the use case details with the user before calculating.

### Step 2: Calculate

#### Ingestion (Platform Credits)

```
Snowpipe:            credits = GB × 0.0037
Snowpipe Streaming:  credits = uncompressed_GB × 0.0037
COPY INTO:           credits = WH_credits_per_hour × load_hours
Openflow BYOC:       credits = vCPUs × hours × 0.0225
```
For Snowpipe, use **uncompressed** GB for CSV/JSON/XML and **observed** GB for Parquet/Avro/ORC. Rule of thumb for COPY INTO: an S warehouse (2 cr/hr) loads ~1 TB/hr of simple CSV.

#### Warehouse Compute (Platform Credits)

```
monthly_credits = credits_per_hour × clusters × hours_per_day × days_per_month
```
Use Table 1(a) for standard, 1(b) for Gen2, 1(d) for Interactive (and enforce the 60-min minimum per start), 1(c) for Snowpark-optimized. Sum warehouses separately. Add Cloud Services only if metadata-heavy (else assume the 10% adjustment zeroes it).

#### Serverless (Platform Credits)

```
credits = compute_hours × 1 × feature_multiplier  (+ unit charges from Table 5)
```

#### Cortex AI SQL Functions (AI Credits)

```
ai_credits = (input_tokens_M × input_rate) + (output_tokens_M × output_rate)
-- Table 6(a). Default claude-sonnet-4-6: 1.80 in / 9.00 out
-- Flat-rate functions (AI_CLASSIFY 1.62, AI_SENTIMENT 1.60, AI_EXTRACT 5.55, …):
ai_credits = total_tokens_M × flat_rate
```
Token estimation: ~1 token ≈ 4 characters ≈ 0.75 words of English.

#### Document Processing (AI Credits)

```
parse_credits = (pages / 1000) × 0.68    -- OCR
parse_credits = (pages / 1000) × 3.66    -- Layout
extract_credits = tokens_M × 5.55        -- AI_EXTRACT arctic-extract
```

#### Cortex Search (AI Credits)

```
embedding_ai_credits = total_tokens_M × embed_rate            (one-time per reindex)
serving_ai_credits   = indexed_GB × 6.3                       (every month, continuous)
batch_ai_credits     = indexed_GB × hours × 0.12              (Batch Cortex Search)
```

#### Cortex Analyst

```
API:        platform_credits = messages × 0.067
            + query compute: messages × (5/3600) × 1  (XS warehouse)
Via Agents: use Table 6(d) token math below
```

#### Cortex Agents / CoWork / Cortex Code (AI Credits)

```
ai_credits = (uncached_input_M × input_rate)
           + (cache_write_M × cache_write_rate)
           + (cache_read_M × cache_read_rate)
           + (output_M × output_rate)
cost_USD = ai_credits × 2.00     -- Global On Demand
```
Agents/CoWork → Table 6(d) (default claude-sonnet-4-6: 1.95 / 9.76 / 2.44 / 0.20).
Cortex Code → Table 6(e) (default claude-sonnet-4-6: 1.65 / 8.25 / 2.07 / 0.17).

#### Storage

```
compressed_TB = raw_TB / 3
monthly_storage_cost = compressed_TB × 1.10 × region_rate   -- 1.10 = TT + Fail-safe
```

#### Data Transfer

```
egress_cost = TB_out × rate_for_destination
```

### Step 3: Build Summary Table

Present results with the credit currency on every row:

```
| Use Case | Category | Currency | Credits | Rate | Period | $ Cost |
|----------|----------|----------|---------|------|--------|--------|
| S3 initial load | Ingestion | Platform | X | $3.00 | One-time | $X |
| S3 daily load | Ingestion | Platform | X | $3.00 | Monthly | $X |
| ETL warehouse (M) | Compute | Platform | X | $3.00 | Monthly | $X |
| BI warehouse (S) | Compute | Platform | X | $3.00 | Monthly | $X |
| Serverless tasks | Serverless | Platform | X | $3.00 | Monthly | $X |
| AI_PARSE_DOCUMENT (OCR) | AI – Docs | AI | X | $2.00 | Monthly | $X |
| AI_EXTRACT | AI – Docs | AI | X | $2.00 | Monthly | $X |
| Cortex Search serving | AI – Search | AI | X | $2.00 | Monthly | $X |
| Cortex Analyst API | AI – Analyst | Platform | X | $3.00 | Monthly | $X |
| Cortex Agents | AI – Agents | AI | X | $2.00 | Monthly | $X |
| Cortex Code | AI – CoCo | AI | X | $2.00 | Monthly | $X |
| Storage | Storage | — | — | $23/TB | Monthly | $X |
| Egress | Transfer | — | — | $90/TB | Monthly | $X |
| **Subtotal Platform Credits** | | Platform | **X** | | Monthly | **$X** |
| **Subtotal AI Credits** | | AI | **X** | | Monthly | **$X** |
| **TOTAL Monthly** | | | | | Monthly | **$X** |
| **TOTAL Annual** | | | | | Annual | **$X** |
```

Always give the **Platform vs AI credit split** as separate subtotals — they are separate currencies.

### Step 4: Present and Review

Present with:
1. **Assumptions listed** — compression ratio, token counts per call, WH sizes, credit rates, region, edition, caching assumptions
2. **Credit currency called out** per line, plus the two subtotals
3. **Sensitivity notes** — e.g., "Medium instead of Small doubles compute"; "no prompt caching triples agent input cost"; "Layout instead of OCR is 5.4× on document parsing"
4. **Optimization tips** (below)

**STOP**: Ask the user if they want to adjust assumptions or add use cases.

---

## Optimization Levers, Highest Impact First

1. **Prompt caching** — up to ~90% off input tokens for repeated prefixes. Biggest single AI lever for Agents/CoCo.
2. **Model routing** — use `llama3.1-8b` (0.11) or `gemini-2-5-flash-lite` (0.06) for bulk classification/extraction; reserve `claude-opus`/`gpt-5.5` (3.00+) for genuine reasoning. That's a 25–50× spread.
3. **OCR instead of Layout** in AI_PARSE_DOCUMENT — 5.4× cheaper when structure doesn't matter.
4. **Right-size Cortex Search indexes** — 6.3 AI Credits/GB/month is continuous. Index only what's searched; drop stale corpora.
5. **Auto-suspend at 60s** and right-size warehouses — still the top platform-credit lever.
6. **Route Analyst through Agents** for high volume (~9× cheaper per message than the API at Enterprise).
7. **Serverless Tasks Flex** (0.5 multiplier) instead of Serverless Tasks (0.9) for latency-tolerant jobs — 44% off.
8. **Parquet over CSV** for Snowpipe — billed on observed rather than uncompressed size.
9. **Transient tables** for staging — no Fail-safe storage.
10. **Keep egress in-region.** $0 vs $90/TB.
11. **Avoid legacy AI_EXTRACT models** — `arctic-tilt-table` is 5.1× `arctic-extract`.
12. **Don't buy Provisioned Throughput** until sustained load is proven; it's non-cancellable and billed in full.

---

## Stopping Points

- After Step 1: Confirm use case details
- After Step 4: Review estimate and adjust

## Output

A formatted cost estimate table with per-use-case credits, credit currency, dollar costs, Platform/AI subtotals, assumptions, sensitivities, and optimization notes. Suitable for sharing with customers or internal stakeholders.

---

## Notes & Common Errors

- **Pricing changes frequently.** Caveat every estimate with "based on rates effective August 26, 2026" and link the [Snowflake Service Consumption Table](https://www.snowflake.com/legal-files/CreditConsumptionTable.pdf).
- **Biggest error #1 — currency.** Nearly all Cortex AI now bills in **AI Credits**, including AI SQL functions and Cortex Search. Only Cortex Analyst API, fine-tuning *training*, Provisioned Throughput, serverless, and compute are Platform Credits. Any guidance that AI SQL functions are edition-priced is out of date.
- **Biggest error #2 — mixing currencies.** Price Platform Credits and AI Credits separately, each at its own rate.
- **Biggest error #3 — wrong AI rate table.** The same model has five different rates (6(a) functions / 6(b) cached / 6(c) open-source / 6(d) Agents+CoWork / 6(e) CoCo). `claude-sonnet-4-6` output ranges 7.50 → 9.76 depending on surface, a 30% spread.
- **Biggest error #4 — forgetting Cortex Search serving.** 6.3 AI Credits/GB/month accrues continuously regardless of query volume.
- **Biggest error #5 — ignoring egress.** Credit-only estimates miss $90/TB internet egress entirely.
- **Interactive Warehouses** have a **60-minute minimum per resume**. Great for sustained serving, terrible for occasional queries — an XS Interactive touched 5× a day across the day costs more than an XS standard warehouse running continuously.
- **Adaptive Warehouses** (Preview) have no published rate. Model as an equivalent standard warehouse and flag it.
- **Promotional pricing:** `claude-sonnet-5` +50% on Sept 1, 2026; `gemini-3.7-flash` +100% on Jan 1, 2027. Flag on any multi-year estimate.
- **Openflow Connector for Oracle** carries a separate per-core software license ($70/core/month) on top of compute. Easy to miss on Oracle migrations.
- **Storage $23/TB/month** (US On Demand). The old $40/TB figure is long obsolete; note São Paulo ($40.50) and US Gov ($39.00) genuinely are that high.
- **Cloud Services** is usually $0 thanks to the 10% adjustment, but Serverless and SPCS cloud services are never adjusted away.
- Compression ratios vary 2:1–10:1. Default 3:1 is conservative; adjust to the customer's data profile.
