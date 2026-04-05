# 🧠 Production RAG Pipeline

> Cache-Augmented Generation with Context Engineering, Semantic Search, Web Chat UI, and MCP Server for Claude Desktop.

## Diagrams

### Standard RAG Pipeline
![Standard RAG](docs/standard_rag.svg)

### Agentic RAG Pipeline
![Agentic RAG](docs/agentic_rag.svg)

## ✨ Features

| Feature | Description |
|---|---|
| **Two-stage Retrieval** | Page index (broad) + Chunk index (precise) via ChromaDB |
| **Context Engineering** | Query rewriting, MMR re-ranking, compression, token budgeting |
| **Redis Cache** | SHA256-keyed answer cache with configurable TTL |
| **Conversation Memory** | Rolling window with auto-summary compression |
| **Dual LLM** | Google Gemini (cloud) or Ollama (local, no quota) |
| **Web Chat UI** | FastAPI + WebSocket dark UI at `localhost:8000` |
| **MCP Server** | 5 tools exposed to Claude Desktop via stdio |
| **CLI Search** | Interactive semantic search REPL |

## 📁 Project Structure

```
production-ready-ai-solution/
├── src/
│   ├── ingestion/
│   │   └── create_vector_db.py   # Document ingestion → chunk + page indexes
│   ├── retrieval/
│   │   └── search.py             # CLI semantic search (chunk/page/both modes)
│   ├── generation/
│   │   └── rag.py                # CAG pipeline (Redis + ChromaDB + LLM)
│   ├── context/
│   │   └── context_engine.py     # Query rewrite, MMR, compress, token budget
│   └── server/
│       ├── chat_server.py        # Web chat UI (FastAPI + WebSocket)
│       └── mcp_server.py         # MCP server for Claude Desktop
├── data/                         # Drop your documents here
├── docs/
│   └── architecture.svg          # Pipeline architecture diagram
├── scripts/
│   ├── ingest.sh                 # Run ingestion
│   ├── start_chat.sh             # Start web chat
│   └── start_mcp.sh              # Start MCP server
├── chroma_db/                    # Auto-created vector database
├── .env.example                  # Environment variable template
├── requirements.txt              # Python dependencies
└── README.md
```

## 🚀 Quick Start

### 1. Clone & Install

```bash
git clone https://github.com/yourusername/production-ready-ai-solution
cd production-ready-ai-solution

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env with your settings
```

### 3. Add Documents

Drop any `.txt`, `.md`, `.pdf`, `.json`, or `.csv` files into `data/`.

### 4. Ingest Documents

```bash
./scripts/ingest.sh
# or: python src/ingestion/create_vector_db.py
```

### 5. Choose Your Interface

**Web Chat UI:**
```bash
./scripts/start_chat.sh
# Open http://localhost:8000
```

**CLI Search:**
```bash
python src/retrieval/search.py                         # interactive REPL
python src/retrieval/search.py "what is RAG?"          # single query
python src/retrieval/search.py "query" --mode page     # page-level search
python src/retrieval/search.py "query" --mode both     # chunk + page
```

**RAG CLI:**
```bash
python src/generation/rag.py                           # interactive REPL
python src/generation/rag.py "what is RAG?"            # single query
python src/generation/rag.py --flush                   # clear cache
python src/generation/rag.py --stats                   # cache stats
```

**Claude Desktop (MCP):**
```bash
./scripts/start_mcp.sh   # or configure via claude_desktop_config.json
```

## 🔌 MCP Setup (Claude Desktop)

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "rag": {
      "command": "/path/to/.venv/bin/python",
      "args": ["/path/to/src/server/mcp_server.py"],
      "env": {
        "PYTHONPATH": "/path/to/src/generation",
        "GOOGLE_API_KEY": "your-key",
        "PATH": "/path/to/.venv/bin:/usr/local/bin:/usr/bin:/bin"
      }
    }
  }
}
```

**Available MCP Tools:**

| Tool | Description |
|---|---|
| `rag_query` | Full RAG pipeline — cache → retrieve → generate |
| `semantic_search` | Search only, no LLM |
| `list_documents` | List indexed files |
| `cache_stats` | Redis statistics |
| `flush_cache` | Clear cached answers |

## ⚙️ Configuration

Edit constants at the top of each file, or set via `.env`:

| Variable | Default | Description |
|---|---|---|
| `LLM_PROVIDER` | `ollama` | `ollama` or `gemini` |
| `OLLAMA_MODEL` | `llama3.2` | Local Ollama model |
| `GEMINI_MODEL` | `gemini-2.0-flash-lite` | Gemini model |
| `GOOGLE_API_KEY` | — | Required for Gemini |
| `HF_MODEL` | `BAAI/bge-small-en-v1.5` | Embedding model |
| `CHUNK_SIZE` | `200` | Words per chunk |
| `CHUNK_OVERLAP` | `40` | Overlap between chunks |
| `CACHE_TTL` | `3600` | Redis cache TTL (seconds) |
| `MAX_CONTEXT_TOKENS` | `3000` | Token budget for context |
| `MMR_LAMBDA` | `0.7` | Relevance vs diversity (0–1) |
| `COMPRESS_RATIO` | `0.7` | Sentence keep ratio |

## 🏗️ Architecture

```
Query
  │
  ▼
