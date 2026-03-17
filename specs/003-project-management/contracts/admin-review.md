# API Contract: Admin Review

**Feature**: 003-project-management | **Date**: 2026-03-17

## GET /v1/admin/projects/pending

Returns all projects pending Super Admin review.

**Authentication**: Required (SUPER_ADMIN role)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |
| sortBy | string | createdAt | Sort field: createdAt, title, fundingRequested |
| sortOrder | string | asc | Sort direction: asc, desc |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "organizationName": "GreenPower Africa",
      "thematicAreas": ["Renewable Energy"],
      "location": "Kenya, East Africa",
      "fundingRequested": 250000,
      "status": "PENDING_APPROVAL",
      "ownerId": "user123",
      "ownerName": "Jane Smith",
      "ownerEmail": "jane@greenpower.org",
      "publishedAt": "2026-03-17T14:30:00Z",
      "createdAt": "2026-03-17T10:00:00Z"
    }
  ],
  "meta": {
    "total": 3,
    "page": 1,
    "pageSize": 20,
    "totalPages": 1
  }
}
```

---

## POST /v1/admin/projects/:id/approve

Approves a pending project.

**Authentication**: Required (SUPER_ADMIN role)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "status": "APPROVED",
    "reviewedBy": "admin_user_id",
    "reviewedAt": "2026-03-17T15:00:00Z"
  }
}
```

### Error (409 Conflict — already reviewed)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ALREADY_REVIEWED",
    "message": "This project has already been reviewed."
  }
}
```

### Error (404 Not Found — project deleted)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "This project has been removed by the owner."
  }
}
```

---

## POST /v1/admin/projects/:id/reject

Rejects a pending project with a mandatory reason.

**Authentication**: Required (SUPER_ADMIN role)

### Request

```json
{
  "reason": "The project description lacks sufficient detail about the implementation plan and expected outcomes. Please provide more information about milestones and measurable impact."
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| reason | string | Yes | Min 10 chars, max 2000 chars |

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "status": "REJECTED",
    "rejectionReason": "The project description lacks sufficient detail...",
    "reviewedBy": "admin_user_id",
    "reviewedAt": "2026-03-17T15:00:00Z"
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
    "message": "Rejection reason is required.",
    "details": {
      "reason": ["Must be at least 10 characters"]
    }
  }
}
```

### Error (409 Conflict — already reviewed)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ALREADY_REVIEWED",
    "message": "This project has already been reviewed."
  }
}
```
