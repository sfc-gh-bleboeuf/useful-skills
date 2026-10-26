---
name: unstructured-data-processing
description: "Build unstructured data processing pipelines in Snowflake. Parse documents with AI_PARSE_DOCUMENT, chunk, vectorize, create Cortex Search services, and optionally extract structured fields with AI_EXTRACT. Use when: document processing, unstructured data, PDF pipeline, RAG pipeline, Cortex Search setup, document extraction, AI_PARSE_DOCUMENT, AI_EXTRACT. Triggers: unstructured data, document pipeline, parse documents, cortex search, document extraction, RAG setup."
---

# Unstructured Data Processing Pipeline

Build end-to-end document processing pipelines in Snowflake: parse → chunk → vectorize → search, with optional structured field extraction.

## Workflow

### Step 1: Gather Requirements

**Ask** the user:

```
To build your document processing pipeline, I need:

1. **Stage Setup** — Choose one:
   a) Use an existing external stage (S3, Azure Blob, GCS)
   b) Use an existing internal stage
   c) Create a new internal stage

2. **Document Details:**
   - What types of documents? (PDF, DOCX, PPTX, images, etc.)
   - Approximate volume (number of docs, average page count)
   - Where do files currently live?

3. **Target Objects:**
   - Database and schema to use (or create new)
   - Warehouse name and size (Small/Medium recommended for AI_PARSE_DOCUMENT)

4. **Output Directory:**
   - Where should the generated SQL files be saved locally?
```

**STOP**: Confirm requirements before proceeding.

### Step 2: AI_EXTRACT Decision

**Ask** the user:

```
Do you want to extract structured fields from your documents using AI_EXTRACT?

This is for use cases where documents follow a consistent format and you want
to pull specific fields from each one. Examples:
  - Vendor invoices → invoice_number, vendor_name, total_amount, due_date
  - Contracts → effective_date, parties, term_length, renewal_clause
  - Tax forms → employee_name, wages, tax_withheld

If your documents are varied and you just want to search across them,
you can skip this — Cortex Search alone will handle that.
```

**If user wants AI_EXTRACT:**

1. **Collect fields** — Ask for the field names and extraction prompts. Example:
   ```
   Field: invoice_number
   Prompt: "What is the invoice number?"

   Field: total_amount
   Prompt: "What is the total amount due? Include currency."
   ```

2. **Ask about confidence scores:**
   ```
   AI_EXTRACT supports confidence scores (currently in private preview).
   When enabled, each extracted field returns a score between 0 and 1
   indicating the model's certainty. You can use these to:
     - Flag low-confidence extractions for human review
     - Set thresholds for automated processing
     - Build fallback/escalation workflows

   Syntax: scores => TRUE (added to the AI_EXTRACT call)
   No additional cost for requesting scores.

   Note: This feature requires private preview enablement on your account.
   If you see errors, contact your Snowflake account team to enable it.

   Would you like to include confidence scores in your extraction pipeline?
   ```

**STOP**: Confirm extraction fields (if any) and confidence score preference.

### Step 3: Generate Code

Based on the gathered requirements, generate SQL files into the user's chosen output directory. Follow this structure:

```
<output_dir>/
├── 01_setup.sql
├── 02_parse_and_chunk.sql
├── 03_cortex_search_setup.sql
├── 04_extraction_pipeline.sql   (only if AI_EXTRACT requested)
└── README.md
```

Use the templates below, substituting user-provided values.

**STOP**: Present generated code for review before writing files.

### Step 4: Review & Optional Execution

Present the generated SQL files and ask if the user wants to:
1. Just save the files locally (default)
2. Execute the setup SQL against their Snowflake account

---

## SQL Templates

### 01_setup.sql

```sql
-- =============================
-- SETUP: Database, Schema, Warehouse, Stage
-- =============================
USE ROLE <ROLE>;  -- e.g., SYSADMIN or ACCOUNTADMIN

-- Create database and schema
CREATE DATABASE IF NOT EXISTS <DATABASE>;
CREATE SCHEMA IF NOT EXISTS <DATABASE>.<SCHEMA>;
USE DATABASE <DATABASE>;
USE SCHEMA <SCHEMA>;

-- Create or use warehouse (Small/Medium recommended for AI_PARSE_DOCUMENT)
CREATE WAREHOUSE IF NOT EXISTS <WAREHOUSE>
  WITH WAREHOUSE_SIZE = '<SIZE>'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE;
USE WAREHOUSE <WAREHOUSE>;
```

**If creating a new internal stage:**
```sql
-- Create internal stage with server-side encryption and directory table
CREATE OR REPLACE STAGE <DATABASE>.<SCHEMA>.<STAGE_NAME>
  DIRECTORY = (ENABLE = TRUE)
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE')
  COMMENT = 'Internal stage for document processing pipeline';

-- Upload files via Snowsight UI, SnowSQL PUT, or Snow CLI
-- Example: PUT file:///local/path/*.pdf @<STAGE_NAME>/;

-- Refresh directory table after upload
ALTER STAGE <DATABASE>.<SCHEMA>.<STAGE_NAME> REFRESH;
```

