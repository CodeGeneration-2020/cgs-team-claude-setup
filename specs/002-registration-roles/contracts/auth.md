# API Contract: Authentication

**Feature**: 002-registration-roles | **Date**: 2026-03-17

## POST /v1/auth/register

Creates a new user account.

**Authentication**: None (public endpoint, marked with @Public)

### Request

```json
{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "password": "SecurePass1!",
  "companyName": "GreenPower Africa",
  "companyType": "NGO"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| name | string | Yes | Min 2 chars, max 100 |
| email | string | Yes | Valid email format, unique |
| password | string | Yes | Min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char |
| companyName | string | Yes | Min 2 chars, max 200 |
| companyType | string | Yes | Non-empty |

### Response (201 Created)

```json
{
  "success": true,
  "data": {
    "userId": "clx1abc123",
    "email": "jane@example.com",
    "name": "Jane Smith",
    "needsRoleSelection": true
  }
}
```

### Error (409 Conflict)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "DUPLICATE_EMAIL",
    "message": "An account with this email already exists."
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
      "password": ["Must contain at least 1 uppercase letter, 1 number, and 1 special character"]
    }
  }
}
```

---

## POST /v1/auth/login

Authenticates a user and returns JWT tokens.

**Authentication**: None (public endpoint)

### Request

```json
{
  "email": "jane@example.com",
  "password": "SecurePass1!"
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g...",
    "expiresIn": 900,
    "user": {
      "id": "clx1abc123",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "role": "ORG_ADMIN",
      "companyName": "GreenPower Africa"
    }
  }
}
```

### Response (200 OK — Role not yet selected)

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g...",
    "expiresIn": 900,
    "user": {
      "id": "clx1abc123",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "role": null,
      "companyName": "GreenPower Africa"
    },
    "needsRoleSelection": true
  }
}
```

### Error (401 Unauthorized)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password."
  }
}
```

### Error (429 Too Many Requests)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many login attempts. Please try again later."
  }
}
```

---

## POST /v1/auth/refresh

Refreshes the access token using a valid refresh token.

**Authentication**: None (uses refresh token in body)

### Request

```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g..."
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "bmV3IHJlZnJlc2ggdG9rZW4...",
    "expiresIn": 900
  }
}
```

### Error (401 Unauthorized)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Refresh token is invalid or expired."
  }
}
```

---

## POST /v1/auth/logout

Revokes the current refresh token and ends the session.

**Authentication**: Required (any role)

### Request

```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g..."
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": null
}
```

---

## POST /v1/auth/forgot-password

Sends a password reset email. Always returns success to prevent email enumeration.

**Authentication**: None (public endpoint)

### Request

```json
{
  "email": "jane@example.com"
}
```

### Response (200 OK — always, regardless of email existence)

```json
{
  "success": true,
  "data": {
    "message": "If an account exists with this email, a reset link has been sent."
  }
}
```

---

## POST /v1/auth/reset-password

Resets the user's password using a valid reset token.

**Authentication**: None (public endpoint, uses reset token)

### Request

```json
{
  "token": "a1b2c3d4e5f6...",
  "newPassword": "NewSecurePass2!"
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "message": "Password has been reset successfully. Please log in with your new password."
  }
}
```

### Error (400 Bad Request)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_RESET_TOKEN",
    "message": "This reset link has expired or has already been used. Please request a new one."
  }
}
```
