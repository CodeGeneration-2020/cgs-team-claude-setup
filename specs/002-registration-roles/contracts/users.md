# API Contract: Users

**Feature**: 002-registration-roles | **Date**: 2026-03-17

## POST /v1/users/select-role

Sets the user's role after registration.

**Authentication**: Required (user must be logged in, role must be null)

### Request

```json
{
  "role": "ORG_ADMIN"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| role | string | Yes | One of: "ORG_ADMIN", "FUNDER". "SUPER_ADMIN" is rejected. |

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "userId": "clx1abc123",
    "role": "ORG_ADMIN",
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "bmV3IHJlZnJlc2ggdG9rZW4..."
  }
}
```

**Notes:**
- New tokens are issued with the role included in the payload.
- Old tokens (without role) are invalidated.

### Error (400 Bad Request)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid role. Must be ORG_ADMIN or FUNDER."
  }
}
```

### Error (409 Conflict)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ROLE_ALREADY_SET",
    "message": "Role has already been selected."
  }
}
```

---

## POST /v1/users/convert-role

Converts an Org Admin account to a Funder account.

**Authentication**: Required (ORG_ADMIN role only)

### Request

```json
{
  "targetRole": "FUNDER"
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "userId": "clx1abc123",
    "previousRole": "ORG_ADMIN",
    "newRole": "FUNDER",
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "bmV3IHJlZnJlc2ggdG9rZW4..."
  }
}
```

**Notes:**
- All existing refresh tokens are revoked.
- New tokens issued with FUNDER role.
- Auth audit log entry created with event type ROLE_CONVERTED.

### Error (400 Bad Request — has active projects)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ACTIVE_PROJECTS_EXIST",
    "message": "Cannot convert account while you have active, approved, or pending projects. Please archive or delete them first."
  }
}
```

### Error (400 Bad Request — invalid conversion)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_CONVERSION",
    "message": "Only Organization Admin accounts can be converted to Funder."
  }
}
```

---

## GET /v1/users/me

Returns the current user's profile.

**Authentication**: Required (any role)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "clx1abc123",
    "name": "Jane Smith",
    "email": "jane@example.com",
    "role": "ORG_ADMIN",
    "companyName": "GreenPower Africa",
    "companyType": "NGO",
    "isEmailVerified": true,
    "createdAt": "2026-03-15T10:00:00Z",
    "canConvertToFunder": true
  }
}
```

**Notes:**
- `canConvertToFunder` is true only if role is ORG_ADMIN and user has no projects with status Active, Approved, or Pending Approval.