**If using existing external stage:**
```sql
-- Verify external stage exists and has files
SELECT relative_path, size, last_modified
FROM DIRECTORY(@<DATABASE>.<SCHEMA>.<STAGE_NAME>)
ORDER BY last_modified DESC
LIMIT 10;
```

### 02_parse_and_chunk.sql

```sql
-- =============================
-- PARSE: AI_PARSE_DOCUMENT in LAYOUT mode
-- =============================
USE DATABASE <DATABASE>;
USE SCHEMA <SCHEMA>;
USE WAREHOUSE <WAREHOUSE>;

-- Table to store parsed document content
CREATE OR REPLACE TABLE parsed_documents (
  document_id VARCHAR(100) DEFAULT UUID_STRING() PRIMARY KEY,
  file_name VARCHAR(500) NOT NULL,
  file_path VARCHAR(1000) NOT NULL,
  file_size NUMBER,
  parsed_content VARIANT,
  content_text STRING,
  page_count NUMBER,
  parse_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
  status VARCHAR(50) DEFAULT 'parsed'
)
COMMENT = 'Stores AI_PARSE_DOCUMENT results for all processed documents';

-- Parse all documents from stage
INSERT INTO parsed_documents (file_name, file_path, file_size, parsed_content, content_text, page_count)
SELECT
  SPLIT_PART(relative_path, '/', -1) AS file_name,
  relative_path AS file_path,
  size AS file_size,
  AI_PARSE_DOCUMENT(
    TO_FILE('@<STAGE_NAME>', relative_path),
    {'mode': 'LAYOUT'}
  ) AS parsed_content,
  parsed_content:content::STRING AS content_text,
  parsed_content:metadata:pageCount::NUMBER AS page_count
FROM DIRECTORY(@<STAGE_NAME>)
WHERE relative_path ILIKE '%.pdf'
   OR relative_path ILIKE '%.docx'
   OR relative_path ILIKE '%.pptx'
   OR relative_path ILIKE '%.png'
   OR relative_path ILIKE '%.jpg'
   OR relative_path ILIKE '%.jpeg'
   OR relative_path ILIKE '%.tiff';

-- =============================
-- CHUNK: Split parsed text into searchable segments
-- =============================

-- Table to store document chunks
CREATE OR REPLACE TABLE document_chunks (
  chunk_id VARCHAR(150) DEFAULT UUID_STRING() PRIMARY KEY,
  document_id VARCHAR(100) NOT NULL,
  file_name VARCHAR(500),
  file_path VARCHAR(1000),
  chunk_index INTEGER,
  chunk_text STRING,
  chunk_size INTEGER,
  created_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
  FOREIGN KEY (document_id) REFERENCES parsed_documents(document_id)
)
COMMENT = 'Chunked document content for Cortex Search and RAG';

-- Chunk documents using SPLIT_TEXT_RECURSIVE_CHARACTER
INSERT INTO document_chunks (document_id, file_name, file_path, chunk_index, chunk_text, chunk_size)
SELECT
  p.document_id,
  p.file_name,
  p.file_path,
  c.SEQ AS chunk_index,
  c.VALUE::STRING AS chunk_text,
  LENGTH(c.VALUE::STRING) AS chunk_size
FROM parsed_documents p,
  LATERAL FLATTEN(
    INPUT => SPLIT_TEXT_RECURSIVE_CHARACTER(
      p.content_text,
      'markdown',
      1500,   -- chunk size in characters
      300     -- overlap in characters
    )
  ) c
WHERE p.content_text IS NOT NULL;
```

### 03_cortex_search_setup.sql

```sql
-- =============================
-- CORTEX SEARCH SERVICE
-- =============================
USE DATABASE <DATABASE>;
USE SCHEMA <SCHEMA>;
USE WAREHOUSE <WAREHOUSE>;

-- Create Cortex Search service over chunked documents
CREATE OR REPLACE CORTEX SEARCH SERVICE document_search_service
  ON chunk_text
  ATTRIBUTES file_name, file_path
  WAREHOUSE = <WAREHOUSE>
  TARGET_LAG = '2 minutes'
  COMMENT = 'Semantic search over processed document chunks'
AS (
  SELECT
    chunk_text,
    file_name,
    file_path,
    document_id,
    chunk_index
  FROM document_chunks
);

-- =============================
-- TEST: Verify search works
-- =============================

-- Test a search query
SELECT *
FROM TABLE(
  document_search_service.SEARCH(
    '<test query relevant to your documents>',
    5  -- return top 5 results
  )
);
```

### 04_extraction_pipeline.sql (Only if AI_EXTRACT requested)

