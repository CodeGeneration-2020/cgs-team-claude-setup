# Data Model: Project Map

**Feature**: 008-project-map | **Date**: 2026-03-17

## Entities

### Project (existing — extended)

Add geocoded coordinates as cached fields.

**New fields:**

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| latitude | Float | Geocoded latitude | Nullable, set on create/edit |
| longitude | Float | Geocoded longitude | Nullable, set on create/edit |
| geocode_status | String | Result of geocoding: "success", "failed", "pending" | Default: "pending" |

**Notes:**
- Geocoding runs asynchronously after project creation or location field edit.
- If geocoding fails (ambiguous/unmappable location), geocode_status = "failed" and lat/lng remain null.
- Map component reads lat/lng — if null, map section is hidden.
- No new indexes needed — lat/lng are only read with the project record, not queried independently.

## No New Entities

No new tables. Just 3 nullable columns on the existing project.
