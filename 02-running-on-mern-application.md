# Tutorial 2: Running graphify on a Real MERN Application

## Use Case Scenarios

**Scenario A — Full-stack visibility:** Your MERN app has 147 files across 5 directories. You need to understand how the React frontend talks to Express routes, which MongoDB models are used where, and where the auth middleware is applied — without reading every file.

**Scenario B — CI onboarding artifact:** Every new hire runs graphify on the main repo during their first week. They get an interactive graph + a GRAPH_REPORT.md that maps the entire architecture before they write their first line of code.

**Scenario C — Pre-refactor safety net:** Before migrating from REST to GraphQL, run graphify to snapshot every REST endpoint, every controller that calls it, and every frontend component that consumes it.

## What Happens When You Run graphify on a MERN Stack

graphify processes each file type differently:

| File Type | Extension | Extraction Method |
|-----------|-----------|-------------------|
| JavaScript/TypeScript | .js, .jsx, .ts, .tsx | AST — classes, functions, imports, exports |
| Markdown | .md | Semantic (LLM) — concepts, decisions, architecture |
| JSON | .json | AST — data structures, config schemas |
| Images | .png, .jpg | Semantic (vision) — UI screenshots, diagrams |

## Step-by-Step: Running on a MERN App

### Step 1 — Navigate to the project root

```bash
cd ~/projects/my-mern-app
```

The project structure might look like:

```
my-mern-app/
  client/          # React frontend
    src/
      components/
      pages/
      App.jsx
      index.js
  server/          # Express backend
    models/
    routes/
    controllers/
    middleware/
    server.js
  package.json
  README.md
```

### Step 2 — Run graphify

```bash
graphify .
```

This single command triggers the full pipeline.

### Step 3 — Observe the detection phase

graphify first scans every file and prints a summary:

```
Corpus: 147 files · ~89,000 words
  code:     126 files (.js .jsx .ts .json)
  docs:      12 files (.md)
  images:     9 files (.png .jpg)
```

If the project is smaller than 200,000 words and under 500 files, graphify proceeds directly. If larger, it asks which subdirectory to process.

### Step 4 — AST extraction (code files)

graphify parses all 126 code files. For each file it extracts:

- **Nodes:** Each class, function, React component, Express route handler, and MongoDB model becomes a node.
- **Edges:** Import statements, function calls, and JSX component references become edges.

Example output:
```
AST: 613 nodes, 892 edges
```

### Step 5 — Semantic extraction (docs and images)

If you have a Gemini API key set, graphify analyzes the 12 .md files and 9 images. README files, architecture docs, and UI screenshots are processed to extract:

- Architecture concepts and decisions
- UI layout patterns from screenshots
- Cross-document references

If no API key is set, graphify skips this step for docs/images and uses the host AI instead.

### Step 6 — Graph building and clustering

graphify merges AST and semantic extraction into one graph. It detects communities of related nodes:

```
Graph: 848 nodes, 1574 edges, 109 communities
```

A MERN app typically produces communities like:

- **Authentication flow** (models → middleware → routes → frontend login page)
- **Payment processing** (Razorpay integration, order models, webhooks)
- **Frontend UI** (React components, pages, shared layouts)
- **API routes** (Express routes, controllers, validation)
- **Data models** (MongoDB schemas, relationships)

### Step 7 — Health check

graphify runs a read-only integrity gate:

```
Graph health: OK (no dangling/missing/collapsed edges).
```

If issues are found (dangling references from a deleted file, etc.), they're surfaced as warnings but don't block the build.

### Step 8 — Community labeling

Each community is auto-assigned a label like "Authentication & Session Management" or "Frontend UI Components". You can relabel them manually by editing the labels dict in Step 5 of the pipeline.

### Step 9 — Export outputs

graphify generates:

```
Graph complete. Outputs in /path/to/project/graphify-out/
  graph.html            - interactive graph, open in browser
  GRAPH_REPORT.md       - audit report with stats and insights
  graph.json            - raw graph data (GraphRAG-ready)
```

## Real Output: A 147-File MERN Example

When we ran graphify on an actual 147-file MERN project, here's what surfaced:

| Metric | Value |
|--------|-------|
| Nodes | 848 |
| Edges | 1,574 |
| Communities | 109 |
| God nodes detected | 5 |
| Surprising connections | 3 |
| AST extraction cost | Free (no API key) |
| AST + semantic tokens | ~10K input, ~25K output |

### God Nodes Discovered

The 5 nodes with the highest centrality (everything touches them):

1. `server.js` — Express app setup, middleware registration, route mounting
2. `db.js` — Database connection, reused by every model
3. `authMiddleware.js` — Token verification, applied to protected routes
4. `App.jsx` — React root component, renders all pages
5. `paymentRoutes.js` — Checkout, webhook, order status endpoints

### Surprising Connections

graphify found edges no one expected:

- A jQuery carousel plugin in the frontend linked to the testimonial data model (the carousel renders dynamic testimonials)
- A video transcript connected to an email template (both referenced the same campaign concept)
- The refund policy modal shared a data dependency with the orders table

## Interpreting the Results

In `GRAPH_REPORT.md`, you'll find:

- **Community cohesion scores** — how tightly connected each community is (0 to 1)
- **God nodes** — critical files that everything depends on (high-risk refactor targets)
- **Surprising connections** — cross-cutting edges that suggest hidden coupling
- **Suggested questions** — the most insightful queries the graph can answer

---
