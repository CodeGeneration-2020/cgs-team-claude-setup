# Research: CFC Homepage

**Feature**: 001-homepage | **Date**: 2026-03-17

## R1: Semantic Search Approach

**Decision**: Use an AI embedding service to generate vector embeddings for project data, combined with a vector similarity search for natural-language queries.

**Rationale**: The spec requires semantic search that interprets intent beyond keyword matching (US2). Vector embeddings capture semantic meaning and support the requirement that similar queries with different wording return overlapping results (SC-003: 70% overlap). This is the industry-standard approach for semantic search in 2026.

**Alternatives Considered**:
- **Full-text search (PostgreSQL tsvector)**: Fast and simple but only supports keyword matching, not semantic intent. Fails US2 acceptance criteria.
- **Hybrid approach (keyword + vector)**: Adds complexity but could improve precision. Deferred — start with pure semantic, add keyword boosting later if relevance scores are insufficient.
- **LLM-based query rewriting + keyword search**: Uses an LLM to expand the query into keywords, then runs traditional search. Simpler infrastructure but less accurate for nuanced queries and adds latency.

**Implementation Notes**:
- Generate embeddings for project fields: title + description + thematic area + location.
- Store embeddings alongside project records (pgvector extension or dedicated vector store).
- At query time: embed the user's query, perform cosine similarity search, return top-N results.
- Relevance score from similarity is used as the default sort order.

---

## R2: Search Ambiguity Detection

**Decision**: Use query length and result confidence thresholds to detect ambiguous or overly broad queries.

**Rationale**: US2 requires the system to prompt users to refine when queries are too broad (e.g., "help"). Rather than building a complex NLU classifier, use practical heuristics: very short queries (< 3 meaningful words) or queries where the top results have low similarity scores (< configurable threshold) trigger a refinement prompt.

**Alternatives Considered**:
- **LLM-based intent classification**: Accurate but adds latency and cost per query. Over-engineered for the initial release.
- **No detection (always return results)**: Simpler but fails the spec requirement to prompt refinement for broad queries.

---

## R3: Filter Implementation Strategy

**Decision**: Filters are applied as post-search database-level filters on the AI search result set.

**Rationale**: The spec requires that filters narrow AI search results without re-running the search (FR-007). The approach: run the semantic search to get a ranked list of project IDs, then apply SQL WHERE clauses for the selected filters. This preserves the AI relevance ranking while constraining results.

**Alternatives Considered**:
- **Client-side filtering**: Filter the full result set in the browser. Simple but doesn't scale — with thousands of projects, sending all results to the client is impractical.
- **Re-run search with filter context in the query**: Append filter terms to the AI query. Could alter relevance ranking unpredictably and violates the spec requirement that filters don't re-run the search.

**Implementation Notes**:
- Pagination is server-side. Filters + sort are query parameters on the search results endpoint.
- Filter options are populated from a distinct values query on the projects table.

---

## R4: Trending Projects Storage

**Decision**: Use a dedicated `trending_projects` join table with a `display_order` column.

**Rationale**: The spec requires Super Admins to add, remove, and reorder trending projects (US8). A join table with ordering is the simplest model that supports all CRUD operations. The trending list is small (likely < 20 items), so ordering operations are trivial.

**Alternatives Considered**:
- **Boolean flag on projects table**: Simple for add/remove but doesn't support ordering.
- **JSON array column**: Supports ordering but makes it harder to enforce referential integrity and query joins.

---

## R5: Project Card Shared Component

**Decision**: Build a shared `ProjectCard` component in `/packages/ui-components` used by both the search results and trending section.

**Rationale**: Constitution Principle IX (Shared-Before-Custom) requires shared UI components for reusable elements. The project card appears in search results (US3), trending section (US6), and will be reused by other features (bookmarks, dashboard). A single shared component ensures visual consistency.

**Implementation Notes**:
- Component accepts a `ProjectCardData` type from `/packages/shared-types`.
- Visual tags (US7) are a sub-component `ProjectTag` rendered within the card.
- Card supports responsive layout (full width on mobile, grid on desktop).

---

## R6: Search Input Sanitization

**Decision**: Server-side input sanitization with a 500-character query length limit.

**Rationale**: FR-016 requires input sanitization and length limits. 500 characters is generous for natural-language queries while preventing abuse. Sanitization strips HTML tags and normalizes whitespace.

**Alternatives Considered**:
- **Client-side only**: Insufficient — server must validate regardless of client behavior.
- **Shorter limit (100 chars)**: Too restrictive for complex multi-criteria natural-language queries.

---

## R7: Pagination Strategy

**Decision**: Cursor-based pagination for search results.

**Rationale**: FR-013 requires pagination. Cursor-based pagination is more performant for large result sets and avoids the "shifting results" problem with offset-based pagination when data changes. Page size: 12 projects per page (divisible by 2, 3, and 4 for grid layouts).

**Alternatives Considered**:
- **Offset-based (page number)**: Simpler to implement but less performant for deep pagination and can show duplicate/missing results when data changes between pages.
