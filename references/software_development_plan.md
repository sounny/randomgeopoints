# Master Software Development Plan: Random GeoPoints

**Status:** Updated Dec 2025
**Basis:** Literature Review, Dissertation Feedback, and NotebookLM Expert Analysis.

## 1.0 Project Overview & Mission

**Mission:** To democratize scientifically rigorous geospatial sampling.
Most existing tools are either too simple (ignoring statistical validity) or too complex (requiring expensive desktop GIS). "Random GeoPoints" aims to fill the "Accessibility Gap" by providing a client-side, browser-based tool that implements advanced probability sampling designs correctly.

**Core Philosophy:**
*   **Scientific Rigor:** Adhere to best practices (Independence, Probability Sampling, Equal-Area Projections).
*   **Accessibility:** Zero-install, offline-capable, vanilla JavaScript.
*   **Education:** Guide the user *why* they are choosing a method (e.g., "Use Stratified Equal Allocation to ensure you don't miss rare classes").

## 2.0 System Architecture

*   **Platform:** Single-Page Application (SPA).
*   **Tech Stack:** Vanilla JavaScript, HTML5, CSS3.
*   **External Dependencies (Minimal):**
    *   `Turf.js` (or similar) for geospatial operations.
    *   **CRITICAL ADDITION:** `Proj4js` (or similar) to handle **Equal-Area Projections**. The "Expert Guide" research confirms that sampling in WGS84 (Lat/Lon) introduces density bias near poles. All sampling calculations must occur in an equal-area projection before being re-projected to WGS84 for output.
*   **Data Privacy:** All processing occurs client-side. No user data is uploaded to a server.

## 3.0 Core Features & Phases

### Phase 1: The Foundation (Completed/Maintenance)
*   **Functionality:**
    *   SRS (Simple Random Sampling) within a bounding box.
    *   Basic Point-in-Polygon validation.
    *   GeoJSON Export.
*   **Validation:** Ensure SRS implementation uses a correct PRNG (Pseudo-Random Number Generator) with a user-definable **Seed** for reproducibility.
*   **State Persistence (New):** Implement URL parameters (e.g., `?seed=12345&method=srs&n=100`) to allow researchers to bookmark or share exact sampling configurations.

### Phase 2: Stratified Sampling & Allocation (High Priority)
*   **Rationale:** Research indicates StRS is the "Gold Standard" for thematic accuracy.
*   **Features:**
    *   **Strata Upload:** User uploads a GeoJSON Polygon file (e.g., Land Cover Map).
    *   **Exclusion Masks (New):** Allow uploading a secondary "No-Go" layer (e.g., water bodies, restricted zones) where points are strictly forbidden.
    *   **Strata Parsing:** Auto-detect unique classes (e.g., "Forest", "Urban").
    *   **Allocation Modes (Crucial):**
        *   **Proportional Allocation:** Default. Samples $n_h$ proportional to area. Best for *Overall Accuracy*.
        *   **Equal Allocation:** Fixed $n$ per class. Best for *User's Accuracy* and rare classes (e.g., Change Detection).
        *   **Custom/Manual:** User overrides counts per stratum.
*   **Visualizer:** A "Power Analysis" chart showing the trade-off between sample size and margin of error per class.

### Phase 3: Advanced Sampling Strategies (New Research)
*   **Systematic Sampling (SS):**
    *   **Why:** Research shows SS guarantees spatial balance and often yields higher precision than SRS if spatial autocorrelation is present.
    *   **Implementation:** Grid-based point generation (Random Start + Fixed Interval).
    *   **Warning:** UI must warn about "Periodicity Bias" (alignment with landscape patterns).
*   **Spatially Balanced / Inhibition Sampling:**
    *   **Why:** To prevent random "clumping".
    *   **Implementation:** Minimum Allowed Distance (MAD) constraint (Rejection sampling).
*   **Multi-Stage Sampling (MSS) (New):**
    *   **Why:** Critical for hierarchical populations or managing cost in large areas (e.g., Provinces -> Facilities).
    *   **Implementation:** Two-stage sampler. 1. Select Primary Sampling Units (PSUs) (e.g., counties). 2. Generate random points ($n_{sub}$) only within selected PSUs.
*   **Change Detection Mode (New):**
    *   **Why:** Change is a "Rare Event" (<10%). Random sampling is inefficient.
    *   **Implementation:** Specialized stratified mode. User uploads "Change Mask". Tool automatically sets high allocation for Change strata and low allocation for Background/No-Change strata.

### Phase 4: Metrics & Statistical Rigor (The "Smart" Layer)
*   **Rationale:** Moving beyond just coordinates to providing statistical context.
*   **Calculated Metrics (Real-time):**
    *   **Design Effect (DEFF):** Estimate the variance inflation caused by clustering (if applicable).
    *   **Effective Sample Size ($n_{eff}$):** Display $n / DEFF$.
    *   **Spatial Autocorrelation Check:** Run a quick neighbor check (e.g., NNI - Nearest Neighbor Index) on generated points to flag potential clustering issues.
*   **Positional Accuracy:**
    *   Add metadata fields for "Expected RMSE" if user knows their map source quality.
*   **Expanded Exports (New):**
    *   **GPX:** For direct upload to handheld GPS units for fieldwork.
    *   **KML:** For visualization in Google Earth (common in "Response Design" workflows like Collect Earth).

### Phase 5: Protocol & Response Design (The "Methodology" Layer)
*   **Rationale:** The "Response Design Gap" identified in research. Users need help defining *how* to label points.
*   **Feature:** "Field Protocol Generator".
    *   User inputs: "What is your MMU (Minimum Mapping Unit)?", "What is your classification confidence?"
    *   Output: A downloadable PDF/Markdown "Field Sheet" explaining the rules for labeling the generated points.
*   **Fuzzy Accuracy Template:** Include a blank "Fuzzy Error Matrix" structure in the CSV export to encourage users to track "Acceptable" vs "Absolute" errors.

## 4.0 User Workflow

1.  **Load AOI:** Drag & Drop GeoJSON.
2.  **Projection Check (Auto):** App converts to Equal-Area internally.
3.  **Choose Strategy:**
    *   *Simple:* "I just need points." (SRS)
    *   *Systematic:* "I need even coverage." (Grid)
    *   *Stratified:* "I have a map and need to validate specific classes." (StRS)
    *   *Multi-Stage:* "I need to cluster my fieldwork." (MSS)
4.  **Configure Allocation:** (If Stratified) Choose Proportional vs Equal.
5.  **Generate:** App runs sampling algorithm with Seed.
6.  **Review:** See Map, NNI, and Estimated $n_{eff}$.
7.  **Export:** GeoJSON (Points), GPX, KML + Protocol Doc.

## 5.0 Future Roadmap (Long Term)
*   **Adaptive Sampling:** Integration with a backend (optional) to allow iterative sampling based on initial findings (advanced).