# Implementation Plan: Enhancing Random GeoPoints App
Based on the literature review on "Random Point and Area Sampling in Geospatial Accuracy Validation", the following features/improvements are planned to enhance the application's scientific validity and utility.

## 1. New Sampling Methods

### A. Systematic Sampling (Rectangular Grid)
*   **Description**: Generate points in a regular rectangular grid pattern.
*   **Rationale**: Ensures complete spatial coverage and is a standard probability sampling design.
*   **Implementation**: 
    *   Add option `systematic-grid` to dropdown.
    *   Use the existing 'Cell size' input to determine spacing.
    *   Logic: Iterate bounds with `x += cell`, `y += cell`. Keep points inside AOI.

### B. Cluster Sampling
*   **Description**: Two-stage sampling. 1. Select random cluster centers. 2. Select points within a radius of each center.
*   **Rationale**: Essential for specific ecological or cluster-based accuracy assessments found in literature.
*   **Implementation**:
    *   Add option `cluster` to dropdown.
    *   Add UI inputs for "Number of Clusters" and "Points per Cluster" (or reuse "Points" as total and split it).
    *   Add UI input for "Cluster Radius".

### C. Stratified Sampling (Per Feature)
*   **Description**: If the input AOI has multiple features (polygons), allow allocating points per feature.
*   **Modes**:
    *   **Proportional**: Points allocated based on feature area.
    *   **Equal**: Same number of points per feature.
*   **Rationale**: "Stratified Random Sampling" is heavily cited for thematic accuracy to ensure rare classes are represented.

## 2. Analysis & Validation Tools

### A. Nearest Neighbor Index (NNI)
*   **Description**: Calculate and display the NNI for the generated points to indicate spatial pattern (Clustered vs Random vs Dispersed).
*   **Rationale**: The literature emphasizes validating the randomness/spatial structure of the sample.

### B. Sample Size Calculator
*   **Description**: A helper to estimate required sample size.
*   **Formula**: Cochran’s formula suitable for simple random sampling proportion (e.g. $N = Z^2 * p * q / e^2$).

## 3. UI Improvements
*   **Modernization**: Refresh the UI with a "Premium" feel (Glassmorphism, gradients) as per general instructions, while keeping the scientific tool utility.
*   **Responsiveness**: Ensure controls wrap neatly.

## Execution Steps
1.  **Refactor**: Prepare the `index.html` `methods` section to handle more diverse inputs (like cluster params).
2.  **Implement Methods**: Add `systematicGrid`, `clusterSampling`, `stratifiedByFeature` functions.
3.  **Implement Validation**: Add `calculateNNI` function and display results in the footer/status area.
4.  **Update UI**: Apply design updates and wire up new controls.
