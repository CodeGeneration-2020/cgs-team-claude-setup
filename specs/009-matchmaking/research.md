# Research: Matchmaking

**Feature**: 009-matchmaking | **Date**: 2026-03-17

## R1: Scoring Architecture

**Decision**: Three-layer scorer: StructuredScorer (field matching) → AIScorer (vector similarity) → CompositeScorer (weighted combination). Composite score = 0.6 * structured + 0.4 * AI.

**Rationale**: FR-004 requires structured matches weighted higher than AI. A 60/40 split ensures exact taxonomy matches (theme, region) dominate while AI adds value for nuanced preference text. Weights are configurable.

**Alternatives Considered**:
- **AI-only scoring**: Ignores exact field matches, which are the most reliable signal.
- **Structured-only**: Already exists in 005-onboarding. Adding AI is the whole point of this feature.
- **Equal weighting (50/50)**: AI is less reliable than exact matches — structured should dominate.

---

## R2: Structured Scoring Components

**Decision**: Score across 4 dimensions, each contributing to the structured score:
1. **Theme overlap** (0.4 weight): Count of matching thematic areas between preferences keywords and project.thematic_areas.
2. **Region match** (0.3 weight): Preference geography keywords match project.region or project.location.
3. **Sector match** (0.15 weight): Preference focus keywords match project.stakeholder_sector.
4. **Funding gap** (0.15 weight): Projects with larger unfunded gaps score higher (funding_requested is high = more opportunity).

**Rationale**: Theme and region are the primary signals per PRD (US-31). Sector and funding gap add nuance.

---

## R3: AI Scoring Strategy

**Decision**: Embed the Funder's preference text (concatenated areas + geography + focus) using the same embedding model as homepage search (001-homepage). Compute cosine similarity against each project's embedding. Normalize to 0-1.

**Rationale**: Reuses the existing vector infrastructure (pgvector + embedding model) from 001-homepage. No new AI service needed. The Funder's preference embedding can be cached and only regenerated when preferences change.

**Implementation Notes**:
- Generate preference embedding on preference save/update (async, stored in user_preferences table).
- On match request: compute cosine similarity between preference embedding and all active project embeddings.
- Cache the preference embedding column: `preference_embedding` (vector) on UserPreferences entity.

---

## R4: Determinism

**Decision**: Ensure deterministic ordering by using a stable sort: composite_score DESC, project.created_at DESC, project.id ASC. Same inputs always produce same output.

**Rationale**: FR-007 requires reproducible rankings. Floating-point scores can have ties. Adding tiebreakers on created_at and id ensures total ordering.

---

## R5: Caching vs On-the-Fly

**Decision**: Compute scores on-the-fly. No cache table. With pgvector cosine similarity queries and structured field matching, scoring hundreds of projects is fast enough (< 3 seconds).

**Rationale**: Caching adds complexity (invalidation on project change, preference change). On-the-fly is simpler and meets the 3-second SLA for hundreds of projects. Can add caching later if scale demands.

**Alternatives Considered**:
- **Precomputed match_scores table**: Accurate but requires invalidation logic on every project or preference change. Over-engineered for current scale.
