# Tutorial 1: Installing and Configuring graphify

## What is graphify?

graphify turns any folder of code, docs, PDFs, images, or videos into a queryable knowledge graph. It extracts entities and relationships, detects communities, and generates an interactive visualization — all without a single API key for code-only projects.

## Use Case Scenarios

**Scenario A — Onboarding to a legacy codebase:** You just joined a team with a 50,000-file monolith. Instead of reading files one by one, run graphify once and explore the architecture visually.

**Scenario B — Documenting a side project:** You maintained a 20-file MERN app over the weekend. Run graphify to see how everything connects before you forget.

**Scenario C — Pre-refactor audit:** Your team plans to rip out an old auth system. graphify shows you every file that touches it — no grep firehose.

## Prerequisites

- Python 3.10 or later installed (`python3 --version`)
- pip or uv (recommended) for package management

## Installation

### Option 1 — Install via uv (recommended, fastest)

```bash
uv tool install graphifyy
```

This installs graphify as an isolated tool. The `graphify` command becomes available globally.

### Option 2 — Install via pip

```bash
pip install graphifyy
```

If you're on a system that blocks `--break-system-packages`:

```bash
pip install graphifyy --break-system-packages
```

Verify the installation:

```bash
graphify --help
```

You should see the full usage guide with all commands and flags.

## Configuration

graphify requires **zero configuration** for most use cases. It works out of the box on any folder.

There are only three optional settings:

| Setting | How to set | Purpose |
|---------|-----------|---------|
| API key for semantic extraction | `export GEMINI_API_KEY="your_key"` | Enables LLM-powered extraction for docs, PDFs, and images (skip for code-only projects) |
| Whisper model for video | `export GRAPHIFY_WHISPER_MODEL=medium` | Change transcription accuracy/speed (default: `base`) |
| Gemini model for LLM | `export GRAPHIFY_GEMINI_MODEL=gemini-2.5-flash` | Override the LLM used for semantic extraction |

### When do you need semantic extraction?

graphify has two extraction modes:

1. **Structural (AST) — free, deterministic, no API needed:** Runs on all code files (.py, .ts, .js, .go, .rs, .java, .cpp, .rb, .php, etc.). Extracts imports, classes, functions, and call relationships directly from the syntax tree.

2. **Semantic (LLM) — optional, for non-code content:** Extracts concepts, entities, and cross-document connections from docs (.md, .txt), PDFs, and images. Falls back to the host AI if no API key is set.

**For a code-only project**, you never need an API key. graphify's AST extraction handles everything.

**For projects with docs, papers, or images**, semantic extraction is richer with a Gemini API key set:
```bash
pip install 'graphifyy[gemini]'
export GEMINI_API_KEY="your_key_here"
```

## Verifying the Setup

Run graphify on a small test directory to confirm everything works:

```bash
mkdir -p test-project
echo 'function hello() { return "world"; }' > test-project/app.js
graphify ./test-project
```

Expected output includes the corpus summary, extraction counts, clustering results, and the final output paths. You should see `graph.html` appear in `test-project/graphify-out/`.

---
