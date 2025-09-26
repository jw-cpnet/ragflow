# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Default Configuration (Updated)

RAGFlow now uses the following defaults for easy Docker Compose setup:
- **Vector Store**: Infinity (instead of Elasticsearch)
- **Database**: PostgreSQL (instead of MySQL)
- **MinIO Port**: 9002 (instead of 9000)
- **MCP Server**: Available but disabled by default (uncomment to enable)
- **Quick Start**: `docker compose -f docker/docker-compose.yml up -d`

## Development Commands

### Backend Development
- **Start backend services for development**: `bash docker/launch_backend_service.sh`
- **Start task executors**: The launch script handles this automatically with `WS` workers
- **Main server entry**: `python api/ragflow_server.py`
- **Task executor entry**: `python rag/svr/task_executor.py`

### Frontend Development
- **Install frontend dependencies**: `cd web && npm install`
- **Start frontend dev server**: `cd web && npm run dev`
- **Build frontend**: `cd web && npm run build`
- **Lint frontend code**: `cd web && npm run lint`
- **Run frontend tests**: `cd web && npm test`

### Python Development
- **Install dependencies**: `uv sync --python 3.10 --all-extras`
- **Download ML dependencies**: `uv run download_deps.py`
- **Python linting**: Uses ruff (configured in pyproject.toml)
- **Run tests**: `pytest` (with markers p1, p2, p3 for priority levels)

### Docker Development
- **Start base services**: `docker compose -f docker/docker-compose-base.yml up -d`
- **Full Docker setup**: `docker compose -f docker/docker-compose.yml up -d`
- **GPU-enabled**: `docker compose -f docker/docker-compose-gpu.yml up -d`

## Architecture Overview

### Core Components

**API Layer (`api/`)**
- `ragflow_server.py`: Main Flask application server
- `apps/`: REST API endpoints organized by domain (kb_app, dialog_app, document_app, etc.)
- `db/`: Database models, services, and data access layer using Peewee ORM
- `utils/`: Common utilities for validation, logging, file handling, etc.

**RAG Engine (`rag/`)**
- `app/`: Document processing templates (naive, manual, qa, resume, etc.)
- `llm/`: LLM integrations (chat_model, embedding_model, rerank_model, etc.)
- `nlp/`: Natural language processing utilities (tokenization, search, query processing)
- `flow/`: Document processing pipeline (chunker, parser, tokenizer)
- `svr/task_executor.py`: Background task processing worker
- `utils/`: Storage connectors (ES, Redis, MinIO, etc.)

**Document Processing (`deepdoc/`)**
- `parser/`: Format-specific parsers (PDF, DOCX, PPT, Excel, HTML, etc.)
- `vision/`: OCR and layout recognition capabilities
- Specialized resume parsing with entity extraction

**Agent System (`agent/`)**
- `component/`: Workflow components (LLM, iteration, categorize, invoke, etc.)
- `tools/`: External tool integrations (search engines, APIs, databases)
- `templates/`: Pre-built agent workflows (customer service, research, etc.)
- `canvas.py`: Agent workflow orchestration

**Knowledge Graph (`graphrag/`)**
- `general/`: Full GraphRAG implementation with community detection
- `light/`: Lightweight graph extraction
- Entity resolution and graph-based search capabilities

**Frontend (`web/`)**
- React + TypeScript + Ant Design + UmiJS
- Component-based architecture with modern React patterns
- Real-time chat interface with streaming responses

### Key Architectural Patterns

**Multi-Tenant Design**: All data is tenant-scoped through the database layer

**Plugin Architecture**: Modular components for parsers, LLMs, and tools

**Async Task Processing**: Background workers handle document processing and embeddings

**Microservice-Ready**: Clean separation between API, RAG engine, and processing workers

**Storage Abstraction**: Unified interface for different storage backends (Infinity, ES, OpenSearch)

**Document Pipeline**: Configurable processing flows with chunking strategies and parsing templates

## Configuration Files

- `docker/.env`: Environment variables for Docker deployment
- `docker/service_conf.yaml.template`: Backend service configuration template
- `pyproject.toml`: Python dependencies and tool configuration
- `web/package.json`: Frontend dependencies and build scripts

## Important Environment Setup

For development, ensure these hosts are in `/etc/hosts`:
```
127.0.0.1    es01 infinity postgres mysql minio redis sandbox-executor-manager
```

Set `HF_ENDPOINT=https://hf-mirror.com` if HuggingFace access is restricted.

## Testing Strategy

- **Backend**: Uses pytest with priority markers (p1=high, p2=medium, p3=low)
- **Frontend**: Jest with React Testing Library
- **Integration**: Docker-based testing with real services

## Common Development Workflows

1. **Adding new document parser**: Extend `deepdoc/parser/` and register in processing pipeline
2. **Adding new LLM provider**: Implement in `rag/llm/` following existing model patterns
3. **Adding new agent tool**: Create in `agent/tools/` extending base tool class
4. **Adding new API endpoint**: Create app in `api/apps/` with corresponding service in `api/db/services/`

## Storage and Database

- **Primary DB**: PostgreSQL (default) with Peewee ORM, MySQL also supported
- **Document Store**: Infinity vector database (default), Elasticsearch and OpenSearch also supported
- **Object Storage**: MinIO for file storage (port 9002)
- **Cache**: Redis for sessions and caching
- **Vector Search**: Integrated with Infinity or other vector stores