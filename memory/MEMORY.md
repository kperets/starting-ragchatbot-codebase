# Project Memory

## Preferences
- Always use `uv` to run the server and manage dependencies. Never use `pip` directly.
  - Run server: `cd backend && uv run uvicorn app:app --reload --port 8000`
  - Install deps: `uv sync`
