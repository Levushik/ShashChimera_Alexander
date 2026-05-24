# ShashChimera_Alexander

`ShashChimera_Alexander` is a chess analysis and commentary project that combines the **Alexander** engine, Shashin-style evaluation zones, BM25-based retrieval, and an LLM layer to turn raw engine output into readable move-by-move explanations.
The repository includes the core Python package, evaluation scripts, tests for retrieval and commentary robustness, and a full-stack web application for interactive PGN/FEN analysis.

## Features

- Engine-driven analysis through the `alexander_interpreter` package, including engine orchestration, evaluation parsing, Shashin zone labeling, opening knowledge, prompting, and verbalization logic.
- Retrieval-augmented commentary with BM25 search over chess knowledge chunks plus opening-theory lookup for book-adjacent positions.
- LLM commentary generation with retry safeguards for empty or think-only outputs, based on the test suite and evaluation scripts in `tests/`.
- A web application that streams engine analysis and commentary to the frontend with Server-Sent Events for PGN and FEN inputs.
- Utility scripts and notebooks for smoke testing, regression testing, and deeper commentary evaluation workflows.

## Repository layout

```text
.
├── alexander_interpreter/   # Core package: engine, retrieval, prompting, verbalization
├── webapp/                  # FastAPI + frontend application for interactive analysis
├── tests/                   # Unit tests and end-to-end evaluation scripts
├── scripts/                 # Smoke tests and helper scripts
├── archive/                 # Older notes / archived materials
└── requirements.txt         # Python dependencies
```

## Core components

### `alexander_interpreter/`

The main package contains the project logic for parsing engine output, assigning Shashin-style strategic zones, retrieving opening knowledge, and generating natural-language commentary around each position.
Important modules visible in the repository include `engine.py`, `eval_parser.py`, `retriever.py`, `opening_book.py`, `opening_theory_kb.py`, `shashin.py`, `prompt.py`, `llm.py`, and `verbalizer.py`.

### `webapp/`

The web application is described as a full-stack analyzer that accepts PGN or FEN input, runs sequential engine analysis, and then launches concurrent commentary generation through an LLM API.
Its documented architecture uses a FastAPI backend, Server-Sent Events streaming, an Alexander engine subprocess, and an OpenAI-compatible LLM endpoint such as LM Studio, Ollama, or vLLM.

### `tests/`

The test suite mixes lightweight pytest-based unit tests with full evaluation scripts that require a real engine and LLM endpoint.
Documented tests cover BM25 retrieval, opening-book lookup, phase detection, commentary retry behavior, anomaly detection, and Deepeval-based commentary checks.

## Installation

### Prerequisites

- Python 3.12 recommended by the documented test commands.
- Access to the Alexander engine binary for full analysis workflows.
- An OpenAI-compatible local or remote LLM endpoint for commentary generation, such as LM Studio.

### Install dependencies

```bash
python3.12 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Current declared Python dependencies are `httpx`, `rank-bm25`, and `deepeval`.

## Quick start

### Run unit tests

```bash
python3.12 -m pytest tests/ -v
```

This runs the lightweight test suite without requiring the chess engine or an LLM backend.

### Run a full game evaluation

```bash
python tests/eval_game.py
python tests/eval_game.py --pgn my_game.pgn --config medium --out trace.json
```

The evaluation script runs a real game through Alexander and an LLM, then records traces including prompts, retries, engine outputs, and final commentary.

### Run the web app

Start from the `webapp/` directory and follow its local or Docker setup, as the included app is designed for interactive analysis of arbitrary games and positions.
A dedicated `webapp/README.md` already documents architecture, backend/frontend roles, local startup, Docker deployment, configuration, and API behavior.

## How it works

1. Input is provided as a PGN game or a FEN position through scripts or the web app.
2. The system parses positions and sends them to the Alexander engine through a UCI subprocess interface.
3. Engine output is converted into structured fields such as centipawn or mate scores, best moves, WDL estimates, principal variations, and Shashin zones.
4. Retrieval modules add opening-book or theory context, while the prompt builder assembles a compact explanation request for the LLM.
5. The LLM returns move commentary, and retry logic handles truncated or think-only responses before final text is emitted or streamed.

## Configuration

The repository exposes configuration through package modules such as `config.py`, and the tests mention environment variables like `ALEXANDER_ENGINE_PATH` and `LM_STUDIO_URL` for engine and model connectivity.
For the web application, consult `webapp/README.md` for exact runtime and deployment settings.

## Development notes

- Use `scripts/smoke_test_alexander.py` to verify the engine path and basic engine integration.
- Use `tests/test_retriever.py` to validate retrieval logic independently from the LLM stack.
- Use `tests/test_commentary_gaps.py` to regression-test commentary retry behavior and empty-output handling.
- Use the notebooks in `tests/` for deeper analysis of evaluation traces and commentary quality.

## Suggested environment variables

```bash
export ALEXANDER_ENGINE_PATH=/path/to/alexander
export LM_STUDIO_URL=http://localhost:1234
```

These names are explicitly referenced in the repository test documentation for full evaluation runs.

## Roadmap ideas

- Add a top-level CLI entry point for PGN/FEN analysis.
- Document expected engine build steps and supported binaries in the main README.
- Add sample screenshots or GIFs for the web application.
- Provide a minimal `.env.example` for engine and LLM configuration.
- Include an example analysis output JSON for quicker onboarding.

## Related docs

- See [`webapp/README.md`](./webapp/README.md) for the application architecture and deployment details.
- See [`tests/README.md`](./tests/README.md) for test categories and evaluation scripts.
- See [`archive/README.md`](./archive/README.md) for archived project notes.

## License

No license file was visible in the inspected repository snapshot, so usage terms should be clarified before external redistribution or commercial reuse.
