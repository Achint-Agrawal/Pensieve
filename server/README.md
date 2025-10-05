# Pensieve Server

A Django-based RESTful API service that stores and retrieves memories with vector embeddings using Azure Cosmos DB and Azure OpenAI.

> **Note:** The Django project is internally named `memories_project` for structural reasons, but the overall system is called "Pensieve".

## Features

- Store memories with content
- Automatic vector embedding generation using Azure OpenAI
- Semantic search capabilities using vector similarity
- CosmosDB integration for persistent storage
- RESTful API endpoints
- Docker support for easy deployment

## Prerequisites

- Python 3.11 or higher
- Azure Cosmos DB account
- Azure OpenAI service access
- Docker (optional)

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd server
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Configure environment variables:
   - Copy `.env.template` to `.env`
   - Fill in your Azure credentials and settings:
```bash
cp .env.template .env
# Edit .env with your credentials
```

Required environment variables:
```
# Django Settings
DJANGO_SECRET_KEY=your-secret-key
DEBUG=1

# CosmosDB Settings
COSMOS_DB_HOST=your-cosmos-db-host
COSMOS_DB_KEY=your-cosmos-db-key
COSMOS_DB_NAME=memories_db
COSMOS_DB_CONTAINER=memories

# Azure OpenAI Settings
AZURE_OPENAI_ENDPOINT=your-openai-endpoint
AZURE_OPENAI_KEY=your-openai-key
AZURE_OPENAI_VERSION=2023-05-15
AZURE_OPENAI_DEPLOYMENT=your-model-deployment-name
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=your-embedding-deployment-name
```

## Running the Service

### Using Python directly:
```bash
python manage.py runserver
```

### Using Docker:
```bash
docker-compose up --build
```

The service will be available at `http://localhost:8000`

## API Endpoints

### 1. Add Memory
- **URL**: `/api/memories/add/`
- **Method**: POST
- **Request Body**:
```json
{
   "content": "Memory content goes here..."
}
```
- **Response**: Created memory object with ID and timestamps

### 2. Retrieve Memories
- **URL**: `/api/memories/retrieve/`
- **Method**: GET
- **Response**: List of all memories with their embeddings

## Technical Details

### Architecture
- Django REST framework for API endpoints
- Azure Cosmos DB for data persistence
- Azure OpenAI for vector embeddings
- Vector similarity search for memory retrieval
- Docker containerization

### Data Model
The Memory model includes:
- `id`: Unique identifier
- `content`: Memory content
- `created_at`: Creation timestamp
- `updated_at`: Last update timestamp
- `embedding`: Vector embedding of the memory content

### Vector Embeddings
- Generated automatically for each memory
- Used for semantic similarity search
- Based on Azure OpenAI's text-embedding model

## Development

### Project Structure
```
memories_service/
├── memories/
│   ├── models.py      # Memory data model
│   ├── views.py       # API endpoints
│   ├── cosmos_db.py   # CosmosDB integration
│   └── azure_openai.py # OpenAI integration
├── memories_project/
│   └── settings.py    # Project settings
├── Dockerfile         # Docker configuration
├── docker-compose.yml # Docker Compose configuration
├── requirements.txt   # Python dependencies
└── .env              # Environment variables
```

### Adding New Features
1. Modify the Memory model in `models.py`
2. Update API endpoints in `views.py`
3. Run migrations if using Django models
4. Update tests and documentation

## Security Notes
- Keep your `.env` file secure and never commit it to version control
- Use secure values for `DJANGO_SECRET_KEY`
- Configure `ALLOWED_HOSTS` in production
- Use HTTPS in production
- Implement authentication as needed

## Troubleshooting

### Common Issues
1. Module not found errors:
   ```bash
   pip install -r requirements.txt
   ```

2. CosmosDB connection issues:
   - Verify credentials in `.env`
   - Check network connectivity
   - Ensure container exists

3. OpenAI API issues:
   - Verify API key and endpoint
   - Check deployment names
   - Monitor rate limits

## Graphiti Integration

This service now ingests processed conversational memories into a Graphiti temporal knowledge graph (Neo4j backend).

Environment variables required for Graphiti:

```
OPENAI_API_KEY=your_openai_key
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
# Optional
MODEL_NAME=gpt-4o-mini
SMALL_MODEL_NAME=gpt-4o-mini
GRAPHITI_TELEMETRY_ENABLED=false  # disable anonymous telemetry if desired
SEMAPHORE_LIMIT=10                # increase cautiously for faster ingestion
```

Ingestion occurs in `memories/views.py` inside `process_memory` after memory decision logic. Failures to ingest are non-fatal and returned under the `graphiti` key of the JSON response.

See `memories/graphiti_client.py` for lazy initialization logic.

## Demo Memory Seeding

To populate the system with a curated demo dataset (35 synthetic engineering/project memories) and corresponding Graphiti episodes:

1. Ensure the server is running (and Graphiti/Neo4j reachable if enabled):
    ```bash
    python manage.py runserver 8000
    ```
2. Run the seeding script:
    ```bash
    python scripts/seed_demo_memories.py --host http://localhost:8000
    ```
    Optional flags:
    - `--limit 10` seed only first 10
    - `--dry-run` show what would be posted
    - `--delay 0.2` add slight delay between posts

### New Endpoint

`POST /api/memories/add-with-graphiti/`

Request JSON:
```json
{
   "id": "M-001",            // optional stable id; if omitted generated
   "content": "text...",      // required
   "episode_name": "episode-m-001", // optional Graphiti episode name
   "source_description": "seed_demo" // optional
}
```

Idempotency behavior:
* If provided `id` already exists with identical content -> returns 200 with `idempotent: true`.
* If recent memory has same content -> duplicate skipped (200) to avoid spam.
* Otherwise creates Cosmos item (embedding auto-generated) then ingests Graphiti episode.

Response (201 Created):
```json
{
   "memory": { "id": "M-001", "content": "...", ... },
   "graphiti": { "ingested": true, "episode_name": "episode-m-001" }
}
```
On skip (200):
```json
{
   "memory": { ...existing... },
   "graphiti": { "ingested": false, "skipped": true, "reason": "duplicate content" },
   "idempotent": true
}
```

### Verifying
* List recent memories: `GET /api/memories/list/?limit=10`
* Hybrid (future) or vector search: `GET /api/memories/retrieve/?q=partition+key`
* Inspect Graphiti (via its UI / Cypher) for new Episodic nodes containing the seeded texts.


## Contributing
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License
[MIT License](LICENSE)