Redis Cache ──HIT──► return cached answer instantly
  │ MISS
  ▼
Context Engine
  ├── QueryRewriter     (resolve pronouns via conversation history)
  ├── Two-stage Retrieve (page index + chunk index)
  ├── MMR Re-rank        (balance relevance vs diversity)
  ├── ContextCompressor  (strip low-relevance sentences)
  ├── TokenBudgetManager (fit within 3000 token window)
  └── SystemPromptBuilder (structured multi-part prompt)
  │
  ▼
LLM (Ollama / Gemini)
  │
  ▼
ConversationMemory (record turn → feeds next QueryRewriter)
  │
  ▼
Cache answer in Redis → return
```

## 🧩 Embedding Models

| Model | Dims | Size | Best For |
|---|---|---|---|
| `BAAI/bge-small-en-v1.5` | 384 | 130MB | Default — best free RAG model |
| `BAAI/bge-base-en-v1.5` | 768 | 440MB | Better quality |
| `BAAI/bge-large-en-v1.5` | 1024 | 1.3GB | Best quality |
| `all-MiniLM-L6-v2` | 384 | 90MB | Lightweight fallback |

## 🛠️ Dependencies

- **chromadb** — vector database
- **sentence-transformers** — local embeddings
- **redis** — answer cache
- **fastapi + uvicorn** — web server
- **mcp** — Model Context Protocol server
- **google-genai** — Gemini API
- **ollama** — local LLM client
- **pypdf** — PDF text extraction

## 📄 License

MIT

## 🕵 Agentic RAG

Unlike standard RAG which blindly retrieves top-k chunks and passes them to an LLM, **Agentic RAG lets the LLM decide what to retrieve, how many times, and when it has enough context.**

```
Query
  ↓
Agent Plans: "What tools do I need?"
  ├── search_chunks(query)       — fine-grained chunk search
  ├── search_pages(query)        — broad page-level search
  ├── filter_by_file(filename)   — search within a specific doc
  ├── summarise_doc(filename)    — get full doc overview
  └── answer(text, confident)    — produce final answer
  ↓
Self-Reflection: "Is my answer complete?"
  ├── YES → return answer + trace
  └── NO  → plan next tool call (up to 5 iterations)
```

**Run Agentic RAG:**
```bash
python src/generation/agentic_rag.py "What are the key differences between HNSW and LSH?"
python src/generation/agentic_rag.py "Summarise all documents about vector databases" --trace
python src/generation/agentic_rag.py --iter 3   # max 3 reasoning loops
```

**Standard vs Agentic RAG:**

| | Standard RAG | Agentic RAG |
|---|---|---|
| Retrieval | Fixed top-k | Dynamic, LLM-driven |
| Tools | None | 5 tools |
| Iterations | 1 | Up to 5 |
| Multi-doc | Merged blindly | Strategically selected |
| Transparency | None | Full reasoning trace |
| Speed | Fast (~1s) | Slower (~3–10s) |
| Complex queries | ❌ Struggles | ✅ Handles well |

Use **Standard RAG** for simple factual questions. Use **Agentic RAG** for complex multi-part questions, comparisons across documents, or when you need to see the reasoning process.

## 🧠 Key Algorithms Explained

**TF-IDF** — Scores words by rarity across documents. Used for offline embedding fallback and context compression (strip low-TF-IDF sentences before sending to LLM).

**HNSW** (Hierarchical Navigable Small World) — Multi-layer graph index inside ChromaDB. Achieves O(log n) approximate nearest-neighbor search instead of O(n) brute force.

**LSH** (Locality Sensitive Hashing) — Hashes similar vectors to the same bucket. Used as a pre-filter to generate candidates before HNSW refinement.

**MMR** (Maximal Marginal Relevance) — Balances relevance vs diversity in retrieval results. Prevents three chunks from the same paragraph dominating the context.

**CAG** (Cache-Augmented Generation) — Redis cache layer intercepts the pipeline. Identical queries return instantly without touching the vector DB or LLM.
