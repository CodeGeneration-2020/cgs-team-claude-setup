# Research: Project Map

**Feature**: 008-project-map | **Date**: 2026-03-17

## R1: Map Library

**Decision**: Use Leaflet (open-source) with OpenStreetMap tiles as the default. Mapbox GL JS as an alternative if richer styling is needed.

**Rationale**: Leaflet is lightweight (~40KB), free, requires no API key for basic OSM tiles, and handles the simple read-only map use case well. It supports markers (pins) and polygon/region overlays. Mapbox offers better styling but requires an API key and has usage costs.

**Alternatives Considered**:
- **Google Maps**: Requires API key, usage-based pricing, heavier SDK. Over-engineered for a static informational map.
- **Mapbox GL JS**: Better styling and vector tiles but adds cost. Can swap in later if needed.

---

## R2: Geocoding Strategy

**Decision**: Server-side geocoding at project creation/edit time. Store latitude/longitude on the project record. Frontend reads cached coordinates — no geocoding on every page view.

**Rationale**: Client-side geocoding on every page load wastes API calls and adds latency. Geocoding once at creation/edit and caching coordinates is efficient and reliable. If geocoding fails, lat/lng remain null and the map is hidden.

**Alternatives Considered**:
- **Client-side geocoding on page load**: Wasteful — same project geocoded thousands of times by different visitors.
- **Geocode lazily on first view**: Still wasteful for popular projects. Better to do it once on save.

**Implementation Notes**:
- Add `latitude` (Float, nullable) and `longitude` (Float, nullable) to the project model.
- On project create/edit: geocode the location text → store coordinates. Use a geocoding service (Nominatim for OSM, or Mapbox Geocoding API).
- If geocoding fails (e.g., "Global" or ambiguous text): leave lat/lng null, map won't render.
- Geocoding runs asynchronously after save — doesn't block the project creation response.

---

## R3: Region vs Pin Display

**Decision**: Use a pin marker for all geocoded locations. For broad regions (country/region level), zoom out appropriately. Don't attempt polygon region highlighting in v1.

**Rationale**: The spec says "pin or highlighted region" but region highlighting requires GeoJSON boundary data for every possible region — complex and data-heavy. A pin with appropriate zoom level (country-level zoom for broad locations, city-level for specific ones) delivers 90% of the value with 10% of the complexity. Region highlighting can be added later.

**Alternatives Considered**:
- **GeoJSON region polygons**: Accurate but requires boundary data for all regions. Significant data and rendering complexity.
- **Circle overlay**: Approximate but unclear for the user. Pin with zoom is simpler and more intuitive.
