# API Contract: Bookmarks

**Feature**: 004-bookmarks | **Date**: 2026-03-17

## POST /v1/bookmarks/:projectId

Bookmarks a project. Idempotent — bookmarking an already-bookmarked project returns success.

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Response (201 Created / 200 OK if already bookmarked)

```json
{
  "success": true,
  "data": {
    "bookmarkId": "clx_bm_001",
    "projectId": "clx1abc123",
    "createdAt": "2026-03-17T10:00:00Z"
  }
}
```

### Error (404 Not Found)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "Project not found."
  }
}
```

---

## DELETE /v1/bookmarks/:projectId

Removes a bookmark. Idempotent — removing a non-existent bookmark returns success.

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Response (200 OK)

```json
{
  "success": true,
  "data": null
}
```

---

## GET /v1/bookmarks

Returns the current user's bookmarked projects with project details.

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "bookmarkId": "clx_bm_001",
      "project": {
        "id": "clx1abc123",
        "title": "Solar Microgrid for Rural Kenya",
        "organizationName": "GreenPower Africa",
        "thematicAreas": ["Renewable Energy"],
        "location": "Kenya, East Africa",
        "status": "APPROVED",
        "fundingRequested": 250000,
        "shortDescription": "Deploying solar microgrids...",
        "tags": [
          { "label": "Renewable Energy", "type": "thematic" },
          { "label": "Active", "type": "status" }
        ]
      },
      "isAvailable": true,
      "hasNewUpdates": true,
      "bookmarkedAt": "2026-03-17T10:00:00Z"
    },
    {
      "bookmarkId": "clx_bm_002",
      "project": null,
      "isAvailable": false,
      "hasNewUpdates": false,
      "bookmarkedAt": "2026-03-10T08:00:00Z"
    }
  ],
  "meta": {
    "total": 15,
    "page": 1,
    "pageSize": 20,
    "totalPages": 1
  }
}
```

**Notes:**
- `isAvailable: false` when the project has been deleted (project is null).
- `hasNewUpdates: true` when audit log entries exist after `last_seen_at`.
- Projects are returned with the same card data shape as search results.

### Response (200 OK — empty)

```json
{
  "success": true,
  "data": [],
  "meta": {
    "total": 0,
    "page": 1,
    "pageSize": 20,
    "totalPages": 0
  }
}
```

---

## GET /v1/bookmarks/updates

Returns recent updates for the user's bookmarked projects.

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |
| onlyNew | boolean | false | If true, only return updates newer than last_seen_at |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "projectId": "clx1abc123",
      "projectTitle": "Solar Microgrid for Rural Kenya",
      "changeType": "funding_changed",
      "changeSummary": "Funding requested changed from $250,000 to $300,000",
      "occurredAt": "2026-03-17T14:00:00Z",
      "isNew": true
    },
    {
      "projectId": "clx2def456",
      "projectTitle": "Mangrove Restoration",
      "changeType": "status_changed",
      "changeSummary": "Project status changed from Pending Approval to Approved",
      "occurredAt": "2026-03-17T12:00:00Z",
      "isNew": false
    }
  ],
  "meta": {
    "total": 5,
    "page": 1,
    "pageSize": 20,
    "totalPages": 1
  }
}
```

---

## POST /v1/bookmarks/:projectId/mark-seen

Marks updates for a specific bookmarked project as seen (updates last_seen_at).

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "bookmarkId": "clx_bm_001",
    "lastSeenAt": "2026-03-17T15:00:00Z"
  }
}
```

---

## GET /v1/bookmarks/check?projectIds=id1,id2,id3

Checks which projects in a list are bookmarked by the current user. Used by ProjectCard to show the bookmark icon state without N+1 queries.

**Authentication**: Required (FUNDER or SUPER_ADMIN role)

### Query Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| projectIds | string | Yes | Comma-separated list of project IDs (max 50) |

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "clx1abc123": true,
    "clx2def456": false,
    "clx3ghi789": true
  }
}
```

**Notes:**
- Returns a map of projectId → isBookmarked.
- Used by search results and trending section to show bookmark state on all visible cards in a single request.
