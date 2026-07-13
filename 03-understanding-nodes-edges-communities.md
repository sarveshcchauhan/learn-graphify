# Tutorial 3: Understanding Nodes, Edges, and Communities

## Use Case Scenarios

**Scenario A — Architecture review:** Your team lead asks: "What does the `PaymentService` class depend on?" You open graphify's graph, find the node, and see every edge — no code navigation needed.

**Scenario B — Dependency audit:** Before upgrading a library, you need to know every file that imports it. The graph's community structure shows you all affected modules in one view.

**Scenario C — Team onboarding:** A new developer asks: "How does authentication work here?" Instead of tracing 12 files manually, you show them the auth community — 25 nodes, all connected.

## What Are Nodes?

A **node** is any named entity in your codebase. Examples:

- Files — `server.js`, `authMiddleware.js`
- Functions — `validateToken()`, `handlePayment()`
- Classes — `OrderController`, `UserModel`
- React components — `<LoginPage />`, `<Navbar />`
- Document concepts — "Authentication Flow", "Refund Policy"

Each node has attributes:

```json
{
  "id": "server_controllers_payment_paymentcontroller",
  "label": "PaymentController",
  "file_type": "code",
  "source_file": "server/controllers/paymentController.js",
  "source_location": "1:1"
}
```

### Node ID Format

graphify uses a deterministic ID scheme: `{repo-relative-path}_{entity-name}`, with all path segments and the name lowercased and non-alphanumeric characters replaced by underscores.

Example: `src/utils/helpers.js` + `parseUrl` → `src_utils_helpers_parseurl`

This ensures the same entity always gets the same ID, even across re-extractions.

### Node Types

| `file_type` | Meaning | Examples |
|-------------|---------|---------|
| `code` | A code symbol (function, class, variable) | `validateToken`, `UserModel` |
| `document` | A concept from a .md or .txt file | "Installation Guide", "API Design" |
| `paper` | A concept from a PDF/academic paper | "Transformer Architecture" |
| `image` | Content extracted from an image via vision | "Login Screen UI", "Architecture Diagram" |
| `rationale` | A design decision or reason | "Why we chose PostgreSQL over MongoDB" |
| `concept` | Abstract idea, pattern, or principle | "Dependency Injection", "JWT Auth" |

## What Are Edges?

An **edge** is a relationship between two nodes. Every edge has:

- **source** — the node doing the connecting
- **target** — the node being connected to
- **relation** — the type of relationship
- **confidence** — how sure graphify is about this edge
- **confidence_score** — a numeric value (0.0 to 1.0)

### Edge Types

| Relation | Meaning | Example |
|----------|---------|---------|
| `calls` | Function A calls function B | `validateToken` → `jwt.verify` |
| `imports` | Module A imports module B | `server.js` → `authRoutes` |
| `references` | File A mentions file B | README → installation section |
| `cites` | Doc A cites doc B | "See §3.2 for details" |
| `implements` | Class A implements interface B | `MongoUserRepo` → `UserRepository` |
| `conceptually_related_to` | A and B are related by topic | "JWT" → "Session Management" |
| `shares_data_with` | A and B use the same data | `UserModel` → `AuthController` |
| `semantically_similar_to` | A and B solve the same problem | Two validation functions in different modules |

### Confidence Levels

| Level | Score | Meaning |
|-------|-------|---------|
| `EXTRACTED` | 1.0 | Certain — directly from code (import, call) |
| `INFERRED` | 0.55–0.95 | Reasonable inference (shared data, functional alignment) |
| `AMBIGUOUS` | 0.1–0.3 | Uncertain, flagged for human review |

### How Edges Are Extracted

**AST extraction** (code-only, no LLM):
- `imports` — from `import` and `require` statements
- `calls` — from function call expressions (same-language only — cross-language calls are phantom artifacts and never emitted)
- `implements` — from `class Foo implements Bar` and `extends`
- `references` — from JSDoc/TSDoc `@see` and `@link` tags

**Semantic extraction** (docs/papers/images, LLM-powered):
- `cites` — explicit citations ("as discussed in §2.1")
- `conceptually_related_to` — thematic relationships
- `semantically_similar_to` — same problem, different solutions
- `rationale_for` — design intent ("We chose X because...")

### Cross-Language Rule

**Critical:** graphify never creates `calls` edges across languages. A PHP function cannot `calls` a JavaScript function and vice versa — those are phantom artifacts. Cross-language connections use `references` or `conceptually_related_to` instead.

## What Are Communities?

A **community** is a cluster of nodes that are more connected to each other than to the rest of the graph. graphify uses community detection algorithms (Louvain/Leiden) to automatically discover these groups.

### How Communities Form

Community detection examines the edge structure:
1. Nodes with many connections between them cluster together
2. Boundaries form where connectivity drops off
3. Each community is assigned a unique ID and label

### Real-World Examples from a MERN App

| Community | Nodes | Typical Contents |
|-----------|-------|-----------------|
| Authentication Flow | 25 | auth routes, middleware, JWT utils, login page, user model |
| Payment Processing | 18 | Razorpay integration, order model, webhook handler, invoices |
| Frontend UI | 42 | React components, pages, shared layout, CSS modules |
| API Routes | 31 | Express controllers, route definitions, validation middleware |
| Data Models | 15 | MongoDB schemas, relationships, migration scripts |
| Email System | 8 | email templates, mailer utility, subscription handlers |

### Cohesion Score

Each community has a **cohesion score** (0 to 1) measuring how tightly integrated it is:

- **0.8–1.0:** Very tight — all nodes are interconnected (e.g., a single module's internal functions)
- **0.5–0.8:** Moderate — clear grouping with some external connections
- **0.2–0.5:** Loose — nodes are more connected to outside than each other (possible over-splitting)

### God Nodes

**God nodes** are the most central nodes in the graph. They have the highest degree (most connections). In any codebase, touching a god node means touching everything:

```
Top 5 God Nodes:
  1. server.js (degree: 47) — mounts all routes, all middleware
  2. db.js (degree: 38) — every model imports the connection
  3. authMiddleware.js (degree: 31) — protects all routes
  4. App.jsx (degree: 29) — renders every page component
  5. config.js (degree: 26) — env vars used everywhere
```

### Surprising Connections

graphify surfaces edges that cross community boundaries — connections no one expected:

> "A jQuery carousel plugin (Frontend community) connects to a testimonial data model (Data community) because the carousel renders dynamic testimonials."

These are **refactoring risks** and **onboarding gotchas** — the kind of hidden coupling that breaks production.

---
