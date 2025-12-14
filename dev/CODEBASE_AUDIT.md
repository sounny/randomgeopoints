# Codebase Audit: Random GeoPoints (v2.0 Pre-Refactor)

**Date:** December 14, 2025
**Scope:** `index.html` (Single Page Application)

## 1. Executive Summary
The current application consists of a single HTML file (`index.html`) containing CSS, HTML structure, and approximately 300 lines of Vanilla JavaScript. It relies on CDN-hosted libraries for mapping and spatial analysis. The architecture is monolithic, with all logic (UI, State, Math, Map) tightly coupled within the global scope.

## 2. Technical Dependencies
The application loads the following external libraries via CDN (unpkg.com):
*   **Leaflet.js (v1.9.4):** Interactive mapping, layer management, and event handling.
*   **Leaflet Draw (v1.0.4):** UI controls for drawing bounding boxes.
*   **Turf.js (v6.5.0):** Client-side geospatial analysis (`bbox`, `point`, `booleanPointInPolygon`, `area`, `centerOfMass`).

**Missing Critical Dependency:**
*   **Proj4js:** Not currently present. This confirms the need for Phase 1 of the development plan, as accurate area calculations and density sampling require equal-area projections, which Turf.js alone does not handle robustly for global coordinates.

## 3. Architecture & Logic Flow

### 3.1 Implicit Modules
Although physically located in one `<script>` block, the code is logically divided into:

1.  **Utilities & State:**
    *   `state` object: Stores `aoi`, `strata`, `sample`, and `analysisData`.
    *   `seedRNG`: Mulberry32 implementation for reproducible randomness.
    *   `halton`: Generator for low-discrepancy sequences (bases 2 and 3).
    *   `toFeatureCollection`: Normalizes diverse GeoJSON inputs.

2.  **UI Logic:**
    *   Tab switching logic (`.tab` click listeners).
    *   Dynamic form visibility based on `#method` selection.
    *   Drag-and-drop event handlers (`setupDrop`).

3.  **Map Logic:**
    *   Leaflet initialization (`L.map`).
    *   Layer groups (`layers` object) for clean state management.
    *   `L.Control.Draw` integration for creating AOIs.

4.  **Generation Logic (The "Sampling Engine"):**
    *   Triggered by `#genBtn`.
    *   **Current Limitations:**
        *   Uses a `while` loop with rejection sampling (`pointInPolygons`).
        *   **Stratified Sampling is missing logic:** The code detects "stratified" methods but falls back to simple random sampling logic in the `else` block. It does *not* yet use the `state.strata` geometry for allocation.
        *   **SRS:** Uses bounding box rejection sampling.
        *   **Plots:** Generates square polygons based on meters-to-degrees conversion (simplified latitude adjustment).

5.  **Analysis Logic (The "Olofsson Engine"):**
    *   Triggered by `#calcBtn`.
    *   Parses CSV/GeoJSON inputs.
    *   Builds a basic Confusion Matrix.
    *   **Current Limitation:** Calculates accuracy ($OA, UA, PA$) based on raw counts (Simple Random Sampling assumption). **It does not implement Olofsson equations 1-6** (Area-Weighted estimates) as required for the final product.

## 4. Key Functions & Signatures

| Function | Signature | Purpose | Notes |
| :--- | :--- | :--- | :--- |
| `seedRNG` | `(str) => () => number` | Returns a seeded random number generator function. | Uses Mulberry32 algorithm. |
| `halton` | `(index, base) => number` | Returns the nth number in a Halton sequence. | Used for Spatially Balanced sampling. |
| `extractPolygons` | `(geojson) => Feature[]` | Filters input for Polygon/MultiPolygon features. | Robustness helper. |
| `polygonsAreaSqKm` | `(features) => number` | Calculates total area using Turf.js. | **Risk:** Geodesic calculation may vary; Proj4js needed for precision. |
| `pointInPolygons` | `(pt, polygons) => boolean` | Checks inclusion. | Uses `turf.booleanPointInPolygon`. |

## 5. Risk Assessment for Refactoring
*   **Global State:** The `state` object is mutable and accessed directly by event listeners. Refactoring to modules requires creating a distinct `Store` or `StateManager`.
*   **Async/UI Blocking:** The generation loop runs on the main thread inside a `setTimeout`. For large N or complex polygons, this will freeze the UI. **Recommendation:** Move sampling logic to a Web Worker during refactoring.
*   **Coupled DOM:** Logic is heavily tied to specific ID selectors (`getElementById`). Refactoring should separate View logic from Business Logic.

## 6. Conclusion
The codebase is a functional prototype but requires significant structural changes to meet v2.0 goals. The primary technical gaps are the absence of `Proj4js` for projection handling and the missing implementation of Stratified Sampling allocation logic and Area-Weighted Analysis.