```sql
-- =============================
-- AI_EXTRACT: Structured Field Extraction
-- =============================
USE DATABASE <DATABASE>;
USE SCHEMA <SCHEMA>;
USE WAREHOUSE <WAREHOUSE>;

-- Table to store extraction results
CREATE OR REPLACE TABLE document_extractions (
  extraction_id VARCHAR(100) DEFAULT UUID_STRING() PRIMARY KEY,
  document_id VARCHAR(100),
  file_name VARCHAR(500),
  file_path VARCHAR(1000),
  extraction_result VARIANT,
  extraction_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
)
COMMENT = 'Structured field extractions from AI_EXTRACT';

-- Extract fields from documents
-- Replace <FIELDS> with your responseFormat
INSERT INTO document_extractions (document_id, file_name, file_path, extraction_result)
SELECT
  document_id,
  file_name,
  file_path,
  AI_EXTRACT(
    file => TO_FILE('@<STAGE_NAME>', file_path),
    responseFormat => {
      <RESPONSE_FORMAT_FIELDS>
    }
    -- SCORES_PARAM_PLACEHOLDER
  ) AS extraction_result
FROM parsed_documents;
```

**responseFormat example (entity extraction):**
```sql
responseFormat => {
  'invoice_number': 'What is the invoice number?',
  'vendor_name': 'What is the vendor or supplier name?',
  'total_amount': 'What is the total amount due? Include currency.',
  'due_date': 'What is the payment due date?'
}
```

**With confidence scores (private preview):**
```sql
AI_EXTRACT(
  file => TO_FILE('@<STAGE_NAME>', file_path),
  responseFormat => {
    'invoice_number': 'What is the invoice number?',
    'vendor_name': 'What is the vendor or supplier name?',
    'total_amount': 'What is the total amount due?'
  },
  scores => TRUE
)
```

When `scores => TRUE`, the output includes a `scoring` object:
```json
{
  "response": {
    "invoice_number": "INV-2024-001",
    "vendor_name": "Acme Corp",
    "total_amount": "$15,000"
  },
  "scoring": {
    "scores": {
      "invoice_number": { "score": 0.99 },
      "vendor_name": { "score": 0.95 },
      "total_amount": { "score": 0.82 }
    }
  },
  "error": null
}
```

**Query extracted fields with scores:**
```sql
-- Flatten extraction results and filter by confidence
SELECT
  e.file_name,
  e.extraction_result:response:invoice_number::STRING AS invoice_number,
  e.extraction_result:scoring:scores:invoice_number:score::FLOAT AS invoice_number_score,
  e.extraction_result:response:vendor_name::STRING AS vendor_name,
  e.extraction_result:scoring:scores:vendor_name:score::FLOAT AS vendor_name_score,
  e.extraction_result:response:total_amount::STRING AS total_amount,
  e.extraction_result:scoring:scores:total_amount:score::FLOAT AS total_amount_score
FROM document_extractions e;

-- Flag low-confidence extractions for human review
SELECT *
FROM (
  SELECT
    file_name,
    extraction_result:response AS extracted_fields,
    extraction_result:scoring:scores AS scores
  FROM document_extractions
)
WHERE scores:<field_name>:score::FLOAT < 0.80;
```

**Confidence scores notes:**
- Scores range from 0 to 1 (higher = more certain)
- No additional cost for requesting scores
- Per-element scores for list items and table cells are not available (aggregate only)
- Scores are not supported for fine-tuned models
- **Private preview** — must be enabled on your account. If you get errors, contact your Snowflake account team.
- Ref: https://docs.snowflake.com/en/LIMITEDACCESS/ai-extract-scores

---

## Stopping Points

- After Step 1: Confirm stage, documents, and target objects
- After Step 2: Confirm AI_EXTRACT fields and confidence scores preference
- After Step 3: Review generated SQL before writing files

## Output

A directory of numbered SQL files that implement the full pipeline:
1. `01_setup.sql` — Infrastructure (DB, schema, warehouse, stage)
2. `02_parse_and_chunk.sql` — AI_PARSE_DOCUMENT + chunking
3. `03_cortex_search_setup.sql` — Cortex Search service
4. `04_extraction_pipeline.sql` — AI_EXTRACT (optional)
5. `README.md` — Setup instructions

## Notes

- **Warehouse sizing**: Snowflake recommends Small or Medium for AI_PARSE_DOCUMENT. Larger warehouses do not increase performance.
- **File limits**: AI_PARSE_DOCUMENT supports files up to 100 MB and 125 pages. For larger documents, split them first (see reference repo pattern).
- **Supported formats**: PDF, DOC/DOCX, PPT/PPTX, JPEG/JPG, PNG, TIFF/TIF, HTML, TXT.
- **AI_EXTRACT limits**: Max 100 entity questions or 10 table questions per call. All values returned as strings. Up to 125 pages per file.
- **Stage encryption**: Internal stages must use server-side encryption (`SNOWFLAKE_SSE`). Client-side encrypted stages are not supported by AI_EXTRACT.
- **Cortex Search serving cost**: ~6.3 credits/GB/month of indexed data, charged continuously while the service is active.
- **Batch processing**: Process documents via `FROM DIRECTORY(@stage)` for efficiency rather than looping individually.
- **Reference architecture**: https://github.com/sfc-gh-bleboeuf/unstructured-data-pipeline
