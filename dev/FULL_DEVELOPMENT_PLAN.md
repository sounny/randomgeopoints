# Random GeoPoints: Full Development Plan

**Version:** 1.0
**Date:** December 14, 2025
**Status:** Approved

---

## 1. Scope Definition

### 1.1 Project Mission
To democratize scientifically rigorous geospatial accuracy assessment by providing an accessible, client-side-only web tool. The tool bridges the gap between statistical theory (Olofsson et al., 2014) and practical application, enabling users to generate probability-based samples, define response design protocols, and calculate area-weighted accuracy metrics without requiring GIS software or server infrastructure.

### 1.2 In-Scope
*   **Sampling Generation:** Implementation of Simple Random (SRS), Stratified Random (Proportional & Equal Allocation), and Spatially Balanced (BAS/Halton) sampling within user-defined Areas of Interest (AOI).
*   **Geospatial Processing:** Client-side handling of GeoJSON inputs, point-in-polygon validation, and **equal-area projection** transformations (critical for unbiased density).
*   **Response Design:** Interactive creation of "Field Protocols" (MMU, confidence rules) and export of structured Field Sheets (PDF/Markdown).
*   **Analysis Engine:** Calculation of Olofsson et al. (2014) accuracy metrics, including confusion matrices, User's/Producer's accuracy, and **area-weighted standard errors**.
*   **Diagnostics:** Pre-sampling spatial health checks (Nearest Neighbor Index) and post-sampling analysis checks.
*   **Privacy:** Zero-data-retention policy; all processing occurs locally in the user's browser.

### 1.3 Out-of-Scope
*   **Backend Database:** No server-side storage of user data or maps.
*   **Multi-User Collaboration:** No real-time syncing between users.
*   **Image Hosting:** The tool will not host or serve satellite imagery/basemaps beyond standard web tile services (e.g., OSM, Carto).

---

## 2. Feature Prioritization (MoSCoW)

| Priority | Feature | Description |
| :--- | :--- | :--- |
| **Must Have** | **Stratified Sampling Logic** | Core requirement. Ability to parse uploaded strata polygons, calculate areas, and allocate samples ($n_h$) based on Proportional or Equal strategies. |
| **Must Have** | **Analysis Engine (Olofsson)** | Implementation of Eq. 1-6 (Olofsson et al. 2014) to compute accuracy and standard errors using mapped area weights. |
| **Must Have** | **Equal-Area Projection** | Integration of `Proj4js` to ensure sampling occurs in a metric planar projection, preventing lat/lon density bias. |
| **Must Have** | **Protocol Export** | Generation of a downloadable "Field Guide" (PDF/Text) containing the response design rules. |
| **Should Have** | **Power Analysis Visualizer** | Interactive chart showing margin of error vs. sample size trade-offs per stratum. |
| **Should Have** | **Time-Series Augmentation** | "Longitudinal Mode" to import a previous year's sample and add new points for changed strata. |
| **Could Have** | **Spatial Diagnostics** | Moran's I calculation to detect clustering in error distribution. |
| **Could Have** | **Hexagonal Grids** | Alternative systematic sampling frames. |

---

## 3. Technical Stack Selection

*   **Frontend Framework:** Vanilla JavaScript (ES6+) with HTML5/CSS3. No heavy build step (React/Vue) required initially to maintain "zero-install" simplicity, though modular structure will be enforced.
*   **Map Visualization:** **Leaflet.js** (Lightweight, robust).
*   **Geospatial Logic:** **Turf.js** (Client-side geospatial analysis).
*   **Projections:** **Proj4js** (Crucial for projecting WGS84 inputs to Albers/UTM for accurate area/density calculations).
*   **PDF Generation:** **jspdf** (For generating professional Field Sheets).
*   **Styling:** Modern CSS Variables + Flexbox/Grid (Glassmorphism dark theme existing).
*   **Testing:** **Jest** (for logic/math verification) + **Puppeteer** (for E2E browser tests).

---

## 4. Architecture Design

### 4.1 Modular Architecture
The application is structured as a Single Page Application (SPA) with distinct functional modules:

1.  **`InputManager`**: Handles File API interactions, GeoJSON parsing, and topology validation (checking for self-intersections).
2.  **`ProjectionEngine`**: Wraps `Proj4js`. Auto-detects UTM zones or applies standard Equal-Area Albers based on AOI centroid.
3.  **`SamplingEngine`**:
    *   *StratifiedAllocator*: Calculates $n_h$ based on inputs.
    *   *PointGenerator*: Implementation of SRS, Halton, and Grid algorithms.
