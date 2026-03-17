# API Contract: Notifications

**Feature**: 006-notifications | **Date**: 2026-03-17

## GET /v1/notifications

Returns the current user's notifications, paginated, most recent first.

**Authentication**: Required (FUNDER or SUPER_ADMIN)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |
| unreadOnly | boolean | false | If true, return only unread notifications |

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "clx_ntf_001",
      "eventType": "BOOKMARK_PROJECT_UPDATED",
      "title": "Project updated",
      "message": "Solar Microgrid for Rural Kenya — funding requested changed from $250,000 to $300,000",
      "projectId": "clx1abc123",
      "projectTitle": "Solar Microgrid for Rural Kenya",
      "isRead": false,
      "changeCount": 1,
      "createdAt": "2026-03-17T14:00:00Z"
    },
    {
      "id": "clx_ntf_002",
      "eventType": "NEW_MATCHING_PROJECT",
      "title": "New project matching your interests",
      "message": "Mangrove Restoration in Bangladesh matches your interest in Climate Adaptation and South Asia",
      "projectId": "clx2def456",
      "projectTitle": "Mangrove Restoration in Bangladesh",
      "isRead": true,
      "changeCount": 1,
      "createdAt": "2026-03-17T12:00:00Z",
      "readAt": "2026-03-17T13:00:00Z"
    }
  ],
  "meta": {
    "total": 25,
    "unreadCount": 8,
    "page": 1,
    "pageSize": 20,
    "totalPages": 2
  }
}
```

---

## GET /v1/notifications/unread-count

Returns the current user's unread notification count. Lightweight endpoint for badge polling.

**Authentication**: Required (FUNDER or SUPER_ADMIN)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "unreadCount": 8
  }
}
```

---

## POST /v1/notifications/:id/read

Marks a single notification as read.

**Authentication**: Required (must be notification owner)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx_ntf_001",
    "isRead": true,
    "readAt": "2026-03-17T15:00:00Z"
  }
}
```

---

## POST /v1/notifications/read-all

Marks all unread notifications as read.

**Authentication**: Required (FUNDER or SUPER_ADMIN)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "markedCount": 8,
    "readAt": "2026-03-17T15:00:00Z"
  }
}
```

---

## GET /v1/notifications/settings

Returns the user's notification preferences.

**Authentication**: Required

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "isEmailNotificationsEnabled": true
  }
}
```

---

## PATCH /v1/notifications/settings

Updates the user's notification preferences.

**Authentication**: Required

### Request

```json
{
  "isEmailNotificationsEnabled": false
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "isEmailNotificationsEnabled": false,
    "updatedAt": "2026-03-17T15:00:00Z"
  }
}
```
