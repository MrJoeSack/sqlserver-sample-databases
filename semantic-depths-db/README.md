# SemanticDepthsDB

Real estate embedding comparison database used to test OpenAI's text-embedding-3-large (3072 dimensions) vs text-embedding-3-small (1536 dimensions) models. Contains 100,000 property descriptions with embeddings from both models for semantic search testing and evaluation.

## Database Overview

This database was created to evaluate the semantic precision differences between OpenAI's large and small embedding models through real-world real estate search scenarios.

**Key Features:**
- 100,000 synthetic real estate property descriptions
- Dual embeddings: Both text-embedding-3-large (3072-dim) and text-embedding-3-small (1536-dim)
- 20 test queries across 8 semantic dimensions
- 400 evaluation results from blind LLM judge
- SQL Server 2025 float16 vector support (2 bytes per dimension)

## Schema

### Tables

**deep.Properties**
- `property_id` (INT): Unique property identifier
- `street_address` (NVARCHAR): Property street address
- `neighborhood` (NVARCHAR): Neighborhood name
- `city` (NVARCHAR): City name
- `state_code` (NCHAR): Two-letter state code
- `zip_code` (NCHAR): 5-digit ZIP code
- `property_type` (NVARCHAR): Type (Contemporary, Farmhouse, Craftsman, etc.)
- `bedrooms` (TINYINT): Number of bedrooms
- `bathrooms` (DECIMAL): Number of bathrooms
- `square_feet` (INT): Interior square footage
- `lot_size_sqft` (INT): Lot size in square feet
- `year_built` (INT): Year constructed
- `list_price` (INT): Listing price
- `listing_date` (DATE): Date listed
- `agent_id` (INT): Agent identifier
- `listing_description` (NVARCHAR): Property description text
- `description_vector` (VECTOR): Embedding vector (3072-dim float16 OR 1536-dim float32)
- `image_filename` (NVARCHAR): Associated image filename

**deep.SearchPhrases**
- Test queries used for evaluation (20 queries across architectural styles, lifestyle concepts, compound requirements, etc.)

**deep.ComparisonResults**
- Evaluation results from blind LLM judge comparing model outputs

## Database Statistics

- **Size**: ~610 MB (compressed .bak)
- **Rows**: 100,000 properties
- **Vector Storage**:
  - text-embedding-3-large: 3,072 dimensions × 2 bytes (float16) = 6,144 bytes/vector
  - text-embedding-3-small: 1,536 dimensions × 4 bytes (float32) = 6,144 bytes/vector
  - Same storage footprint, different semantic capacity

## Use Cases

This database is ideal for:
- **Embedding Model Evaluation**: Compare large vs small model performance on your own queries
- **Vector Search Testing**: Test SQL Server 2025 vector search with real-world data
- **Float16 Exploration**: Understand float16 vector storage benefits
- **Semantic Search Learning**: Learn how embedding models handle nuanced queries
- **Cost Analysis**: Evaluate if 3.25× cost difference justifies semantic precision gains

## Requirements

- **SQL Server**: 2025 RC1 or later
- **PREVIEW_FEATURES**: Must be enabled for float16 vector support
- **Memory**: Recommend 8GB+ RAM for optimal performance

## Installation

```sql
-- Enable preview features (required for float16)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'PREVIEW_FEATURES', 1;
RECONFIGURE;

-- Restore database
RESTORE DATABASE SemanticDepthsDB
FROM DISK = 'path\to\SemanticDepthsDB.bak'
WITH MOVE 'SemanticDepthsDB' TO 'C:\Data\SemanticDepthsDB.mdf',
     MOVE 'SemanticDepthsDB_log' TO 'C:\Data\SemanticDepthsDB_log.ldf',
     REPLACE;
```

## Sample Queries

### Basic Vector Search
```sql
-- Find properties similar to a query embedding
DECLARE @search_vector VECTOR(3072, float16);
-- (Set @search_vector to your query embedding)

SELECT TOP 10
    property_id,
    street_address,
    property_type,
    listing_description,
    VECTOR_DISTANCE('cosine', description_vector, @search_vector) AS distance
FROM deep.Properties
ORDER BY VECTOR_DISTANCE('cosine', description_vector, @search_vector);
```

### Explore Test Queries
```sql
-- View the 20 test queries used in evaluation
SELECT search_id, search_phrase, category
FROM deep.SearchPhrases
ORDER BY search_id;
```

### Comparison Results
```sql
-- See evaluation results
SELECT TOP 10 *
FROM deep.ComparisonResults
ORDER BY search_id, model, rank;
```

## Related Blog Post

This database was featured in the blog post **"When Contemporary Means Farmhouse"** which explores what the 3.25× cost difference between OpenAI's embedding models actually delivers through blind LLM evaluation methodology.

## License

MIT License - Free to use for learning, testing, and demonstrations.

## Credits

Created by [Joe Sack Consulting LLC](https://github.com/MrJoeSack)
