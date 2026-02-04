# SemanticSonarDB

Real estate property database demonstrating production patterns for SQL Server 2025 vector search.

## Overview

SemanticSonarDB extends the SemanticShoresDB foundation with additional tables and embeddings for advanced vector search patterns including model comparison, hybrid search, RAG, chunking, and reranking.

## Requirements

- SQL Server 2025 RC1 or later (compatibility level 170)
- Support for VECTOR data type
- ~1GB disk space for backup file
- ~1.6GB disk space for restored database
- Ollama with bge-m3 and all-minilm models (optional, for generating new embeddings)

## Database Contents

### Core Tables

**PropertyListings** (100,036 rows)
- Property listings with addresses, features, and descriptions

```sql
CREATE TABLE PropertyListings (
    property_id INT PRIMARY KEY,
    street_address NVARCHAR(200),
    neighborhood NVARCHAR(100),
    property_type NVARCHAR(50),
    bedrooms TINYINT,
    bathrooms DECIMAL(3,1),
    square_feet INT,
    list_price INT,
    listing_description NVARCHAR(MAX),
    status NVARCHAR(20)
);
```

**PropertyEmbeddings** (100,041 rows)
- Normalized embeddings table using bge-m3 model (1024 dimensions)

```sql
CREATE TABLE PropertyEmbeddings (
    property_id INT NOT NULL,
    model_name NVARCHAR(50) NOT NULL,
    embedding VECTOR(1024) NOT NULL,
    created_at DATETIME2,
    CONSTRAINT PK_PropertyEmbeddings PRIMARY KEY (property_id, model_name)
);
```

**PropertyEmbeddings_384** (1,010 rows)
- Embeddings using all-minilm model (384 dimensions) for model comparison

```sql
CREATE TABLE PropertyEmbeddings_384 (
    property_id INT NOT NULL,
    model_name NVARCHAR(50) NOT NULL,
    embedding VECTOR(384) NOT NULL,
    created_at DATETIME2,
    CONSTRAINT PK_PropertyEmbeddings_384 PRIMARY KEY (property_id, model_name)
);
```

### Supporting Tables

**EmbeddingModels** (3 rows)
- Model registry for version tracking

**InspectionReports** (15 rows)
- Property inspection reports with embeddings for RAG patterns

**HOADocuments** (15 rows)
- HOA documents for hybrid search patterns

**BuyerSearchCases** (55 rows) / **BuyerSearchRelevance** (1,000,000 rows)
- Ground truth data for quality metrics
- query_embedding column (bge-m3, 1024-dim) for SQL-first demos

**BuyerSearchCaseEmbeddings_384** (50 rows)
- Same buyer queries embedded with all-minilm (384-dim)
- Used for model comparison demos

**QueryBank** (11 rows)
- Pre-embedded queries organized by module number
- Enables SQL-only demos without Python dependency

**IndexConfigurations** (4 rows)
- DiskANN index configuration options

**SearchConfig** (1 row)
- Active model configuration for runtime switching

## Vector Embeddings

| Model | Dimensions | Provider | Count |
|-------|------------|----------|-------|
| bge-m3 | 1024 | BAAI/Ollama | 100,041 |
| all-minilm | 384 | sentence-transformers/Ollama | 1,010 |

Distance metric: Cosine similarity

## Restore Instructions

```sql
-- Adjust paths as needed for your environment
RESTORE DATABASE SemanticSonarDB
FROM DISK = 'path\to\SemanticSonarDB.bak'
WITH MOVE 'SemanticSonarDB' TO 'C:\SQLData\SemanticSonarDB.mdf',
     MOVE 'SemanticSonarDB_log' TO 'C:\SQLData\SemanticSonarDB_log.ldf',
     REPLACE;
```

## Use Cases

- Vector search with multiple embedding models
- Hybrid search (vector + keyword)
- RAG patterns with inspection reports
- Search quality metrics (P@K, Recall, MRR, nDCG)
- Model migration and A/B testing patterns
- Candidate pool sizing experiments

## Related

- [SemanticShoresDB](../semantic-shores-db) - Foundation database with OpenAI embeddings

## Sample Data

All property data is synthetic and generated for educational purposes. Inspection reports and HOA documents are fictional.

## License

MIT License - See [LICENSE](../LICENSE) for details.
