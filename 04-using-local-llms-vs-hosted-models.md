# Tutorial 4: Using Local LLMs vs Hosted Models

## Use Case Scenarios

**Scenario A — No internet, no problem:** You're working on an air-gapped government project. All your code, docs, and PDFs must stay local. graphify runs AST extraction without any API call. For semantic extraction (docs/images), it uses the host AI that's already in your terminal session — no external service needed.

**Scenario B — Rich semantic analysis:** You have a Gemini API key. graphify uses Gemini to extract deep relationships from your docs, with vision analysis for images and screenshots.

**Scenario C — Hybrid team setup:** Your CI pipeline runs graphify on every commit. CI machines have no API keys, so they get AST-only extraction (fast, free). Developers run with a Gemini key on their local machines for richer semantic results.

## How graphify Handles LLMs

graphify has a clear separation of concerns:

### No API Key Mode (Default)

**graphify needs no API key. Never blocks on one.**

For code-only projects, graphify uses **AST extraction** — deterministic, free, zero API calls. It parses the syntax tree of every code file and extracts:
- Classes, functions, methods
- Import/export relationships
- Call expressions (same-language only)
- Interface implementations

This covers 100% of what you need for code navigation. No LLM required.

For projects with docs, papers, or images, graphify's **semantic extraction** falls back to the **host AI** — the model running the terminal session (e.g., Claude, GPT, DeepSeek). This happens automatically. You don't need to configure anything.

### Gemini API Key Mode (Enhanced)

When you set a Gemini API key, semantic extraction uses Gemini directly:

```bash
pip install 'graphifyy[gemini]'
export GEMINI_API_KEY="your_key_here"
```

Benefits of Gemini mode:

| Feature | Host AI (no key) | Gemini (with key) |
|---------|-----------------|-------------------|
| Code extraction | AST (free) | AST (free) |
| Doc extraction | Host AI | Gemini |
| Image/vision analysis | Host AI (limited) | Gemini vision (screenshots, diagrams) |
| Speed | Limited by host context | Parallel subagent dispatch |
| Cost | Free (included in session) | Gemini API cost |
| Privacy | Depends on host | Data sent to Gemini |

### API Key Detection

graphify checks for these environment variables in order:
1. `GEMINI_API_KEY`
2. `GOOGLE_API_KEY`

If neither is set, graphify prints a one-liner hint:

> Tip: set `GEMINI_API_KEY` or `GOOGLE_API_KEY` to use Gemini for semantic extraction (`pip install 'graphifyy[gemini]'`)

Then continues anyway — it never waits for a key.

### Important: What graphify Does NOT Support

graphify reads **only these** API keys:
- `GEMINI_API_KEY` / `GOOGLE_API_KEY` (for Gemini)

It does **NOT** read:
- `ANTHROPIC_API_KEY` (Claude)
- `OPENAI_API_KEY` (GPT)
- `DEEPSEEK_API_KEY`
- Any other provider key

If you catch yourself prompting for a non-Gemini key, that's a misread — graphify doesn't use them.

## Choosing the Right Mode

### When to Use AST-Only (No Key)

✅ **Your project is code-only** (no docs, PDFs, or images)
✅ **You need fast results** — AST is instant, LLM takes seconds per file
✅ **You're in an air-gapped or offline environment**
✅ **You're running graphify in CI** — every commit gets a free architecture snapshot
✅ **Privacy is critical** — all data stays on your machine

### When to Use Gemini Key

✅ **You have architecture documents, READMEs, or design docs** you want analyzed
✅ **Your project has UI screenshots or diagrams** you want interpreted via vision
✅ **You want deeper semantic relationships** — "why" connections beyond "what imports what"
✅ **You have academic PDFs or research papers** in your corpus

## Performance Comparison

| Dimension | AST Only | AST + Gemini |
|-----------|----------|-------------|
| Time (147-file MERN) | ~15 seconds | ~45 seconds |
| Nodes extracted | ~600 | ~850 |
| Edges extracted | ~900 | ~1,574 |
| Communities detected | ~60 | ~109 |
| Cost | $0 | ~$0.02 (10K in / 25K out tokens) |
| API dependency | None | Requires internet + Gemini key |

## Practical Example

### Running Without a Key

```bash
cd my-project
graphify .
# AST extraction runs automatically
# No API key prompt. No wait. No configuration.
```

Output:
```
Corpus: 147 files
  code: 147 files (no docs/images — semantic extraction skipped)
AST extraction: 613 nodes, 892 edges
Graph: 613 nodes, 892 edges, 58 communities
```

### Running With Gemini Key

```bash
export GEMINI_API_KEY="your_key_here"
pip install 'graphifyy[gemini]'
graphify .
```

Output:
```
Corpus: 147 files, ~89,000 words
  code: 126 files
  docs: 12 files
  images: 9 files
AST extraction: 613 nodes, 892 edges
Semantic extraction: 235 additional nodes, 682 additional edges
Graph: 848 nodes, 1574 edges, 109 communities
```

## Hybrid Workflow (Recommended)

1. **In CI:** Run graphify with `--no-viz` flag on every commit. AST-only. Fast, free, and catches architecture drift.

2. **Locally:** Keep a Gemini key in your `.env` file or shell profile. Run `graphify --update` when you change docs or add features.

3. **For onboarding:** Run `graphify --mode deep` with a Gemini key to get the richest graph possible, then share the `graph.html` and `GRAPH_REPORT.md`.

---
