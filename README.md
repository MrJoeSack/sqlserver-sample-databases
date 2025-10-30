# SQL Server Sample Databases

Sample databases for SQL Server demonstrations and learning.

## Databases

### [SemanticShoresDB](./semantic-shores-db/)
Real estate property database demonstrating SQL Server 2025 vector search capabilities. Includes 100,000 properties with 1536-dimensional text embeddings for semantic search.

### [SemanticDepthsDB](./semantic-depths-db/)
Real estate database with text-embedding-3-large (3072-dim) embeddings for the same 100,000 properties in semantic-shores-db. Use both databases together to compare semantic precision between OpenAI's large and small embedding models.

### [SemanticInspectDB](./semanticinspectdb/)
Property inspection database demonstrating SQL Server 2025 text chunking with AI_GENERATE_CHUNKS. Includes 401 inspection reports chunked into 1,682 segments with pre-generated embeddings for comparing diluted vs non-diluted search.

## Requirements

- SQL Server 2025 RC1 or later
- Restore using standard RESTORE DATABASE command

## Usage

Each database folder contains:
- Database backup (.bak file)
- README with schema details
- Sample queries (where applicable)

## License

MIT License - See [LICENSE](./LICENSE) for details.
