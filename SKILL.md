---
name: cost-estimator
description: "Estimate Snowflake costs for customer use cases. Use when: sizing, pricing estimate, cost calculator, estimate credits, TCO, consumption forecast, how much will it cost. Triggers: estimate cost, cost estimate, sizing exercise, credit estimate, consumption estimate, pricing calculator."
---

# Snowflake Cost Estimator

Estimate Snowflake credit and dollar costs for customer use cases. Collects use case details, maps to cost categories, calculates credits, and produces a summary table.

## Pricing Reference (keep current — check Snowflake Service Consumption Table)

### Warehouse Credits per Hour

| Size | Credits/hr |
|------|-----------|
| XS | 1 |
| S | 2 |
| M | 4 |
| L | 8 |
| XL | 16 |
| 2XL | 32 |
| 3XL | 64 |
| 4XL | 128 |

### Snowpipe Ingestion
- **0.0037 credits per GB** (flat rate, all editions)
- Text files (CSV, JSON): billed on uncompressed size
- Binary files (Parquet, Avro): billed on observed size

### COPY INTO (Warehouse-Based Ingestion)
- Uses warehouse credits × runtime
- Estimate load time based on data volume and WH size

### Storage
- **Capacity (contract):** ~$23/TB/month (US regions)
- **On-Demand:** ~$40/TB/month (US regions)
- Snowflake compresses data ~3-5x; estimate compressed size = raw size ÷ 3
- Time Travel (default 1 day) and Fail-safe (7 days) add ~10-15% overhead

### Cortex Analyst
- **6.7 credits per 100 messages** (0.067 credits/message)
- Fixed per successful message, not token-based
- Plus warehouse compute for executing the generated SQL queries
- Default assumption: XS warehouse, ~5 seconds per query execution

### Cortex AISQL Functions (AI_COMPLETE, AI_SENTIMENT, etc.)
- Token-based pricing, varies by model (0.3–12 credits/million tokens)
- Both input + output tokens are billed for generative functions
- Embeddings: ~0.05 credits/million tokens (input only)

### Cortex Search
- Embedding: ~0.05 credits/million tokens (one-time per row)
- Serving: ~6.3 credits/GB/month of indexed data (continuous)

### Credit-to-Dollar Rates (on-demand, US regions)

| Edition | $/Credit |
|---------|----------|
| Standard | $2.00–$3.10 |
| Enterprise | $3.00–$4.65 |
| Business Critical | $4.00–$6.20 |

Common default: **$3/credit (Enterprise)** unless customer specifies otherwise.

## Workflow

### Step 1: Gather Use Case Details

**Ask** the user to describe each use case. Prompt with this template:

```
To estimate costs, describe your use cases. For each one, provide what you can:

**Data Ingestion:**
- Source(s): (e.g., S3, Oracle, Kafka, API)
- One-time load volume: (e.g., 100 GB)
- Ongoing daily volume: (e.g., 5 GB/day)
- Ingestion method: (Snowpipe, COPY INTO, Snowpipe Streaming, or unsure)

**Warehouse Compute:**
- Number of warehouses
- Size per warehouse (XS through 4XL, or unsure)
- Hours per day active
- Days per week/month active
- Workload type: (ETL, BI queries, data science, etc.)

**Cortex AI:**
- Service: (Analyst, Search, AISQL functions, Agents)
- Number of users
- Usage per user per day (e.g., 30 min, 10 queries)
- If Analyst: estimated messages/queries per user per day

**Storage:**
- Total data volume (raw, before compression)
- Data growth rate (e.g., 5 GB/day)
- Retention needs (Time Travel days)

**Other:**
- Edition: (Standard, Enterprise, Business Critical)
- Credit rate if known (or use default $3/credit)
- Estimation period: (monthly, annual, or one-time)
```

If the user provides partial info, fill reasonable defaults and call them out.

**STOP**: Confirm the use case details with the user before calculating.

### Step 2: Calculate Credits per Use Case

For each use case provided, apply the relevant formula:

#### Ingestion Credits

**Snowpipe:**
```
credits = data_volume_GB × 0.0037
```

