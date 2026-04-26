# Chess Analysis Agent

Standalone LLM agent: takes data from a chess engine and returns an English explanation of the position.  
Designed for qwen3-0.6b via LM Studio. Runs locally, without internet.

***

## Architecture

```text
EngineResult (mock / real UCI)
        │
        ▼
  retriever.py  ──── BM25 ────► knowledge_base.py
        │                        (28 theory chunks)
        ▼
   prompt.py   ──── builds prompt (~150 tokens)
        │
        ▼
    llm.py     ──── POST /v1/chat/completions ──► LM Studio
        │
        ▼
   str response  (think-tags removed)
```

**Data flow:**

| File             | Role                                                                                  |
|------------------|---------------------------------------------------------------------------------------|
| `mock_engine.py` | Input data: eval, best_move_san, WDL, shashin_type                                    |
| `retriever.py`   | BM25 search over knowledge_base, top-2 chunks                                         |
| `shashin.py`     | Dictionary of position types (Tal/Capablanca/Petrosian → text)                        |
| `prompt.py`      | Prompt construction; FEN is intentionally excluded                                    |
| `llm.py`         | LM Studio HTTP client, strips `<think>`                                               |
| `config.py`      | All constants via environment variables                                               |

***

## Input data (`EngineResult`)

```python
EngineResult(
    fen="...",             # only for identifying the position; NOT sent in the prompt
    best_move_uci="e1g1",  # UCI — used only for leak-checks in tests
    best_move_san="O-O",   # SAN — the only notation used in the prompt and answer
    score_cp=200,          # centipawns; None if mate
    mate_in=1,             # plies to mate; None if no forced mate
    wdl_win=620,           # 0–1000, sum = 1000
    wdl_draw=310,
    wdl_loss=70,
    depth=22,
    shashin_type="Tal",    # "Tal" | "Capablanca" | "Petrosian"
    side_to_move="white",  # "white" | "black"
)
```

User parameters: `level` (beginner/intermediate/advanced), `question` (best_move/explain/plan), `moves_history` (recent moves in SAN).

***

## Extending the knowledge base

Add a chunk to `knowledge_base.py`:

```python
{
    "id":   "unique_id",
    "tags": ["topic", "Shashin-type", "BM25 keywords"],
    "text": "A principle written as a single paragraph in English.",
},
```

**Rules:**

- One chunk = one principle, length 2–5 sentences.
- `tags` affect only readability; BM25 searches over `text`.
- The more specific the terms in `text`, the better the retrieval (write `"isolated pawn"`, `"rook on 7th rank"`, not `"weak piece"`).
- The BM25 index is rebuilt automatically when `retriever.py` is imported.

***

## Strengths

- **No hallucinations about the board**: the model does not see the FEN and does not “read” the position; it only sees structured facts from the engine.
- **Dependency-free BM25**: `rank_bm25` uses ~1 MB RAM and runs on any phone.
- **Short prompt** (~150 tokens): fits comfortably into the context window of a 0.6b model.
- **Easy to test**: `smoke_test.py --dry-run` does not require LM Studio.

## Weaknesses

- **Answer quality depends on input quality**: if the engine provides a bad `best_move_san`, that is exactly what the model will explain.
- **BM25 is not semantic**: a query like `"endgame with rook"` will not find a chunk about the `"Philidor position"` if the word `"rook"` is not present there; this is mitigated by adding synonyms into the chunk `text`.
- **`shashin_type` is currently heuristic**: in `mock_engine.py` it is computed from `score_cp`, not from real ShashChess output; when integrating with the engine, you should parse the UCI output instead.
- **The 0.6b model does not validate move legality**: it explains the move it is given; an incorrect `best_move_san` from the engine will not be challenged.

***

## Running

```bash
pip install httpx rank-bm25

# tests without LM Studio
python3 smoke_test.py --dry-run

# tests with a live model → writes smoke_results.md
python3 smoke_test.py

# custom report file
python3 smoke_test.py -o my_report.md
```

Configuration via environment variables: `ENGINE_PATH`, `LM_STUDIO_URL`, `MODEL_NAME`, `MAX_LLM_TOKENS`, `ENGINE_DEPTH`.
