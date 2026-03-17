# API Contract: Search

**Feature**: 001-homepage | **Date**: 2026-03-17

## POST /v1/search

Performs AI-powered semantic search across projects.

**Authentication**: None (public endpoint)

### Request

```json
{
  "query": "renewable energy projects in East Africa",
  "filters": {
    "thematicArea": ["Renewable Energy"],
    "region": ["East Africa"],
    "lifecycleStatus": null,
    "organization": null,
    "fundingMin": null,
    "fundingMax": null,
    "stakeholderSector": null,
    "isLocalNgo": null
  },
  "sort": "RELEVANCE",
  "cursor": null,
  "pageSize": 12
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | Yes | Natural language search query. Max 500 characters. |
| filters | object | No | Filter criteria. All fields optional. |
| filters.thematicArea | string[] | No | Filter by one or more thematic areas |
| filters.region | string[] | No | Filter by one or more regions |
| filters.lifecycleStatus | string[] | No | Filter by lifecycle status values |
| filters.organization | string[] | No | Filter by organization names |
| filters.fundingMin | number | No | Minimum funding requested |
| filters.fundingMax | number | No | Maximum funding requested |
| filters.stakeholderSector | string[] | No | Filter by stakeholder sector |
| filters.isLocalNgo | boolean | No | Filter for Local NGO projects only |
| sort | string | No | Sort order. One of: RELEVANCE (default), DATE_NEWEST, DATE_OLDEST, NAME_ASC |
| cursor | string | No | Pagination cursor from previous response. Null for first page. |
| pageSize | number | No | Results per page. Default: 12. Max: 50. |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "organizationName": "GreenPower Africa",
      "thematicArea": ["Renewable Energy", "Energy Access"],
      "location": "Kenya, East Africa",
      "status": "ACTIVE",
      "fundingRequested": 250000,
      "shortDescription": "Deploying solar microgrids to provide clean electricity to 5,000 households in rural Kenya.",
      "stakeholderSector": "Energy",
      "isLocalNgo": true,
      "lifecycleStatus": "Implementation",
      "tags": [
        { "label": "Renewable Energy", "type": "thematic" },
        { "label": "Active", "type": "status" }
      ],
      "relevanceScore": 0.92,
      "createdAt": "2026-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "totalResults": 47,
    "pageSize": 12,
    "nextCursor": "eyJpZCI6ImNseDFhYmMxMjQifQ==",
    "hasMore": true,
    "query": "renewable energy projects in East Africa",
    "filtersApplied": {
      "thematicArea": ["Renewable Energy"],
      "region": ["East Africa"]
    },
    "sort": "RELEVANCE",
    "isRefinementSuggested": false
  }
}
```

### Response (200 OK — Refinement Suggested)

When the query is too broad or ambiguous:

```json
{
  "success": true,
  "data": [],
  "meta": {
    "totalResults": 0,
    "pageSize": 12,
    "nextCursor": null,
    "hasMore": false,
    "query": "help",
    "filtersApplied": {},
    "sort": "RELEVANCE",
    "isRefinementSuggested": true,
    "refinementMessage": "Your search is too broad. Try specifying a theme, region, or type of project — for example, 'clean water projects in West Africa'."
  }
}
```

### Response (200 OK — No Results)

```json
{
  "success": true,
  "data": [],
  "meta": {
    "totalResults": 0,
    "pageSize": 12,
    "nextCursor": null,
    "hasMore": false,
    "query": "quantum computing in Antarctica",
    "filtersApplied": {},
    "sort": "RELEVANCE",
    "isRefinementSuggested": false,
    "refinementMessage": "No relevant active projects found. Try broadening your search or adjusting your filters."
  }
}
```

### Error Response (400 Bad Request)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Search query exceeds maximum length of 500 characters.",
    "details": {
      "query": ["Must be 500 characters or fewer"]
    }
  }
}
```

---

## GET /v1/search/filters

Returns available filter options based on current project data.

**Authentication**: None (public endpoint)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "thematicAreas": ["Renewable Energy", "Clean Water", "Biodiversity", "Climate Adaptation"],
    "regions": ["East Africa", "West Africa", "Southeast Asia", "Latin America"],
    "lifecycleStatuses": ["Concept", "Planning", "Implementation", "Scaling"],
    "organizations": ["GreenPower Africa", "WaterAid", "Conservation International"],
    "stakeholderSectors": ["Energy", "Agriculture", "Health", "Infrastructure"],
    "fundingRange": {
      "min": 5000,
      "max": 5000000
    }
  }
}
```
