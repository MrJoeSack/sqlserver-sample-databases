# SQL Server Sample Databases

Sample databases for SQL Server demonstrations and learning.

## Databases

### [semantic-shores-db](./semantic-shores-db/)
Real estate property database demonstrating SQL Server 2025 vector search capabilities. Includes 100,000 properties with 1536-dimensional text embeddings for semantic search.

### [semantic-depths-db](./semantic-depths-db/)
Real estate embedding comparison database for testing OpenAI's text-embedding-3-large (3072-dim) vs text-embedding-3-small (1536-dim). Includes 100,000 properties with dual embeddings and 20 test queries for your own evaluation.

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
