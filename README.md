# Pensieve - Cross-Platform Memory Layer for AI Agents

## Vision
Build a user-owned, cross-platform memory layer for AI agents that seamlessly unifies personal data across applications.

## Description
This project explores building an interoperable memory layer that consolidates a user's memories in a single place so different AI agents can leverage richer context and produce better results. The design is inspired by existing systems such as Mem0, Zep, and LangMem.

## Memory Ingestion Methods
Memories are primarily created and updated through two complementary methods:

- **Foreground (Real-Time Extension)**: A browser extension observes user interactions on approved websites and updates relevant information immediately.
- **Background (Periodic Sync)**: Memories are periodically refreshed using APIs for user-specified applications.

## Storage Architecture
Different types of memories are persisted in specialized stores:

- **Semantic & Contextual Memories**: Stored in Azure Cosmos DB and retrieved via vector search.
- **Episodic & Procedural Memories**: Incrementally updated in a knowledge graph using Graphiti and Neo4j.

Both stores are queried when contextualization is required, and their results can be merged or ranked for downstream agent workflows.

## Rationale
Current AI assistants lack persistent, structured, and user-controlled long-term context. A unified, portable memory layer can:

- Reduce redundant prompting and repeated user input.
- Enable personalization across heterogeneous agents and tools.
- Improve retrieval quality by combining semantic embeddings with graph-linked episodic structure.
- Support explainability and traceability of agent behavior.

## Project Structure

### `/extension` - Chrome Extension for Real-Time Memory Ingestion
Browser extension that captures user interactions and conversations in real-time across supported platforms.

- **`manifest.json`**: Extension configuration and permissions
- **`background.js`**: Service worker for extension lifecycle management
- **`popup.html/js`**: Extension popup interface for user controls
- **`src/teams/content.js`**: Microsoft Teams content script for memory retrieval and suggested replies
- **`src/chatgpt/content.js`**: ChatGPT content script for conversation capture
- **`src/services/auth.service.js`**: Authentication service for secure API communication

### `/server` - Django Backend API
RESTful API server providing memory storage, retrieval, and processing capabilities.

- **`memories/`**: Core memory management Django app
  - **`models.py`**: Memory and summary data models
  - **`views.py`**: API endpoints for memory CRUD operations and retrieval
  - **`azure_openai.py`**: Azure OpenAI integration for embeddings and LLM calls
  - **`cosmos_db.py`**: Azure Cosmos DB connection and vector search
  - **`graphiti_client.py`**: Graphiti knowledge graph integration
- **`authentication/`**: User authentication and authorization
- **`requirements.txt`**: Python dependencies
- **`manage.py`**: Django management script

### `/teams_app` - Microsoft Teams Application
Native Teams application for enhanced integration and bot capabilities.

- **`src/`**: TypeScript source code for Teams app logic
- **`appPackage/`**: Teams app manifest and assets
- **`infra/`**: Azure infrastructure as code (Bicep templates)

## Getting Started

### Prerequisites
- Python 3.8+
- Node.js 16+
- Azure Cosmos DB account
- Azure OpenAI service
- Chrome browser for extension development

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Achint-Agrawal/Pensieve.git
   cd Pensieve
   ```

2. **Set up the backend server**
   ```bash
   cd server
   pip install -r requirements.txt
   python manage.py migrate
   python manage.py runserver
   ```

3. **Install the Chrome extension**
   - Open Chrome and navigate to `chrome://extensions/`
   - Enable "Developer mode"
   - Click "Load unpacked" and select the `/extension` folder

4. **Configure environment variables**
   - Copy `.env.example` to `.env` in the server directory
   - Add your Azure Cosmos DB and OpenAI credentials

## Usage

1. **Extension Usage**: Install the Chrome extension and visit supported platforms (Teams, ChatGPT)
2. **Memory Retrieval**: The extension automatically suggests contextual replies based on conversation history
3. **API Access**: Use the REST API endpoints for programmatic memory management
