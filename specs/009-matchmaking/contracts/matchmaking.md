# API Contract: Matchmaking

**Feature**: 009-matchmaking | **Date**: 2026-03-17

Replaces/enhances GET /v1/recommendations from 005-onboarding.

## GET /v1/matches

Returns AI-powered ranked project matches for the authenticated Funder.

**Authentication**: Required (FUNDER or SUPER_ADMIN)

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | number | 1 | Page number |
| pageSize | number | 20 | Results per page (max 50) |

### Response (200 OK — has preferences)

```json
{
  "success": true,
  "data": [
    {
      "projectId": "clx1abc123",
      "title": "Solar Microgrid for Rural Kenya",
      "organizationName": "GreenPower Africa",
      "thematicAreas": ["Renewable Energy", "Energy Access"],
      "location": "Kenya, East Africa",
      "shortDescription": "Deploying solar microgrids...",
      "fundingRequested": 250000,
      "status": "APPROVED",
      "tags": [{ "label": "Renewable Energy", "type": "thematic" }],
      "compositeScore": 0.87,
      "structuredScore": 0.92,
      "aiScore": 0.79,
      "matchReasons": [
        "Matches your interest in Renewable Energy",
        "Located in your preferred region: East Africa",
        "Energy sector aligns with your focus"
      ]
    }
  ],
  "meta": {
    "total": 45,
    "page": 1,
    "pageSize": 20,
    "totalPages": 3,
    "hasPreferences": true,
    "scoringMode": "composite"
  }
}
```

### Response (200 OK — AI unavailable, fallback)

```json
{
  "success": true,
  "data": [ "..." ],
  "meta": {
    "total": 30,
    "page": 1,
    "pageSize": 20,
    "totalPages": 2,
    "hasPreferences": true,
    "scoringMode": "structured_only"
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
    "message": "Complete your preferences to see personalized project matches."
  }
}
```
