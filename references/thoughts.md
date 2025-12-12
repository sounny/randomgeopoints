# Research Gaps & Tool Opportunities
**Based on review of:** `report.md`, `dissertation.md`, `Random Point and Area Sampling...Literature Review.pdf`

This document outlines identified gaps in the current academic and practical literature regarding geospatial sampling and accuracy assessment. These represent opportunities for the "Random GeoPoints" tool to provide novel or under-represented solutions.

## 1. The "Accessibility Gap" for Spatially Balanced Designs
* **Observation:** The literature (Stevens & Olsen 2004, Robertson et al. 2013, dissertation §7.5) strongly advocates for Spatially Balanced Sampling (GRTS, BAS) as statistically superior to simple random or systematic sampling, especially when spatial autocorrelation is present.
* **The Gap:** While R packages (`spsurvey`) exist, there is a distinct lack of accessible, client-side, GUI-based tools for generating these complex designs. Most "easy" tools only do simple random or systematic grids.
* **Tool Opportunity:** Implementing a "Lite" version of BAS (Balanced Acceptance Sampling) or GRTS in pure JavaScript would democratize a sophisticated method that is currently locked behind command-line interfaces or complex desktop GIS plugins.

## 2. The "Response Design" Standardization Gap
* **Observation:** Stehman & Czaplewski (1998) and the Literature Review (p. 5) emphasize that *Response Design* (how a label is assigned) is as critical as sampling design. Inconsistencies here (e.g., how to handle mixed pixels, minimum mapping units) are a major source of error.
* **The Gap:** There is little software support for *formalizing* the response design. Tools generate points but don't help users define *what to do* with them.
* **Tool Opportunity:** The tool could include a "Protocol Builder" module. When exporting the sample, it could generate a "Field Sheet" or "Protocol PDF" that forces the user to define:
    *   "What is the MMU?"
    *   "How do we handle edge cases?"
    *   "What is the confidence threshold?"
    *   This moves the tool from a "coordinate thrower" to a "methodology enforcer."

## 3. The "Rare Class" Visualization Gap
* **Observation:** Stratified sampling is essential for rare classes (Congalton 1991, Olofsson et al. 2014). However, the dissertation (§10.3) notes the conflict between "Proportional Allocation" (good for Overall Accuracy) and "Equal Allocation" (good for Class Accuracy).
* **The Gap:** Users often guess at sample sizes. There is a lack of interactive visualizers that show *diminishing returns*.
* **Tool Opportunity:** A "Power Analysis" visualizer. If the user uploads a strata map, the tool could simulate: "If you sample 50 points in Class X, your margin of error is ±5%. If you sample 100, it becomes ±3.5%." Visualizing this trade-off *before* sampling is a feature absent in most simple generators.

## 4. The "Post-Sampling" Spatial Autocorrelation Check
* **Observation:** The Literature Review (p. 12) mentions that classical estimators assume independence, but spatial autocorrelation often violates this. The review suggests "Geostatistical techniques" as a frontier.
* **The Gap:** Few validation tools automatically check the *generated sample* for spatial independence *before* the user goes to the field.
* **Tool Opportunity:** The tool currently calculates NNI (Nearest Neighbor Index). It could be expanded to calculate a quick **Moran's I** or **Ripley's K** on the *generated sample points* relative to the strata. This would give the user an immediate "Red Flag" if their random sample accidentally clustered in a way that might violate independence assumptions.

## 5. The "Time-Series" Validation Gap
* **Observation:** The Literature Review (p. 11, "Scaling Sampling to Big Data") and dissertation (§15.2) identify validation of time-series/change maps as a major open question. "Does one validate every year separately... or leverage samples across time?"
* **The Gap:** Standard tools treat maps as static snapshots. There is no standard "workflow" for generating a sample that is stable *over time* (e.g., a permanent panel of plots) vs. dynamic.
* **Tool Opportunity:** A "Longitudinal Sample" feature. Allow users to import a *previous* sample and "augment" it for a new year (e.g., "Keep these 50 unchanging points, add 20 new random points for the areas that changed"). This directly addresses the "Change Detection" validation gap.

## 6. The "Reference Error" Uncertainty Gap
* **Observation:** Foody (2013, 2024) and the dissertation (§11.1) argue that "Ground Truth" is rarely perfect. Reference data error biases accuracy estimates.
* **The Gap:** Almost all calculator tools assume the reference data is 100% correct. There are no easy calculators that allow a user to input "Reference Reliability" (e.g., "I am only 90% sure of my field labels") and see how that widens the confidence intervals of the final map accuracy.
* **Tool Opportunity:** An "Advanced Analysis" mode that includes a parameter for **Reference Error** in the Olofsson estimator, giving a more honest (and wider) confidence interval.
