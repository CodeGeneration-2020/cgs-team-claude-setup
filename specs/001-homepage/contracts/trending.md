# API Contract: Trending Projects

**Feature**: 001-homepage | **Date**: 2026-03-17

## GET /v1/trending

Returns the current list of trending projects for homepage display.

**Authentication**: None (public endpoint)

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
      "tags": [
        { "label": "Renewable Energy", "type": "thematic" },
        { "label": "Active", "type": "status" }
      ],
      "displayOrder": 1
    },
    {
      "id": "clx2def456",
      "title": "Mangrove Restoration in Bangladesh",
      "organizationName": "Coastal Resilience Fund",
      "thematicArea": ["Climate Adaptation", "Biodiversity"],
      "location": "Bangladesh, South Asia",
      "status": "ACTIVE",
      "fundingRequested": 180000,
      "shortDescription": "Restoring 500 hectares of mangrove forest to protect coastal communities from storm surges.",
      "tags": [
        { "label": "Climate Adaptation", "type": "thematic" },
        { "label": "Active", "type": "status" }
      ],
      "displayOrder": 2
    }
  ]
}
```

**Notes:**
- Returns only projects with status Active or Approved.
- If a trending project has been archived/deactivated, it is automatically excluded from the response.
- Results are ordered by `displayOrder` ascending.

### Response (200 OK — Empty)

```json
{
  "success": true,
  "data": []
}
```

---

## GET /v1/admin/trending

Returns the full trending list for Super Admin management (includes inactive projects).

**Authentication**: Required (Super Admin role)

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "trending_1",
      "projectId": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "organizationName": "GreenPower Africa",
      "thematicArea": ["Renewable Energy"],
      "fundingRequested": 250000,
      "location": "Kenya, East Africa",
      "shortDescription": "Deploying solar microgrids...",
      "projectStatus": "ACTIVE",
      "displayOrder": 1,
      "addedBy": "admin_user_id",
      "createdAt": "2026-03-10T14:00:00Z"
    },
    {
      "id": "trending_2",
      "projectId": "clx3ghi789",
      "title": "Archived Project Example",
      "organizationName": "Old Org",
      "thematicArea": ["Biodiversity"],
      "fundingRequested": 50000,
      "location": "Peru",
      "shortDescription": "This project was archived...",
      "projectStatus": "ARCHIVED",
      "displayOrder": 2,
      "addedBy": "admin_user_id",
      "createdAt": "2026-02-20T09:00:00Z"
    }
  ]
}
```

**Notes:**
- Includes projects regardless of status (so admin can see archived items in their list).
- `projectStatus` field shows the current status of the project.

---

## PUT /v1/admin/trending

Replaces the entire trending list with a new ordered set.

**Authentication**: Required (Super Admin role)

### Request

```json
{
  "projects": [
    { "projectId": "clx1abc123", "displayOrder": 1 },
    { "projectId": "clx2def456", "displayOrder": 2 },
    { "projectId": "clx4jkl012", "displayOrder": 3 }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| projects | array | Yes | Ordered list of projects to set as trending |
| projects[].projectId | string | Yes | ID of the project to add |
| projects[].displayOrder | number | Yes | Display position (1-based) |

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "count": 3,
    "updatedAt": "2026-03-17T15:30:00Z"
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
    "message": "One or more project IDs are invalid.",
    "details": {
      "projects": ["Project 'clx_invalid' does not exist"]
    }
  }
}
```

### Error Response (403 Forbidden)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "FORBIDDEN",
    "message": "Only Super Admin users can manage trending projects."
  }
}
```