**COPY INTO (warehouse-based):**
```
credits = WH_credits_per_hour × estimated_load_hours
```
Rule of thumb: A Small (2 cr/hr) WH loads ~1 TB/hr for simple CSV; adjust for complexity.

**One-time loads:** Calculate once.
**Ongoing loads:** Calculate per day, multiply by days/month.

#### Warehouse Compute Credits

```
monthly_credits = WH_credits_per_hour × hours_per_day × days_per_month
```

For multiple warehouses, sum each separately.

If user says "5 weeks" or a specific project duration:
```
total_credits = WH_credits_per_hour × hours_per_day × total_days
```

#### Cortex Analyst Credits

```
messages_per_day = users × queries_per_user_per_day
monthly_messages = messages_per_day × workdays_per_month (default 20)
analyst_credits = monthly_messages × 0.067

-- Plus warehouse compute for SQL execution
-- Assume XS (1 credit/hr), ~5 sec per query
query_compute_credits = monthly_messages × (5/3600) × 1
```

Total Cortex Analyst = analyst_credits + query_compute_credits

**Estimating queries per user:** If user says "30 min/day", estimate ~1 query every 3 minutes = ~10 queries per 30-min session. Adjust based on context.

#### Cortex AISQL Credits

```
credits = total_tokens / 1_000_000 × model_credit_rate
```

#### Cortex Search Credits

```
embedding_credits = total_rows × avg_tokens_per_row / 1_000_000 × 0.05  (one-time)
serving_credits = index_size_GB × 6.3  (per month, continuous)
```

#### Storage Cost

```
compressed_TB = raw_data_TB / 3  (default compression ratio)
monthly_storage_cost = compressed_TB × storage_rate_per_TB
```

Add ~10% for Time Travel + Fail-safe overhead if using default 1-day retention.

### Step 3: Build Summary Table

Present results in a clear table:

```
| Use Case | Category | Credits | Period | $ Cost |
|----------|----------|---------|--------|--------|
| S3 Ingestion (one-time) | Ingestion | X | One-time | $X |
| S3 Ingestion (daily) | Ingestion | X | Monthly | $X |
| Oracle Ingestion (one-time) | Ingestion | X | One-time | $X |
| Oracle Ingestion (daily) | Ingestion | X | Monthly | $X |
| Warehouse A | Compute | X | Monthly | $X |
| Warehouse B | Compute | X | Monthly | $X |
| Cortex Analyst | AI Services | X | Monthly | $X |
| Data Storage | Storage | - | Monthly | $X |
| **TOTAL (Monthly)** | | **X** | **Monthly** | **$X** |
| **TOTAL (Annual)** | | **X** | **Annual** | **$X** |
```

Show one-time costs separately from recurring monthly costs.

Dollar cost = credits × credit_rate (default $3/credit Enterprise).
Storage is already in dollars (not credits).

### Step 4: Present and Review

Present the summary table with:
1. **Assumptions listed** — compression ratio, queries/user estimate, WH sizes chosen, credit rate
2. **Sensitivity notes** — e.g., "If WH size is Medium instead of Small, compute cost doubles"
3. **Optimization tips** — auto-suspend, right-sizing, Snowpipe vs COPY INTO tradeoffs

**STOP**: Ask user if they want to adjust any assumptions or add more use cases.

## Stopping Points

- After Step 1: Confirm use case details
- After Step 4: Review estimate and adjust

## Output

A formatted cost estimate table with per-use-case credits, dollar costs, assumptions, and optimization notes. Suitable for sharing with customers or internal stakeholders.

## Notes

- Pricing changes frequently. Always caveat estimates with "based on current published rates" and recommend checking the [Snowflake Service Consumption Table](https://www.snowflake.com/legal-files/CreditConsumptionTable.pdf).
- For Oracle ingestion, if using a connector or third-party tool, the warehouse used by that tool is the cost driver — estimate as warehouse compute.
- Snowpipe is typically used for S3/Azure Blob/GCS; database sources like Oracle typically use COPY INTO or a connector with warehouse compute.
- Compression ratios vary widely (2x–10x). Default 3x is conservative; adjust if customer has specific data characteristics.
