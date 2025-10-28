# SemanticInspectDB

Property inspection database for demonstrating SQL Server 2025 text chunking and semantic search on document sections.

## Overview

SemanticInspectDB contains property inspection reports chunked using SQL Server 2025's AI_GENERATE_CHUNKS function. Each inspection report is split into focused segments with pre-computed embeddings, demonstrating how chunking prevents semantic dilution in vector search.

## Requirements

- SQL Server 2025 RC1 or later (compatibility level 170)
- Support for VECTOR data type
- ~20MB disk space for backup file
- ~50MB disk space for restored database

## Database Contents

### Tables

**properties** (100 rows)
- Property information subset from SemanticShoresDB
- Addresses, property types, sizes, and listing details

```sql
CREATE TABLE dbo.properties (
    property_id INT PRIMARY KEY,
    street_address NVARCHAR(200) NOT NULL,
    city NVARCHAR(100) NOT NULL,
    state NVARCHAR(2) NOT NULL,
    zip_code NVARCHAR(10) NOT NULL,
    property_type NVARCHAR(50) NOT NULL,
    bedrooms INT,
    bathrooms DECIMAL(3,1),
    square_feet INT,
    lot_size_acres DECIMAL(10,2),
    year_built INT,
    list_price DECIMAL(12,2),
    listing_status NVARCHAR(20)
);
```

**inspection_reports** (401 rows)
- Full property inspection reports
- Multi-section documents covering foundation, roof, electrical, plumbing, HVAC, etc.

```sql
CREATE TABLE dbo.inspection_reports (
    report_id INT PRIMARY KEY,
    property_id INT NOT NULL,
    inspection_date DATE NOT NULL,
    inspector_name NVARCHAR(200) NOT NULL,
    inspection_type NVARCHAR(50) NOT NULL,
    full_report NVARCHAR(MAX) NOT NULL,
    report_summary NVARCHAR(1000),
    FOREIGN KEY (property_id) REFERENCES dbo.properties(property_id)
);
```

**inspection_chunks** (1,682 rows)
- Chunked inspection report sections
- Each chunk has its own 1536-dimension vector embedding
- Demonstrates section-level search precision

```sql
CREATE TABLE dbo.inspection_chunks (
    chunk_id INT IDENTITY(1,1) PRIMARY KEY,
    report_id INT NOT NULL,
    chunk_order BIGINT NOT NULL,
    chunk_offset BIGINT NOT NULL,
    chunk_length INT NOT NULL,
    chunk_text NVARCHAR(MAX) NOT NULL,
    chunk_vector VECTOR(1536),
    FOREIGN KEY (report_id) REFERENCES dbo.inspection_reports(report_id)
);
```

**search_phrases** (12 rows)
- Pre-embedded search queries for demonstrating diluted vs non-diluted search
- Includes foundation, roof, electrical, plumbing, HVAC, and multi-system queries
- No API key required to test search comparisons

```sql
CREATE TABLE dbo.search_phrases (
    phrase_id INT IDENTITY(1,1) PRIMARY KEY,
    search_phrase NVARCHAR(500) NOT NULL,
    phrase_vector VECTOR(1536),
    category NVARCHAR(100) NOT NULL,
    expected_results NVARCHAR(MAX)
);
```

## Chunking Strategy

Reports were chunked using SQL Server 2025's AI_GENERATE_CHUNKS function with FIXED chunking:

```sql
INSERT INTO dbo.inspection_chunks (
    report_id, chunk_order, chunk_offset, chunk_length, chunk_text
)
SELECT
    r.report_id,
    c.chunk_order,
    c.chunk_offset,
    c.chunk_length,
    c.chunk
FROM dbo.inspection_reports r
CROSS APPLY AI_GENERATE_CHUNKS(
    source = CAST(r.full_report AS NVARCHAR(MAX)),
    chunk_type = FIXED,
    chunk_size = 500,
    overlap = 10
) AS c;
```

**Parameters:**
- Chunk size: 500 characters
- Overlap: 10% (preserves context across boundaries)
- Strategy: FIXED (character-based splitting)

## Vector Embeddings

- **Model**: OpenAI text-embedding-3-small
- **Dimensions**: 1536
- **Distance Metric**: Cosine similarity
- **Purpose**: Enable semantic search on inspection report sections

## Restore Instructions

```sql
RESTORE DATABASE SemanticInspectDB
FROM DISK = 'path\to\SemanticInspectDB.bak'
WITH MOVE 'SemanticInspectDB' TO 'C:\SQLData\SemanticInspectDB.mdf',
     MOVE 'SemanticInspectDB_log' TO 'C:\SQLData\SemanticInspectDB_log.ldf',
     REPLACE;
```

## Use Cases

- Text chunking demonstrations
- Semantic dilution prevention
- Document-level vs chunk-level search comparison
- AI_GENERATE_CHUNKS function examples
- Vector search on document sections

## Sample Queries

### View chunks from a report

```sql
SELECT
    chunk_id,
    chunk_order,
    LEFT(chunk_text, 100) + '...' AS chunk_preview,
    chunk_length
FROM dbo.inspection_chunks
WHERE report_id = 1
ORDER BY chunk_order;
```

### Database statistics

```sql
SELECT
    (SELECT COUNT(*) FROM dbo.properties) AS total_properties,
    (SELECT COUNT(*) FROM dbo.inspection_reports) AS total_reports,
    (SELECT COUNT(*) FROM dbo.inspection_chunks) AS total_chunks,
    (SELECT COUNT(*) FROM dbo.inspection_chunks WHERE chunk_vector IS NOT NULL) AS chunks_with_embeddings;
```

### Compare diluted vs non-diluted search

Search for "foundation cracks in basement" using pre-embedded search phrases:

**Chunk Search (Focused)**
```sql
SELECT TOP 5
    c.chunk_id,
    VECTOR_DISTANCE('cosine', s.phrase_vector, c.chunk_vector) AS distance,
    c.chunk_text
FROM dbo.inspection_chunks c
CROSS JOIN dbo.search_phrases s
WHERE s.search_phrase = 'foundation cracks in basement'
ORDER BY distance;
```

**Full Report Search (Diluted)**
```sql
SELECT TOP 5
    r.report_id,
    VECTOR_DISTANCE('cosine', s.phrase_vector, r.report_vector) AS distance,
    LEFT(r.full_report, 200) + '...' AS preview
FROM dbo.inspection_reports r
CROSS JOIN dbo.search_phrases s
WHERE s.search_phrase = 'foundation cracks in basement'
ORDER BY distance;
```

Chunk search returns distance scores around 0.32 with foundation-specific results. Full report search returns distance scores around 0.55 with entire reports where foundation signal is diluted by roof, electrical, plumbing, and HVAC content.

### Available search phrases

```sql
SELECT phrase_id, search_phrase, category
FROM dbo.search_phrases
ORDER BY category, phrase_id;
```

Categories include Foundation, Roof, Electrical, Plumbing, HVAC, and Multi-System queries.

## Sample Data

All inspection reports and property data are synthetic and generated for educational purposes. Reports contain realistic inspection content across foundation, roof, electrical, plumbing, HVAC, and other property systems.

## License

MIT License - See [LICENSE](../LICENSE) for details.
