# API Contract: Projects

**Feature**: 003-project-management | **Date**: 2026-03-17

## POST /v1/projects

Creates a new project.

**Authentication**: Required (ORG_ADMIN role)

### Request

```json
{
  "title": "Solar Microgrid for Rural Kenya",
  "description": "Deploying solar microgrids to provide clean electricity...",
  "shortDescription": "Clean energy access for 5,000 households",
  "thematicAreas": ["Renewable Energy", "Energy Access"],
  "location": "Kenya, East Africa",
  "region": "East Africa",
  "lifecycleStatus": "Implementation",
  "fundingRequested": 250000,
  "implementerType": "NGO",
  "stakeholderSector": "Energy",
  "isLocalNgo": true,
  "timeline": "2026-2028",
  "partners": "GreenPower Foundation, Local Energy Co-op",
  "sustainabilityInfo": "Revenue model based on micro-payments..."
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| title | string | Yes | Max 200 chars |
| description | string | Yes | Max 5000 chars |
| shortDescription | string | Yes | Max 300 chars |
| thematicAreas | string[] | Yes | At least 1, each must be valid THEMATIC_AREA taxonomy value |
| location | string | Yes | Non-empty |
| region | string | No | Must be valid REGION taxonomy value if provided |
| lifecycleStatus | string | Yes | Must be valid LIFECYCLE_STATUS taxonomy value |
| fundingRequested | number | Yes | Positive number |
| implementerType | string | Yes | Must be valid IMPLEMENTER_TYPE taxonomy value |
| stakeholderSector | string | No | Must be valid STAKEHOLDER_SECTOR taxonomy value if provided |
| isLocalNgo | boolean | No | Default: false |
| timeline | string | No | Max 1000 chars |
| partners | string | No | Max 2000 chars |
| sustainabilityInfo | string | No | Max 2000 chars |

### Response (201 Created)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "title": "Solar Microgrid for Rural Kenya",
    "status": "PENDING_APPROVAL",
    "createdAt": "2026-03-17T10:00:00Z"
  }
}
```

### Error (400 Validation)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": {
      "thematicAreas": ["'Nuclear Fusion' is not a valid thematic area"],
      "fundingRequested": ["Must be a positive number"]
    }
  }
}
```

---

## GET /v1/projects/my

Returns all projects owned by the authenticated user.

**Authentication**: Required (ORG_ADMIN role)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| status | string | all | Filter by status (DRAFT, PENDING_APPROVAL, APPROVED, REJECTED, ARCHIVED) |
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "thematicAreas": ["Renewable Energy"],
      "location": "Kenya, East Africa",
      "fundingRequested": 250000,
      "status": "APPROVED",
      "rejectionReason": null,
      "createdAt": "2026-03-17T10:00:00Z",
      "updatedAt": "2026-03-17T12:00:00Z"
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

## GET /v1/projects/:id

Returns a single project's full details.

**Authentication**: Public (for APPROVED projects) or Owner/Super Admin (for other statuses)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "title": "Solar Microgrid for Rural Kenya",
    "description": "Deploying solar microgrids...",
    "shortDescription": "Clean energy access for 5,000 households",
    "organizationName": "GreenPower Africa",
    "thematicAreas": ["Renewable Energy", "Energy Access"],
    "location": "Kenya, East Africa",
    "region": "East Africa",
    "lifecycleStatus": "Implementation",
    "fundingRequested": 250000,
    "implementerType": "NGO",
    "stakeholderSector": "Energy",
    "isLocalNgo": true,
    "timeline": "2026-2028",
    "partners": "GreenPower Foundation, Local Energy Co-op",
    "sustainabilityInfo": "Revenue model based on micro-payments...",
    "status": "APPROVED",
    "rejectionReason": null,
    "tags": [
      { "label": "Renewable Energy", "type": "thematic" },
      { "label": "Active", "type": "status" }
    ],
    "ownerId": "user123",
    "createdAt": "2026-03-17T10:00:00Z",
    "updatedAt": "2026-03-17T12:00:00Z"
  }
}
```

---

## PATCH /v1/projects/:id

Updates an existing project.

**Authentication**: Required (ORG_ADMIN, must be project owner)

### Request

```json
{
  "description": "Updated description with more detail...",
  "fundingRequested": 300000,
  "partners": "New partner added"
}
```

All fields are optional. Only provided fields are updated. Same validation rules as creation apply.

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "title": "Solar Microgrid for Rural Kenya",
    "status": "APPROVED",
    "updatedAt": "2026-03-17T14:00:00Z"
  }
}
```

### Error (403 Forbidden)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "FORBIDDEN",
    "message": "You can only edit projects you own."
  }
}
```

---

## POST /v1/projects/:id/publish

Publishes a project (changes status to PENDING_APPROVAL).

**Authentication**: Required (ORG_ADMIN, must be project owner)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "status": "PENDING_APPROVAL",
    "publishedAt": "2026-03-17T14:30:00Z"
  }
}
```

---

## DELETE /v1/projects/:id

Deletes a project permanently.

**Authentication**: Required (ORG_ADMIN, must be project owner)

### Response (200 OK)

```json
{
  "success": true,
  "data": null
}
```

---

## GET /v1/taxonomy

Returns all predefined taxonomy values grouped by category.

**Authentication**: None (public endpoint)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "THEMATIC_AREA": [
      { "value": "Renewable Energy", "displayOrder": 1 },
      { "value": "Clean Water", "displayOrder": 2 },
      { "value": "Biodiversity", "displayOrder": 3 }
    ],
    "STAKEHOLDER_SECTOR": [
      { "value": "Energy", "displayOrder": 1 },
      { "value": "Agriculture", "displayOrder": 2 }
    ],
    "IMPLEMENTER_TYPE": [
      { "value": "NGO", "displayOrder": 1 },
      { "value": "Government", "displayOrder": 2 }
    ],
    "REGION": [
      { "value": "East Africa", "displayOrder": 1 },
      { "value": "West Africa", "displayOrder": 2 }
    ],
    "LIFECYCLE_STATUS": [
      { "value": "Concept", "displayOrder": 1 },
      { "value": "Planning", "displayOrder": 2 },
      { "value": "Implementation", "displayOrder": 3 }
    ]
  }
}
```