4.  **`AnalysisEngine`**: Pure mathematical module implementing the Olofsson estimator matrix operations.
5.  **`ProtocolBuilder`**: Manages form state for response design and generates PDF artifacts.
6.  **`UIManager`**: DOM manipulation and Leaflet layer management.

### 4.2 Data Flow
1.  **User Input** (GeoJSON) -> **InputManager** (Validate)
2.  **ProjectionEngine** (Reproject to Metric CRS) -> **SamplingEngine** (Generate Points in Metric Space)
3.  **ProjectionEngine** (Reproject back to WGS84) -> **UIManager** (Display on Map)
4.  **ProtocolBuilder** (Attach Metadata) -> **Export** (GeoJSON/CSV)
5.  **External Labeling** (User Workflow)
6.  **Analysis Input** (Labeled CSV) -> **AnalysisEngine** (Compute Metrics) -> **UIManager** (Render Tables)

---

## 5. Task Breakdown

### Phase 1: Foundations & Core Sampling (Weeks 1-2)
*   [ ] Integrate `Proj4js` and implement `ProjectionEngine` to handle on-the-fly reprojection.
*   [ ] Refactor `SamplingEngine` to support true Stratified Sampling (parsing strata attributes, calculating polygon areas).
*   [ ] Implement "Equal" and "Proportional" allocation logic.
*   [ ] Validation: Verify point counts per stratum match targets.

### Phase 2: Analysis Engine (Weeks 3-4)
*   [ ] Build `AnalysisEngine` module.
*   [ ] Implement Confusion Matrix generation from CSV input.
*   [ ] Implement Olofsson Estimators (Overall, User's, Producer's Accuracy, and Standard Errors).
*   [ ] **Critical:** Unit test math against the numerical examples in Olofsson et al. (2014) Appendix.

### Phase 3: Protocol & Workflow (Week 5)
*   [ ] Upgrade Protocol tab to support rich text inputs (Confidence definitions, edge rules).
*   [ ] Integrate `jspdf` to generate a professional PDF "Field Sheet".
*   [ ] Implement "Save State" (URL parameters or LocalStorage) to persist design settings.

### Phase 4: Polish & Diagnostics (Week 6)
*   [ ] Implement Time-Series "Augmentation" mode (Import existing points -> Keep/Add).
*   [ ] Add Power Analysis Visualizer (Canvas/SVG chart).
*   [ ] Final UI polish (Tabs, Loading states, Error handling).

---

## 6. Timeline Estimation

*   **Total Duration:** ~6 Weeks
*   **Milestones:**
    *   *Week 2:* Stratified Sampling working with Equal-Area projection.
    *   *Week 4:* Analysis Tab fully functional with correct error margins.
    *   *Week 6:* v2.0 Release Candidate with Protocol exports and UI polish.

---

## 7. Resource Allocation

*   **Developer (1):** Full-stack capability (Frontend + Geospatial Logic).
*   **Scientific Reviewer (1):** (Role played by User/Documentation) To verify statistical correctness of the Olofsson implementation.
*   **Assets:** Existing `references/` folder provides all necessary mathematical formulas and theoretical validation.

---

## 8. Risk Assessment

| Risk | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Browser Memory Limits** | High | Large strata files (e.g., complex vegetation maps) may crash the browser. **Mitigation:** Implement simplification (Turf.simplify) or warn users on file size >10MB. Use Web Workers for processing. |
| **Projection Errors** | High | Incorrect projection leads to biased sampling density. **Mitigation:** Use established libraries (`proj4`) and default to industry-standard Albers Equal Area for continental scales. |
| **Math Implementation Errors** | Critical | Incorrect confidence intervals undermine the tool's purpose. **Mitigation:** Strict Test-Driven Development (TDD) using the Olofsson 2014 numerical examples as the ground truth test case. |

---

## 9. Testing Strategy

*   **Unit Testing (Jest):**
    *   Test `AnalysisEngine` with known datasets (Example: Map has 10,000ha, Ref has error matrix X... result must be Y).
    *   Test `SamplingEngine` allocation logic (Total points = sum of strata points).
*   **Integration Testing:**
    *   End-to-end flow: Upload Strata -> Generate -> Export -> Re-import -> Analyze.
*   **Geometry Testing:**
    *   Edge cases: MultiPolygons, Holes, Self-intersecting polygons (ensure clean failure or repair).

---

## 10. Deployment Plan

*   **Platform:** GitHub Pages (ideal for static client-side apps).
*   **CI/CD:** GitHub Actions to run linting and unit tests on push.
*   **Versioning:** Semantic Versioning (v2.0.0).
*   **Docs:** Update `README.md` with a "How to Guide" and "Scientific Basis" section linking to the implemented literature.
