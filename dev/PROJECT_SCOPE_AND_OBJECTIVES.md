# Project Scope and Objectives: Random GeoPoints

**Version:** 1.0
**Date:** December 14, 2025

## 1. Executive Summary
**Random GeoPoints** is a browser-based software tool designed to democratize scientific rigor in geospatial accuracy assessment. By bridging the gap between complex statistical theory and practical application, it allows users—ranging from students to professional analysts—to generate valid probability samples and calculate defensible accuracy metrics without the need for expensive GIS software or server infrastructure.

## 2. Strategic Objectives
*   **Accessibility:** Provide a "zero-install," offline-capable tool that runs entirely in a standard web browser.
*   **Scientific Validity:** Enforce best practices in spatial sampling (Olofsson et al., 2014), ensuring that users generate statistically sound data by default.
*   **Workflow Standardization:** Move the industry away from ad-hoc point generation toward a structured workflow that includes formally defined Response Designs (protocols) and reproducible Analysis.
*   **Privacy & Security:** Maintain a strict "client-side only" architecture where user data (maps, strata, results) never leaves their device.

## 3. Project Scope

### 3.1 In-Scope Features

#### A. Advanced Sampling Engine
The core function of the tool is to generate unbiased sample locations within a user-defined Area of Interest (AOI).
*   **Simple Random Sampling (SRS):** The baseline probabilistic method.
*   **Stratified Random Sampling:** Allocating samples based on uploaded strata (e.g., land cover classes) using Proportional or Equal allocation strategies to ensure minority classes are represented.
*   **Spatially Balanced Sampling:** Implementing BAS (Balanced Acceptance Sampling) or Halton sequences to maximize spatial independence and reduce clustering.
*   **Equal-Area Projection:** Automatically handling coordinate reprojection (WGS84 $\leftrightarrow$ Albers/UTM) to ensure sampling density is geographically accurate, preventing bias near the poles.

#### B. Response Design & Protocol Builder
Tools to help users define *how* they will validate the data.
*   **Protocol Definition:** A wizard to define Minimum Mapping Units (MMU), confidence levels, and labeling rules.
*   **Field Sheet Export:** Generating professional PDF or Markdown documents that field crews or analysts can use as a guide.

#### C. Analysis & Validation Engine
Transforming raw data into actionable statistics.
*   **Olofsson Estimators:** Implementation of the standard formulas for accuracy assessment (Overall, User's, Producer's Accuracy) weighted by mapped area.
*   **Uncertainty Quantification:** Calculation of Standard Errors and Confidence Intervals for all metrics.
*   **Diagnostics:** Tools to assess the "health" of a sample (e.g., Nearest Neighbor Index) before fieldwork begins.

### 3.2 Out-of-Scope
*   **Backend / Cloud Storage:** The tool will not store user projects, history, or maps on a server. State persistence is handled via local browser storage or file exports.
*   **Collaborative Editing:** No real-time multi-user features (e.g., Google Docs style editing).
*   **Basemap Hosting:** The tool relies on public tile servers (OSM, Carto) and does not host high-resolution satellite imagery itself.
*   **Mobile App:** The tool is a responsive web application, not a native iOS/Android app, though it will be usable on mobile browsers.

## 4. Success Criteria
The project will be considered successful when a user can complete the following "Golden Path" workflow without errors:
1.  **Import:** Drag-and-drop a complex GeoJSON (AOI and Strata).
2.  **Design:** Generate a stratified sample of 100 points, correctly allocated by area.
3.  **Protocol:** Define and download a PDF Field Protocol.
4.  **Analyze:** Upload the labeled results and receive a statistically valid Accuracy Report with confidence intervals.
