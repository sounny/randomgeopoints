# Development Plan – Random GeoPoints

**Last Updated**: December 14, 2024

## Completed Features ✅

### Phase 1: Core Sampling
- [x] Simple Random Sampling (BBox and Polygon)
- [x] Spatially Balanced (Halton sequence)
- [x] Systematic Grid (rectangular)
- [x] Hexagonal Lattice
- [x] Cluster Sampling (two-stage)
- [x] Random Plots (square area units)
- [x] **Stratified Sampling (Proportional)** - NEW
- [x] **Stratified Sampling (Equal Allocation)** - NEW

### Phase 2: UI/UX
- [x] Modern tabbed interface (Design → Protocol → Analyze)
- [x] Drag-and-drop file upload
- [x] Draw-on-map AOI definition
- [x] Power analysis visualization
- [x] Loading overlay with spinner
- [x] **Dark/Light theme toggle** with persistence
- [x] **Scientific info tooltips** (hover for methodology)
- [x] **Sample size calculator** (Cochran's formula)
- [x] **Strata legend with color map** - NEW

### Phase 3: Validation Tools
- [x] Confusion matrix generation
- [x] Overall Accuracy with SE
- [x] User's/Producer's Accuracy per class
- [x] Protocol definition and export

### Phase 4: Data Export
- [x] GeoJSON with metadata
- [x] CSV export
- [x] Protocol text export
- [x] **Design weights** (inclusion_prob, weight) on all outputs - NEW

### Phase 5: Diagnostics
- [x] **NNI calculation** - Real Clark & Evans implementation - NEW
- [x] Strata analysis with proportions

---

## In Progress / Planned 🔄

### Near-term
1. **Area estimation module**
   - Bias-adjusted area estimates per class (Olofsson Eq. 8)
   - Confidence intervals for area

2. **Proj4js integration**
   - Equal-area projection for accurate density
   - Auto-detect UTM zone from AOI centroid

### Medium-term
3. **Enhanced strata handling**
   - Custom per-stratum n overrides
   - Auto-detect spatial join between AOI and strata

4. **Time-Series Augmentation**
   - Import previous year's sample
   - Add new points for changed strata

---

## Non-goals

- Full variance estimators for complex designs (use specialized software)
- Multi-date change detection sampling
- Server-side processing or authentication

---

## Validation Checklist

Before release:
- [x] `npx htmlhint index.html` passes
- [x] All sampling methods generate points correctly
- [x] File uploads work (AOI, Strata, Results)
- [x] Analysis tab calculates accuracy
- [x] All exports download correctly
- [x] Theme toggle persists across sessions
- [x] Stratified sampling allocates by proportion/equal
- [x] NNI displays Clustered/Random/Dispersed correctly
