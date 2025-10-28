# SemanticShoresDB

Real estate property database for demonstrating SQL Server 2025 vector search and semantic similarity queries.

## Overview

SemanticShoresDB contains realistic property listings with pre-computed text embeddings, enabling semantic search demonstrations without requiring external embedding APIs.

## Requirements

- SQL Server 2025 RC1 or later (compatibility level 170)
- Support for VECTOR data type
- ~600MB disk space for backup file
- ~1GB disk space for restored database

## Database Contents

### Tables

**properties** (100,000 rows)
- Property listings with addresses, features, and descriptions
- Each property includes a 1536-dimension vector embedding
- Indexed for fast semantic similarity search

```sql
CREATE TABLE properties (
    property_id INT PRIMARY KEY,
    street_address NVARCHAR(200),
    neighborhood NVARCHAR(100),
    city NVARCHAR(100),
    state_code CHAR(2),
    zip_code CHAR(5),
    property_type NVARCHAR(50),
    bedrooms TINYINT,
    bathrooms DECIMAL(3,1),
    square_feet INT,
    lot_size_sqft INT NULL,
    year_built SMALLINT,
    list_price INT,
    listing_date DATE,
    agent_id INT,
    listing_description NVARCHAR(MAX),
    description_vector VECTOR(1536) NULL,
    image_filename NVARCHAR(255) NULL
);

CREATE VECTOR INDEX ix_properties_description_vector
ON properties(description_vector);
```

**agents** (12 rows)
- Real estate agent information

```sql
CREATE TABLE agents (
    agent_id INT PRIMARY KEY,
    agent_name NVARCHAR(100),
    email NVARCHAR(100),
    phone NVARCHAR(20),
    rating DECIMAL(2,1),
    years_experience INT
);
```

**search_phrases** (955 rows)
- Common search queries with pre-computed embeddings
- Used for semantic search demonstrations

```sql
CREATE TABLE search_phrases (
    search_id INT PRIMARY KEY,
    search_phrase NVARCHAR(500),
    search_vector VECTOR(1536) NULL,
    category NVARCHAR(50) NULL,
    created_date DATETIME2
);
```

## Vector Embeddings

- **Model**: OpenAI text-embedding-3-small
- **Dimensions**: 1536
- **Distance Metric**: Cosine similarity
- **Purpose**: Enable semantic similarity search on property descriptions

## Restore Instructions

```sql
-- WARNING: The REPLACE option will overwrite any existing database named SemanticShoresDB
-- Verify no production database uses this name before running this command
RESTORE DATABASE SemanticShoresDB
FROM DISK = 'path\to\SemanticShoresDB_20251018.bak'
WITH MOVE 'SemanticShoresDB' TO 'C:\SQLData\SemanticShoresDB.mdf',
     MOVE 'SemanticShoresDB_log' TO 'C:\SQLData\SemanticShoresDB_log.ldf',
     REPLACE;
```

## Use Cases

- Vector search demonstrations
- Semantic similarity queries
- Hybrid search (combining traditional and vector search)
- Performance testing with vector indexes
- Educational demonstrations of AI-era database features

## Sample Data

Properties span multiple cities and property types with realistic descriptions and pricing. All data is synthetic and generated for educational purposes.

## License

MIT License - See [LICENSE](../LICENSE) for details.
