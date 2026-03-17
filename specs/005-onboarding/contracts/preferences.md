# API Contract: Preferences & Recommendations

**Feature**: 005-onboarding | **Date**: 2026-03-17

## POST /v1/preferences

Saves onboarding preferences. Creates or updates the user's preferences record.

**Authentication**: Required (any authenticated role)

### Request

```json
{
  "areasOfInterest": "Renewable energy, clean water access, sustainable agriculture",
  "geographicPreferences": "East Africa, Southeast Asia, Latin America",
  "fundingFocus": "Early-stage projects with strong community impact, local NGO implementers"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| areasOfInterest | string | No | Max 2000 chars |
| geographicPreferences | string | No | Max 2000 chars |
| fundingFocus | string | No | Max 2000 chars |

At least one field must be non-empty.

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "userId": "clx1abc123",
    "isOnboardingCompleted": true,
    "completedAt": "2026-03-17T10:00:00Z"
  }
}
```

---

## GET /v1/preferences

Returns the current user's preferences.

**Authentication**: Required (any authenticated role)

### Response (200 OK — has preferences)

```json
{
  "success": true,
  "data": {
    "areasOfInterest": "Renewable energy, clean water access",
    "geographicPreferences": "East Africa, Southeast Asia",
    "fundingFocus": "Early-stage projects with community impact",
    "isOnboardingCompleted": true,
    "completedAt": "2026-03-17T10:00:00Z",
    "updatedAt": "2026-03-17T14:00:00Z"
  }
}
```

### Response (200 OK — no preferences / skipped)

```json
{
  "success": true,
  "data": null
}
```

---

## GET /v1/recommendations

Returns personalized project recommendations based on user preferences.

**Authentication**: Required (any authenticated role)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| limit | number | 6 | Max recommendations to return (max 20) |

### Response (200 OK — has preferences)

```json
{
  "success": true,
  "data": [
    {
      "projectId": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "organizationName": "GreenPower Africa",
      "thematicAreas": ["Renewable Energy"],
      "location": "Kenya, East Africa",
      "shortDescription": "Deploying solar microgrids...",
      "status": "APPROVED",
      "fundingRequested": 250000,
      "tags": [
        { "label": "Renewable Energy", "type": "thematic" }
      ],
      "matchScore": 0.85,
      "matchReasons": [
        "Matches your interest in Renewable Energy",
        "Located in East Africa (your preferred region)"
      ]
    }
  ],
  "meta": {
    "hasPreferences": true,
    "totalMatches": 12
  }
}
```

### Response (200 OK — no preferences)

```json
{
  "success": true,
  "data": [],
  "meta": {
    "hasPreferences": false,
    "message": "Complete your preferences to get personalized recommendations."
  }
}
```

### Response (200 OK — no matches)

```json
{
  "success": true,
  "data": [],
  "meta": {
    "hasPreferences": true,
    "message": "No projects currently match your preferences. Try broadening your interests."
  }
}
```
