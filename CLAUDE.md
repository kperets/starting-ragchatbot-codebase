# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Setup:**
```bash
uv sync                          # Install dependencies
cp .env.example .env             # Then add ANTHROPIC_API_KEY to .env
```

**Run the application:**
```bash
./run.sh                         # Quick start
# or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

App available at `http://localhost:8000` | API docs at `http://localhost:8000/docs`

**No test or lint commands are configured** — this project has no test suite or linter setup.

## Architecture

Full-stack RAG (Retrieval-Augmented Generation) chatbot. Python 3.13+ required. Uses `uv` for dependency management.

**Backend** ([backend/](backend/)) — FastAPI app served at port 8000; also serves the frontend via StaticFiles.

**Frontend** ([frontend/](frontend/)) — Vanilla HTML/CSS/JS; uses `marked.js` for Markdown rendering.

**Docs** ([docs/](docs/)) — `.txt` course files that are ingested at startup into ChromaDB.

### Backend Module Responsibilities

| Module | Role |
|--------|------|
| [app.py](backend/app.py) | FastAPI entry point; startup document ingestion; `/api/query` and `/api/courses` endpoints |
| [rag_system.py](backend/rag_system.py) | Orchestrates the full RAG pipeline |
| [vector_store.py](backend/vector_store.py) | ChromaDB wrapper; two collections: `course_catalog` (metadata) and `course_content` (chunks) |
| [document_processor.py](backend/document_processor.py) | Parses course files; extracts metadata; chunks text by sentences |
| [ai_generator.py](backend/ai_generator.py) | Anthropic Claude API wrapper; handles tool use and conversation history |
| [search_tools.py](backend/search_tools.py) | Tool definitions and `CourseSearchTool` for Claude's tool_use feature |
| [session_manager.py](backend/session_manager.py) | In-memory conversation session storage (resets on server restart) |
| [config.py](backend/config.py) | Centralizes configuration defaults (model names, chunk sizes, etc.) |
| [models.py](backend/models.py) | Pydantic models: `Lesson`, `Course`, `CourseChunk` |

### Data Flow

1. **Startup:** FastAPI loads `docs/*.txt` → `DocumentProcessor` parses + chunks → stored in ChromaDB
2. **Query:** `POST /api/query` → `RAGSystem` builds prompt → Claude invokes `CourseSearchTool` → ChromaDB semantic search → Claude generates response with sources
3. **Session:** Each user gets a `session_id`; history (max 2 exchanges) is passed to Claude for context

### Key Config Defaults (overridable via `.env`)

- `ANTHROPIC_MODEL`: `claude-sonnet-4-20250514`
- `EMBEDDING_MODEL`: `all-MiniLM-L6-v2`
- `CHUNK_SIZE`: 800 characters, `CHUNK_OVERLAP`: 100
- `MAX_RESULTS`: 5 search results, `MAX_HISTORY`: 2 conversation turns
- `CHROMA_PATH`: `./chroma_db`
