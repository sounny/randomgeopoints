# Development Plan – Random GeoPoints v2

**Last Updated**: December 2025

## Completed Features ✅

### Phase 1: Core Sampling
- [x] Simple Random Sampling (BBox and Polygon)
- [x] Spatially Balanced (Halton sequence)
- [x] Systematic Grid (rectangular)
- [x] Hexagonal Lattice
- [x] Cluster Sampling (two-stage)
- [x] Random Plots (square area units)

### Phase 2: UI/UX
- [x] Modern tabbed interface (Design → Protocol → Analyze)
- [x] Drag-and-drop file upload
- [x] Draw-on-map AOI definition
- [x] Power analysis visualization
- [x] Loading overlay with spinner
- [x] Dark theme with glassmorphism

### Phase 3: Validation Tools
- [x] Confusion matrix generation
- [x] Overall Accuracy with SE
- [x] User's/Producer's Accuracy per class
- [x] Protocol definition and export

### Phase 4: Data Export
- [x] GeoJSON with metadata
- [x] CSV export
- [x] Protocol text export

---

## In Progress / Planned 🔄

### Near-term
1. **Stratified sampling refinement**
   - Per-stratum allocation UI with custom counts
   - Area-proportional weighting

2. **Design weights**
   - Attach `inclusionProb` to all outputs
   - Support Olofsson-style variance estimation

3. **NNI calculation**
   - Actual nearest-neighbor index computation
   - Spatial pattern feedback

### Medium-term
4. **Area estimation module**
   - Bias-adjusted area estimates per class
   - Confidence intervals for area

5. **Improved strata handling**
   - Visual strata legend on map
   - Auto-detect spatial join between AOI and strata

---

## Non-goals

- Full variance estimators for complex designs
- Multi-date change detection sampling
- Server-side processing or authentication

---

## Validation Checklist

Before release:
- [x] `npx htmlhint index.html` passes
- [ ] Manual test: All sampling methods generate points
- [ ] Manual test: File uploads work (AOI, Strata, Results)
- [ ] Manual test: Analysis tab calculates accuracy
- [ ] Manual test: All exports download correctly
