# Development Plan – Next Iteration (Dec 2025)

Source: `references/random_point_area_sampling_dissertation.md` (random point and area sampling dissertation).

## Guiding principles pulled from the dissertation
- Match the sampling unit to the map meaning: support both point-based and area/plot-based validation units; avoid pixel-only validation when MMU is larger.
- Keep probability-based selection visible: expose inclusion probabilities/weights and the sampling frame used for each method.
- Use estimators/designs that align: avoid treating stratified or cluster designs as simple random; keep stratum metadata with samples.
- Promote spatial spread: provide a spatially balanced design option for lower variance under spatial autocorrelation.
- Document response design hooks: include metadata fields to record labeling rules and reference source notes for reproducibility.

## Near-term feature goals
1) Area/plot sampling
- Add a “random plots” mode that samples square plots (user-defined side length in meters) fully contained in the AOI.
- Export plots as polygons with unit metadata (plot area, side length, unit type) and design weights.

2) Spatially balanced option
- Implement a Halton/BAS-lite spatially balanced point sampler inside the AOI to reduce clumping versus pure SRS.
- Surface when this method is chosen and how many attempts were required to fill the sample.

3) Design weights and metadata
- Attach `inclusionProb` and `weightAreaSqM` to all outputs; for stratified designs compute weights using stratum area and sample counts.
- Add per-feature fields for stratum ID, cluster ID (where applicable), and unit type (point vs plot).
- Add a UI “Design metadata” panel summarizing method, unit of analysis, weighting rule, and AOI/strata area totals.

4) UI/UX tightening
- Clean unit labels (km^2) and warn when AOI is missing or plot placement fails.
- Keep advanced options context-sensitive (only show plot size for area sampling, etc.).

5) Export/reporting
- Include a `design` metadata block in GeoJSON/CSV exports (method, unit type, AOI area, stratum areas, seed).
- Provide a short inline note reminding users to align analysis/variance estimation with the selected design.

## Non-goals for this iteration
- Full variance estimators for cluster designs, change-area estimators, or reference-label uncertainty modeling.
- GUI for multi-date change sampling or boundary quality scoring.

## Validation plan
- Run `npx htmlhint index.html` after changes.
- Manual smoke tests: generate samples for each method (including plots and spatially balanced) and confirm features render, stay inside AOI, and export weights/metadata.
