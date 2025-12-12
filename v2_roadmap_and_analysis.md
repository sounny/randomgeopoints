# Random GeoPoints v2: Comprehensive Review & Development Roadmap

**Date:** December 12, 2025
**Context:** Synthesis of project references (`report.md`, `dissertation.md`) vs. Current `index.html` Implementation.
**Status:** The current tool is significantly more advanced than a basic generator, having already implemented Spatially Balanced (Halton) and Plot Sampling. The focus of v2 is **Analysis**, **Response Design**, and **Workflow**.

---

## Part 1: Gap Analysis (Current Code vs. Literature)

### 1. Sampling Design (Mostly Solved)
- **Literature:** Stevens & Olsen (2004), Radoux & Bogaert (2017).
- **Current State:** The tool *already* supports `balanced` (Halton) and `plots` (Square Plots).
- **Remaining Gap:** The "Spatially Balanced" implementation is a simple Halton sequence. It lacks the flexibility of full **GRTS** (Generalized Random Tessellation Stratified) for spatially balanced *stratified* designs or legacy site inclusion.
- **Action:** Refine the `balanced` method to support stratification (currently only SRS-like).

### 2. Response Design (Major Gap)
- **Literature:** Stehman & Czaplewski (1998), Foody (2013). "Response design is as important as sampling design."
- **Current State:** The tool generates points and says "good luck." No support for defining *how* to label.
- **Missing:** A way to generate a **Protocol Sheet** or **Field Form** schema. No support for **Time-Series** (augmenting an existing sample for change detection) which is a critical gap identified in the review.
- **Action:** Add a "Protocol Builder" and "Longitudinal Sampling" mode.

### 3. Analysis & Estimation (Critical Gap)
- **Literature:** Olofsson et al. (2014) is the industry standard for *analysis*.
- **Current State:** Users must export data and do the math elsewhere (Excel/R).
- **Missing:** **Olofsson Estimator** (Area-weighted accuracy), **Standard Errors**, and **Confusion Matrices**. Also missing is the ability to account for **Reference Data Uncertainty** (Foody 2013).
- **Action:** Build a full "Analyze" tab.

### 4. Pre-Field Validation (New Opportunity)
- **Literature:** Spatial autocorrelation violates independence assumptions (Congalton 1988).
- **Current State:** Shows NNI (Nearest Neighbor Index).
- **Missing:** More robust checks like **Moran's I** on the generated sample to flag potential clustering issues *before* the user leaves the office.
- **Action:** Add "Spatial Health Check" to the results panel.

---

## Part 2: Development Plan (Version 2.0)

**Objective:** Transform the tool from a "Generator" to a "Validation Workbench."

### Module A: Analysis Engine (The "Olofsson" Upgrade) - **HIGHEST PRIORITY**
*Closing the loop: From generation to results.*

- [ ] **Task A1: Analysis UI & Data Import.**
    *   *Detail:* New "Analyze" tab. Dropzone for CSV/GeoJSON containing `Map_Label` and `Ref_Label`.
- [ ] **Task A2: Confusion Matrix Generator.**
    *   *Detail:* Auto-generate a formatted Cross-Tabulation table from uploaded data.
- [ ] **Task A3: Olofsson Estimator Implementation.**
    *   *Detail:* Implement equations 1-6 from Olofsson et al. (2014):
        *   Area-weighted Proportions.
        *   Overall, User's, and Producer's Accuracy.
        *   **Standard Errors** (Confidence Intervals).
        *   **Adjusted Area Estimates** (Bias-corrected area).
- [ ] **Task A4: Reference Error Adjustment.**
    *   *Detail:* Add an "Advanced" input for "Reference Confidence" (0.0-1.0) to widen CIs based on imperfect ground truth (Foody gap).

### Module B: Response Design & Protocols
*Methodological rigor before the field.*

- [ ] **Task B1: Protocol Builder.**
    *   *Detail:* A form wizard to define:
        *   MMU (Minimum Mapping Unit).
        *   Labeling Confidence rules.
    *   *Output:* Generate a printable PDF "Field Protocol" alongside the sample map.
- [ ] **Task B2: Longitudinal/Time-Series Mode.**
    *   *Detail:* "Augment Sample" feature. Upload an existing GeoJSON sample (Year 1) and generate new points only for "Changed" strata in Year 2, while keeping "Stable" points. (Addresses the Time-Series gap).

### Module C: Advanced Visualization & Stats
*Helping users make better decisions.*

- [ ] **Task C1: Power Analysis Visualizer.**
    *   *Detail:* Interactive chart: "Sample Size vs. Margin of Error." Show the diminishing returns curve so users don't just guess "50 points."
- [ ] **Task C2: Spatial Health Check (Moran's I).**
    *   *Detail:* Calculate Global Moran's I on the sample distribution relative to the strata. Flag if the sample is statistically clustered (p < 0.05).

### Module D: UX Polish
- [ ] **Task D1: Tabbed Interface.**
    *   *Detail:* Split into `1. Design` | `2. Label/Protocol` | `3. Analyze`.
- [ ] **Task D2: Persisted State.**
    *   *Detail:* Use `localStorage` to save the active AOI and Settings.

---

## Part 3: Immediate Execution Plan

1.  **Refactor UI:** The current single-column sidebar is too crowded. Move to a Tabbed interface to make room for "Analyze".
2.  **Implement Task A1 & A2 (Analysis Basics):** Allow data upload and show the Confusion Matrix. This is the highest value addition.
3.  **Implement Task C1 (Power Analysis):** This is a high-impact visual for the "Design" phase.