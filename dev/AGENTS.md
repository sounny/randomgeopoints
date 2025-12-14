# AGENTS

This file acts as a shared memory and change log for the **Random Geo Points** project. Always read and update it when you work in this repository.

## Instructions for Agents

- Use this file as persistent memory.
- Append notes to the "Change Log" for every commit.
- Keep entries concise and dated.
- Run `npx htmlhint index.html` after modifying code.

## Notes

- Single-page client-side app to generate random geographic points using Leaflet and Turf.js.
- **v2.0 Scope:** Expanded to a "Validation Workbench" including Stratified Sampling, Olofsson Analysis Engine, and Response Design Protocol generation.
- **Architecture:** Moving to a modular structure (InputManager, SamplingEngine, AnalysisEngine, ProtocolBuilder).

## Change Log

- 2024-06-03: Initialized repository memory, added README, and fixed CSV export MIME type.
- 2024-06-08: Added bbox validation, snap support for bbox sampling, and clearer status messaging.
- 2024-06-04: Added bbox validation and clearer AOI status messaging in the UI.
- 2025-12-12: Fixed JavaScript bug - `currentBBox()` was undefined; replaced with inline bbox parsing in the 'Fit Map to BBox' button handler.
- 2025-12-12: Added spatially balanced sampling, random plot (area) sampling, design metadata/weights, and refreshed development plan and README.
- 2025-12-12: Hardened AOI/strata loading (FeatureCollection normalization) and seed handling to prevent undefined length errors; removed non-ASCII UI glyphs.
- 2025-12-14: Created `FULL_DEVELOPMENT_PLAN.md` detailing the v2 roadmap, architecture, and feature prioritization. Defined scope for Stratified Sampling logic, Olofsson Analysis Engine, and Protocol Builder.
- 2025-12-15: Fixed AOI point-in-polygon checks to support FeatureCollections and keep cluster samples inside the AOI.