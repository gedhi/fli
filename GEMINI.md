# Fli - Flight Search MCP Server & Library

A reverse-engineered Python wrapper for the Google Flights API, providing programmatic access to flight data with an elegant CLI and Model Context Protocol (MCP) server.

## Project Overview

- **Core Technology**: Python 3.10+
- **Network Stack**: `curl-cffi` for browser-impersonating requests (to bypass simple bot detection).
- **Data Validation**: `pydantic` for robust request/response modeling.
- **CLI Framework**: `typer` for the command-line interface.
- **MCP Framework**: `fastmcp` for the Model Context Protocol server.
- **Architecture**:
    - `fli/search/`: Core API interaction logic. `client.py` handles rate limiting (10 req/s) and retries.
    - `fli/models/`: Pydantic models defining the API and CLI contracts.
    - `fli/mcp/`: MCP server implementation exposing tools to AI assistants.
    - `fli/core/`: Reusable parsing and building logic shared between CLI and MCP.

## Building and Running

### Prerequisites
- Python 3.10 or higher.
- [uv](https://github.com/astral-sh/uv) (recommended) for dependency management.

### Installation
```bash
# Install all dependencies including MCP and Dev extras
uv sync --all-extras
```

### Running the CLI
```bash
# Basic flight search
uv run fli flights JFK LHR 2026-10-25
```

### Running the MCP Server
```bash
# Run on STDIO (for Claude Desktop)
uv run fli-mcp

# Run over HTTP (for remote clients)
uv run fli-mcp-http
```

## Testing and Development

### Running Tests
The project uses `pytest` for testing. You can run tests using `uv` directly or via the `Makefile`.

```bash
# Run all standard tests
uv run pytest
# OR
make test

# Run tests with MCP integration
make test-mcp

# Run fuzzing tests
make test-fuzz

# Run ALL tests (standard + MCP + fuzzing)
make test-all

# Run tests in parallel
uv run pytest -n auto
```

### Linting and Formatting
```bash
# Check code style with Ruff
uv run ruff check .

# Format code with Ruff
uv run ruff format .
```

### Local CI
```bash
# Run the full CI pipeline locally (requires 'act')
make ci
```

## Development Conventions

- **Concurrency & Rate Limiting**: Google Flights API has a global ceiling of 10 requests per second. The `fli.search.client.Client` uses a `TokenBucketRateLimiter` to enforce this across threads.
- **Performance**: Heavy dependencies like `curl-cffi` are imported lazily to ensure the CLI remains responsive for commands that don't require network access.
- **Thread Safety**: `curl_cffi` sessions are not thread-safe and are managed via `threading.local()` in the search client.
- **Error Handling**: Use the typed exceptions in `fli.search.exceptions`. The CLI and MCP server are designed to surface clean error messages to the user while logging full tracebacks.
- **Models**: Always use Pydantic models for data interchange to ensure type safety and validation.
- **Documentation**: Use `mkdocs` for documentation.
    ```bash
    uv run mkdocs serve
    ```

## Key Files
- `fli/search/client.py`: Singleton HTTP client with rate limiting and retries.
- `fli/mcp/server.py`: Definition of MCP tools (`search_flights`, `search_dates`, `find_airports`).
- `fli/models/google_flights/flights.py`: Main data models for flight results.
- `pyproject.toml`: Project metadata, dependencies, and tool configurations.
