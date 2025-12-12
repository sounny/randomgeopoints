# Random Point and Area Sampling for Geospatial Accuracy Validation

**Scope and limits.** This document focuses on sampling for geospatial accuracy validation, with emphasis on random point sampling and random area sampling. It also covers common extensions such as stratified random sampling, systematic sampling, cluster and two-stage designs, and spatially balanced probability sampling. It treats thematic accuracy as the main case because that is where the point-versus-area choice matters most in practice. It also includes a shorter section on positional accuracy because many validation programs must report it.

---

## Abstract

Maps look crisp. Their uncertainty usually does not. Accuracy validation exists to make that uncertainty visible and measurable. In remote sensing and GIS, it is rarely feasible to verify every pixel or every polygon. Projects therefore sample a subset of locations or areas, assign each a reference label, and estimate accuracy metrics and their uncertainty. The details matter. A weak sampling design can produce biased accuracy estimates. A weak response design can bake label error into every statistic. A weak analysis design can mix weights and estimators in ways that invalidate confidence intervals.

Two choices drive much of the practice. The first choice is the **sampling unit**. Many studies use **random point sampling**, which often means sampling pixels and comparing the mapped class to a reference class at each sampled location. This choice aligns with classic confusion-matrix analysis and supports simple design-based estimators. Yet it can clash with how maps are made and used. If the map uses a minimum mapping unit larger than a pixel, if positional error shifts pixels, or if class boundaries produce mixed pixels, then single-pixel evaluation can misrepresent the map’s intended meaning.

The second choice is the use of **random area sampling**, meaning selection of spatial areas such as blocks of pixels, field plots, image segments, or polygons. This choice can align the validation unit with the map’s unit of meaning and can reduce field cost through clustering. Yet it introduces technical issues. Area units can vary in size. Equal-probability sampling of polygons estimates object-average accuracy, not area-average accuracy. Analysts must decide which parameter they want and then apply estimators that match that parameter. Work on object-based accuracy assessment shows how to weight polygon sample units by area when the goal is area-based inference and how to use alternative metrics when the goal is object-level performance.

This synthesis builds a cohesive view of point and area sampling inside the standard accuracy assessment triad: sampling design, response design, and analysis and estimation. It draws on foundational work in map accuracy assessment and survey sampling, plus modern good-practice guidelines for land-change area estimation and land product validation. It also connects sampling choices to operational needs such as estimating unbiased class areas, estimating rare change area, mapping spatially variable accuracy, and documenting methods for reproducibility.


**Keywords:** accuracy assessment, thematic accuracy, confusion matrix, probability sampling, stratified sampling, systematic sampling, cluster sampling, spatially balanced sampling, object-based accuracy, area estimation, reference data uncertainty

---

## Table of contents

- [0. Preface and how to use this document](#0-preface-and-how-to-use-this-document)
- [1. Problem statement and research questions](#1-problem-statement-and-research-questions)
- [2. Historical trajectory and canonical concepts](#2-historical-trajectory-and-canonical-concepts)
- [3. Core framework: sampling design, response design, analysis](#3-core-framework-sampling-design-response-design-analysis)
- [4. Populations, frames, and spatial support](#4-populations-frames-and-spatial-support)
- [5. Random point sampling](#5-random-point-sampling)
- [6. Random area sampling](#6-random-area-sampling)
- [7. Sampling designs: SRS, stratified, systematic, cluster, spatially balanced](#7-sampling-designs-srs-stratified-systematic-cluster-spatially-balanced)
- [8. Estimation from the error matrix: accuracy metrics and uncertainty](#8-estimation-from-the-error-matrix-accuracy-metrics-and-uncertainty)
- [9. Area and change-area estimation from accuracy samples](#9-area-and-change-area-estimation-from-accuracy-samples)
- [10. Sample size and allocation](#10-sample-size-and-allocation)
- [11. Reference data, label error, and response design](#11-reference-data-label-error-and-response-design)
- [12. Positional accuracy and boundary quality](#12-positional-accuracy-and-boundary-quality)
- [13. Implementation templates and code sketches](#13-implementation-templates-and-code-sketches)
- [14. Reporting and reproducibility standards](#14-reporting-and-reproducibility-standards)
- [15. Research frontiers grounded in current literature](#15-research-frontiers-grounded-in-current-literature)
- [References](#references)

---

## 0. Preface and how to use this document

### 0.1 Audience

This document targets researchers and practitioners who design or review validation studies for thematic maps (for example, land cover). It assumes basic knowledge of classification and GIS. It explains sampling concepts with survey sampling language because that is the language that supports defensible inference.

### 0.2 What “accuracy validation” means here

I use **accuracy validation** to mean the evaluation of a map or product against a reference classification, with a probability sample that supports inference to a stated population. This aligns with the design-based emphasis in key remote sensing accuracy assessment work (Stehman & Czaplewski, 1998; Stehman, 2000; Olofsson et al., 2014).

### 0.3 A short note about the seed review

The Provided Literature Review gives a clear overview of random point and random area sampling. It stresses probability-based selection and independence from training data. It also summarizes strengths and limits of simple random, stratified, systematic, and cluster sampling across landscapes. Those points align with the classic remote sensing accuracy assessment literature. The review also flags several open problems, such as validation for object-based products and the mismatch between mature theory and uneven practice.

This synthesis expands on those themes with deeper statistical framing, estimator detail, and additional literature on area estimation, object-based validation, and spatially balanced sampling.

---

## 1. Problem statement and research questions

### 1.1 The problem

A map is a model of reality. It compresses a complex world into a finite set of labels or values. Users often treat the map as truth. That creates risk. If a map contains unknown bias, then decisions derived from it can also contain unknown bias.

Accuracy validation aims to reduce that risk. It gives users a statistical statement about how often the map agrees with a reference, and how uncertain that statement is. Yet validation itself can fail. It can fail when teams sample convenience sites, when reference labels contain unknown error, or when analysts use estimators that do not match the sampling design (Congalton, 1991; Stehman & Czaplewski, 1998; Olofsson et al., 2014; Stehman & Foody, 2019).

The Provided Literature Review states this directly. It calls probability-based selection a fundamental principle and warns that non-probability sampling introduces unknown bias (Provided Literature Review, p. 4).

### 1.2 Research questions that organize this synthesis

I use five questions to structure the rest of this document.

1. **What is the population and what is the parameter?** Are we estimating area-based accuracy, object-based accuracy, class area, or change area?
2. **When should we sample points and when should we sample areas?** What conditions make one choice more defensible than the other?
3. **How do sampling designs trade bias, variance, and cost?** When do stratification, systematic sampling, clustering, or spatial balance help?
4. **Which estimators match which designs?** How do we compute accuracy metrics, area estimates, and uncertainty without mixing assumptions?
5. **How does reference data quality affect inference?** What happens when the “reference” contains error or disagreement?

---

## 2. Historical trajectory and canonical concepts

### 2.1 Early recognition that sampling matters

Remote sensing accuracy assessment grew from early land-use mapping projects that could not verify every location. Hord and Brooner (1976) proposed criteria for sample size in land-use map evaluation. Van Genderen and Lock (1977) described statistical sampling procedures for map accuracy testing. Story and Congalton (1986) framed accuracy assessment from the user’s point of view and stressed error matrices as a standard tool (Hord & Brooner, 1976; Van Genderen & Lock, 1977; Story & Congalton, 1986).

The Provided Literature Review summarizes this history. It notes early adoption in the 1970s and the role of the error matrix in those early works (Provided Literature Review, p. 1).

### 2.2 Congalton’s synthesis and the role of representativeness

Congalton (1991) published a key review in *Remote Sensing of Environment*. One of his most important messages is simple: an accuracy estimate only has meaning if the validation sample represents the mapped area. Without representativeness, accuracy metrics become numbers without inference (Congalton, 1991).

The Provided Literature Review echoes this and ties the validity of accuracy results to the sampling approach (Provided Literature Review, p. 2).

### 2.3 Stehman’s design-based perspective

Stehman’s work formalizes thematic map accuracy assessment as finite population sampling. This perspective treats the map domain as a population of units. It treats the sampling design as the source of randomness. It emphasizes design-based inference and correct variance estimation under common designs (Stehman, 1992; Stehman, 1999; Stehman, 2000; Stehman, 2014).

This perspective also clarifies a common confusion. Spatial autocorrelation does not break design-based inference. Instead, it changes how variance behaves under a given design. It motivates designs that improve spatial spread or exploit auxiliary information (Stehman, 1992; Stevens & Olsen, 2004).

### 2.4 The rise of area estimation and calibration

As land cover maps became inputs to inventories and policy, users needed area estimates for each class. Misclassification bias then became central. Czaplewski (1992) warns that rare class area can be severely overestimated when small error rates from common classes leak into rare classes. Czaplewski and Catts (1992) discuss calibration methods that use reference data to correct area or proportion estimates for misclassification error (Czaplewski, 1992; Czaplewski & Catts, 1992).

Later work by Olofsson and colleagues provides practical estimators for simultaneous accuracy and area estimation for land change studies under stratified sampling. These papers heavily influenced current best practice for large-area products (Olofsson et al., 2013; Olofsson et al., 2014).

### 2.5 Modern validation guidance

Multiple guidance streams now exist:

- **Land-change accuracy and area estimation:** Olofsson et al. (2014).
- **Rigorous accuracy assessment issues and reporting:** Stehman and Foody (2019).
- **Object-based accuracy assessment guidance:** Radoux and Bogaert (2017).
- **Remote sensing validation practices survey:** CEOS LPV (2025).

These works share a core claim: probability sampling plus compatible estimators plus transparent reporting produce trustworthy accuracy statements (Olofsson et al., 2014; Stehman & Foody, 2019; Radoux & Bogaert, 2017; CEOS LPV, 2025).

---

## 3. Core framework: sampling design, response design, analysis

### 3.1 The triad

Stehman and Czaplewski (1998) describe three components of an accuracy assessment:

1. **Sampling design:** how you select sample units from the population.
2. **Response design:** how you assign reference labels to those units.
3. **Estimation and analysis:** how you compute accuracy parameters and their uncertainty.

They argue that a probability sampling design is a key element for a statistically rigorous assessment (Stehman & Czaplewski, 1998).

The Provided Literature Review repeats this principle. It emphasizes probability-based selection and warns against ad hoc sampling (Provided Literature Review, p. 4).

### 3.2 A working definition of rigor

In practice, “rigor” has three visible traits:

- known inclusion probabilities
- a documented labeling protocol
- estimators and uncertainty measures that match the design

Stehman and Foody (2019) describe persistent gaps where studies fail on one of these traits. They call for more honest and informative validation and clearer reporting of sampling and estimation (Stehman & Foody, 2019).

### 3.3 Design-based, model-based, and model-assisted inference

Survey sampling theory draws a line between design-based and model-based inference. In design-based inference, the population values are fixed. The sampling design creates randomness. In model-based inference, a stochastic model creates randomness, and the sample may not need random selection.

Spatial sciences often mix the two. Brus and de Gruijter discuss how spatial autocorrelation caused many applied researchers to switch toward geostatistical modeling. They argue that design-based approaches still have meaning, and that independence in design-based sampling differs from independence assumptions in classical statistics (Brus & de Gruijter, 1997).

Model-assisted sampling keeps probability selection but uses auxiliary variables to improve efficiency. In map validation, stratification by map class is a model-assisted move. It uses the map itself as auxiliary information (Särndal et al., 1992; Olofsson et al., 2014).

---

## 4. Populations, frames, and spatial support

### 4.1 Define the population precisely

Accuracy assessment starts by defining the population. Typical cases include:

- **Pixel population:** all pixels in a raster map for a defined date and extent.
- **Polygon population:** all mapped polygons in a vector map.
- **Object population:** all segments or objects produced by image segmentation.

Stehman and Czaplewski stress the need to define the target population and the sampling unit early because all later steps depend on these definitions (Stehman & Czaplewski, 1998).

### 4.2 The sampling frame is the operational version of the population

A sampling frame can be a raster grid, a polygon feature class, or a list of object IDs. Errors in the frame cause coverage error. For example, if masked areas cannot be sampled, then the target population may need to exclude them.

### 4.3 Spatial support and minimum mapping unit

The map’s minimum mapping unit (MMU) and spatial resolution shape the support of meaningful validation. The Provided Literature Review notes that if MMU is larger than a pixel, then a single pixel may not represent the map’s conceptual unit. It recommends matching the reference sample unit to MMU when possible (Provided Literature Review, p. 5).

This point also shows up in object-based guidance. If the map unit is an object, then the validation unit must represent that object. Otherwise, the validation question does not match the map product (Radoux & Bogaert, 2017).

### 4.4 Units, weights, and what “overall” means

A crucial point: “overall accuracy” depends on how you weight units.

- If you weight each pixel equally, you estimate area-based accuracy under equal-area pixels.
- If you weight each polygon equally, you estimate object-average accuracy.
- If you weight polygons by their area, you estimate area-based accuracy for polygon maps.

Radoux and Bogaert (2014) show that ignoring polygon area can shift estimates. The Provided Literature Review notes the same issue and notes the need for area-weighted formulas (Provided Literature Review, p. 4; Radoux & Bogaert, 2014).

---

## 5. Random point sampling

### 5.1 Definition

Random point sampling selects point locations at random across the map domain. In raster validation, those points often correspond to pixels. The Provided Literature Review defines this approach and notes its simplicity and alignment with design-based analysis (Provided Literature Review, p. 3).

### 5.2 Strengths

Random point sampling has three core strengths.

1. **Implementation is simple.** Most GIS tools can draw random points within a boundary.
2. **Estimators are familiar.** Confusion matrices and accuracy metrics map directly onto point samples.
3. **It fits equal-area units.** Each pixel contributes the same area weight in many raster products.

These traits explain why random pixel sampling remains common in the literature and in operational products (Congalton, 1991; Olofsson et al., 2014).

### 5.3 Limits

Random points can be suboptimal when one pixel does not represent the conceptual map unit. The Provided Literature Review notes several causes: MMU larger than pixel, geolocation uncertainty, and mixed pixels. It also notes that some guidance discourages using single pixels as reference units in those cases (Provided Literature Review, p. 3).

### 5.4 Practical fixes and their side effects

Common fixes include:

- interpret a neighborhood window rather than a single pixel
- shift points away from boundaries
- sample plots and assign labels based on dominant cover

These fixes often reduce labeling ambiguity. Yet they also change the parameter. A window-based rule estimates agreement between map pixels and a smoothed reference. It does not estimate agreement between map pixels and a point truth. Analysts must document this choice because it affects comparability across studies (Stehman & Foody, 2019).

---

## 6. Random area sampling

### 6.1 Definition

Random area sampling selects spatial areas at random. The area can be a block of pixels, a plot, a segment, or a polygon. The Provided Literature Review defines random area sampling and links it to cluster sampling for field efficiency and to object-based products for conceptual alignment (Provided Literature Review, p. 3).

### 6.2 Two distinct goals hide inside “area sampling”

Area sampling can target two different goals.

- **Logistics goal:** sample areas because travel cost dominates. The area is a cluster that contains multiple point observations.
- **Conceptual goal:** sample areas because the map unit is an area. The area is itself the unit of analysis (for example, an object or field).

These goals require different estimators. Cluster sampling needs cluster-based variance estimation. Object-level sampling needs object-aware metrics and weights (Stehman, 1997; Radoux & Bogaert, 2017).

### 6.3 Variable polygon size and weighting

When polygon sizes vary, equal-probability sampling of polygons makes each polygon count the same. That estimates object-average accuracy. If the goal is area-based accuracy, analysts need weights that reflect polygon area or inclusion probabilities.

Radoux and Bogaert (2014) demonstrate that polygon size affects primary accuracy indices. They propose area-weighted formulas to correct this. The Provided Literature Review also highlights unequal inclusion probabilities and the need for area weighting (Provided Literature Review, p. 4; Radoux & Bogaert, 2014).

### 6.4 Defining “correct” for an area

Area sampling forces a response design decision. When is an area “correct”? Options include:

- match the dominant reference class to the map label
- require a threshold proportion of area to match
- use fuzzy or partial agreement scores

The Provided Literature Review notes that studies use different criteria and that analysts must define the response rule clearly (Provided Literature Review, p. 4). Radoux and Bogaert (2017) discuss similar issues in object-based accuracy assessment (Radoux & Bogaert, 2017).

---

## 7. Sampling designs: SRS, stratified, systematic, cluster, spatially balanced

This chapter focuses on how to select units. The same sampling-unit choice (point or area) can pair with several designs.

### 7.1 Simple random sampling (SRS)

SRS gives each unit equal selection probability. It yields unbiased estimators and simple analysis. Yet it can miss rare classes, and it can produce spatial clumps by chance. The Provided Literature Review notes that SRS can have higher variance than more structured designs when errors are spatially autocorrelated (Provided Literature Review, p. 6).

Stehman (1999) describes SRS as a baseline probability design for thematic map accuracy assessment (Stehman, 1999).

### 7.2 Stratified random sampling

Stratified random sampling divides the population into strata and samples within each stratum. In map validation, strata often equal map classes. This ensures representation of rare classes. The Provided Literature Review highlights this role and notes that agencies sometimes require a minimum sample count per class (Provided Literature Review, p. 4).

Olofsson et al. (2014) recommend stratified sampling by map class for land change studies because it supports both accuracy and area estimation when analysts apply design weights (Olofsson et al., 2014).

Stehman (2014) addresses a more complex case: strata differ from map classes. This occurs when one stratifies on one map but assesses another. He provides estimators and standard errors for this case (Stehman, 2014).

### 7.3 Systematic sampling and stratified systematic unaligned sampling

Systematic sampling selects units at regular spatial intervals. It provides strong spatial coverage and can reduce variance when errors show spatial autocorrelation. Yet it complicates variance estimation and can bias results if periodic patterns align with the sampling interval (Cochran, 1977; Stehman, 1992).

The Provided Literature Review notes these pros and cons and describes stratified systematic unaligned sampling as a compromise that keeps spread while reducing alignment risk (Provided Literature Review, p. 7).

### 7.4 Cluster and two-stage sampling

Cluster sampling selects groups of units and then samples within them. It reduces travel cost and can match the map’s MMU when clusters represent meaningful areas. The trade-off is correlation within clusters, which reduces effective sample size. The Provided Literature Review explains this and warns that treating clustered points as independent underestimates uncertainty (Provided Literature Review, p. 9).

Stehman (1997) presents methods to estimate standard errors of accuracy assessment statistics under cluster sampling. This addresses the key analytic need in clustered validation designs (Stehman, 1997).

### 7.5 Spatially balanced probability sampling

Spatially balanced designs aim to spread points over space while preserving probability sampling properties. Stevens and Olsen (2004) propose the Generalized Random Tessellation Stratified (GRTS) design. They show how to select a spatially balanced sample that supports design-based inference (Stevens & Olsen, 2004).

Robertson et al. (2013) propose balanced acceptance sampling (BAS). BAS uses a quasi-random sequence with an acceptance mechanism to produce spatially balanced samples and support unequal inclusion probabilities when needed (Robertson et al., 2013).

Software support affects adoption. The `spsurvey` R package implements several spatial sampling tools for design and analysis, including GRTS, stratification, and unequal probabilities (Dumelle et al., 2023).

### 7.6 Landscape structure and design choice

Landscape heterogeneity affects design performance. A large uniform forest may not need heavy stratification for class representation, but it may benefit from spatial spread for precision. Urban mosaics often need stratification to ensure rare classes appear. The Provided Literature Review discusses these patterns across urban, forest, and agricultural settings (Provided Literature Review, pp. 9-11).

Empirical work on NLCD supports similar relationships. Smith et al. (2002) show that classification accuracy relates to patch size and heterogeneity. This supports the idea that error structure changes across landscapes and can motivate stratification or spatial balance (Smith et al., 2002).

---

## 8. Estimation from the error matrix: accuracy metrics and uncertainty

### 8.1 The error matrix as a sample estimate

An error matrix is not a property of the map. It is a sample estimate of a population cross-tabulation between map labels and reference labels. This is the key insight of the finite population sampling perspective (Stehman, 2000).

### 8.2 Notation

Let the population contain $N$ units. Each unit $k$ has:

- a map class $m_k$ in {1, ..., C}, and
- a reference class $r_k$ in {1, ..., C}.

Define the population cell proportion:

$$
p_{ij} = \frac{1}{N}\sum_{k=1}^N I(r_k=i, m_k=j),
$$

where $I(\cdot)$ is an indicator function. The matrix $[p_{ij}]$ is the population error matrix expressed in proportions. Validation estimates this matrix and functions of it.

### 8.3 Accuracy metrics as functions of $p_{ij}$

Define row and column sums:

- $p_{i+} = \sum_j p_{ij}$ (reference class area proportion)
- $p_{+j} = \sum_i p_{ij}$ (map class area proportion)

Then common metrics are:

- Overall accuracy: $OA = \sum_i p_{ii}$
- Producer’s accuracy for class $i$: $PA_i = p_{ii} / p_{i+}$
- User’s accuracy for class $j$: $UA_j = p_{jj} / p_{+j}$

These are area-based definitions. They match pixel populations and also match polygon populations if analysts treat area as the unit of inference (Olofsson et al., 2014; Radoux & Bogaert, 2014).

### 8.4 Estimation under simple random sampling

Under SRS of $n$ units, the sample cell proportion estimator is:

$$
\hat{p}_{ij} = \frac{n_{ij}}{n},
$$

where $n_{ij}$ is the count of sampled units with reference class $i$ and map class $j$. Plug-in estimators then yield $\widehat{OA}$, $\widehat{PA}_i$, and $\widehat{UA}_j$.

Stehman and Czaplewski (1998) caution that this multinomial sampling model only holds for certain designs. Stratified and cluster designs require different variance estimators, and sometimes different point estimators, for some statistics (Stehman & Czaplewski, 1998; Stehman, 1996; Stehman, 1997).

### 8.5 Estimation under stratified random sampling

Let strata $h=1,...,H$ partition the population. Let $W_h = N_h/N$ be the area weight of stratum $h$. Let $n_h$ be the sample size in stratum $h$. Let $\hat{p}_{ij,h} = n_{ij,h}/n_h$ be the within-stratum cell proportion. Then a common estimator of the population cell proportion is:

$$
\hat{p}_{ij} = \sum_{h=1}^H W_h \hat{p}_{ij,h}.
$$

Olofsson et al. (2014) provide these estimators and discuss standard errors under stratified designs for both accuracy and area estimation (Olofsson et al., 2014).

### 8.6 Variance and uncertainty

A key deliverable is uncertainty. Many studies report only point estimates. That is not enough for decision use. Olofsson et al. (2014) recommend reporting confidence intervals for accuracy and area estimates. Stehman and Foody (2019) emphasize the same reporting need (Olofsson et al., 2014; Stehman & Foody, 2019).

Variance formulas depend on the design.

- Under SRS, variance of a proportion follows standard finite population sampling results (Cochran, 1977).
- Under stratified sampling, variance uses within-stratum variance and stratum weights (Cochran, 1977; Olofsson et al., 2014).
- Under cluster sampling, variance must account for cluster-to-cluster variation, not just within-cluster variation (Stehman, 1997).

### 8.7 Chance-corrected agreement and kappa

Cohen (1960) introduced kappa as a chance-corrected agreement measure for nominal scales. Map accuracy assessment literature adopted kappa widely. Yet two issues matter.

First, stratified designs break the multinomial assumptions behind standard kappa variance formulas. Stehman (1996) derives estimators for kappa and its variance under stratified random sampling (Stehman, 1996).

Second, interpretation of kappa can be problematic. Pontius and Millones (2011) argue that kappa can hide important aspects of disagreement. They propose quantity disagreement and allocation disagreement as alternatives (Pontius & Millones, 2011).

Because of this, modern guidance often centers on OA, UA, PA, plus area-adjusted estimates, rather than kappa alone (Olofsson et al., 2014; Stehman & Foody, 2019).

---

## 9. Area and change-area estimation from accuracy samples

### 9.1 Why map area counts can mislead

A classified map reports class areas through pixel counts or polygon areas. If the map has error, those areas contain bias. This matters most when a class is rare. A small fraction of a common class misclassified into a rare class can dominate the rare class estimate (Czaplewski, 1992).

### 9.2 Using the validation sample to estimate unbiased area

Olofsson et al. (2013) and Olofsson et al. (2014) show how to use a probability validation sample to estimate area and accuracy at the same time. The sample gives an estimate of the population error matrix in terms of area proportions. From that matrix, analysts can estimate the reference-based class area proportion $p_{i+}$. Multiply by total area to estimate class area. Report uncertainty with standard errors from the sampling design (Olofsson et al., 2013; Olofsson et al., 2014).

This approach aligns with design-based inference. It also aligns with the Provided Literature Review’s emphasis on probability sampling and weighted estimators under stratification (Provided Literature Review, p. 4).

### 9.3 Calibration and misclassification correction

Calibration methods treat the map as a biased measurement of class proportions and use reference data to correct it. Czaplewski and Catts (1992) compare calibration estimators and discuss how sample size affects calibration performance (Czaplewski & Catts, 1992).

These methods connect accuracy assessment to survey calibration and measurement error in survey statistics. They also show why validation samples can serve two roles: accuracy reporting and bias correction for area estimation.

### 9.4 Change maps and the “rare change” problem

Land change often occupies small area. This makes change hard to estimate and validate. Stehman (2012) studies the effect of sample size allocation for stratified sampling in a two-class change map (change vs no-change). He shows a core trade-off: an allocation that improves change user’s accuracy does not necessarily minimize variance for change area estimation (Stehman, 2012).

Reference data error makes the problem worse. Foody (2013) shows that reference error can cause large mis-estimation of change area, especially when change is rare (Foody, 2013).

---

## 10. Sample size and allocation

### 10.1 The persistence of rules of thumb

The Provided Literature Review describes a common benchmark: at least 50 samples per class. It traces this to Hay (1979) and Congalton (1991). It notes that the rule balances effort and precision but does not come from a single formula (Provided Literature Review, p. 4).

Rules of thumb can help in planning. They should not replace uncertainty targets.

### 10.2 A basic planning formula for a proportion

A common planning formula for a proportion $p$ with margin of error $d$ at confidence level with normal critical value $z$ is:

$$
n \approx \frac{z^2 p(1-p)}{d^2}.
$$

Olofsson et al. (2014) and Radoux and Bogaert (2017) use this style of planning discussion. It is useful as a starting point (Olofsson et al., 2014; Radoux & Bogaert, 2017).

This formula assumes independent samples and does not encode stratification goals. It also does not encode cluster correlation.

### 10.3 Allocation across strata

Stratification requires allocation. Options include:

- **Equal allocation** across classes. This supports class-specific accuracy. It requires weighting for overall accuracy.
- **Proportional allocation** to stratum area. This supports efficient estimation of overall area-based metrics. It can starve rare classes.
- **Optimal allocation** based on within-stratum variance and cost. This follows standard survey sampling results (Cochran, 1977).

For change maps, Stehman (2012) provides a concrete illustration of how allocation goals can conflict. That study frames allocation as a multi-objective problem (Stehman, 2012).

### 10.4 Cluster designs and effective sample size

Cluster sampling reduces field cost but increases variance because samples inside a cluster correlate. Survey sampling theory often summarizes this with a design effect. A common approximation is:

$$
\text{Deff} \approx 1 + (m-1)\rho,
$$

where $m$ is cluster size and $\rho$ is intra-cluster correlation. Effective sample size becomes approximately $n/\text{Deff}$ (Cochran, 1977; Särndal et al., 1992).

The Provided Literature Review explains the same effect in plain terms and warns that analysts must adjust variance estimation under clustering (Provided Literature Review, p. 9).

---

## 11. Reference data, label error, and response design

### 11.1 Reference data are measurements, not truth

Many validation studies assume the reference label is correct. This assumption often fails. Reference data can contain interpreter error, geolocation error, and temporal mismatch. When reference error exists, reported map accuracy can be biased in either direction, and uncertainty statements can be too narrow.

Foody (2013) shows that reference data error can lead to large mis-estimation of land cover change area. Foody (2024) argues more broadly that “ground truth” is usually not available and that reference error can cause major bias in both accuracy and area estimates (Foody, 2013; Foody, 2024).

### 11.2 Independence from training data

Validation data should be independent from training data. The Provided Literature Review states that using the same locations for training and validation produces optimistic bias (Provided Literature Review, p. 5).

Operational programs implement this separation. For example, NLCD 2006 accuracy assessment used a stratified random sample and reference interpretation from high-resolution imagery, separate from training processes (Wickham et al., 2013).

### 11.3 Labeling rules must match the sampling unit

Response design must specify how to label a sample unit.

- For a pixel sample, specify how to interpret mixed pixels.
- For a plot or block, specify how to define dominant class and what threshold to use.
- For an object sample, specify whether correctness means label match, boundary match, or both.

The Provided Literature Review stresses matching the support of reference data to the map unit and calls for clear rules near class boundaries (Provided Literature Review, p. 5).

### 11.4 Citizen and crowd reference data

Foody (2015) reviews citizen contributed reference data for land cover accuracy assessment. He frames a key trade-off. Citizen data can expand coverage and sample size, but it can carry uncertain quality. That makes response design and quality control essential when using citizen data for inference (Foody, 2015).

---

## 12. Positional accuracy and boundary quality

### 12.1 Why positional accuracy belongs in a sampling discussion

Sampling does not only support thematic accuracy. It also supports positional accuracy of points, lines, and surfaces. Many geospatial standards treat data quality as multi-dimensional, and positional accuracy often appears as a required element in product specifications.

ISO 19157 describes geographic information data quality elements, including positional accuracy and thematic accuracy. The full standard is not always freely accessible, but public descriptions identify these categories as part of geospatial data quality reporting (ISO 19157-1:2023, ISO public page).

The American Society for Photogrammetry and Remote Sensing (ASPRS) publishes positional accuracy standards for digital geospatial data, which practitioners use to report horizontal and vertical accuracy in a consistent way (ASPRS Positional Accuracy Standards, 2014).

### 12.2 Sampling for positional accuracy of point features

A common approach samples a set of well-defined checkpoints, then compares mapped coordinates to surveyed coordinates. Analysts compute error vectors and summarize them with root mean square error (RMSE) or confidence measures. The sampling logic mirrors thematic validation: define a population of potential checkpoints, select a probability sample when possible, and report uncertainty consistent with the design.

### 12.3 Boundary quality for polygon maps

Boundary quality is central for object-based products. A polygon can have the right label and the wrong boundary. Radoux and Bogaert (2017) discuss object-based validation practices, including methods that evaluate boundaries with buffers or edge-based measures. They emphasize that thematic and boundary accuracy answer different questions and may require different sampling and metrics (Radoux & Bogaert, 2017).

---

## 13. Implementation templates and code sketches

This chapter gives implementation patterns. It uses short sketches. They illustrate logic. They are not full software packages.

### 13.1 Template A: Stratified random point sampling for a raster map

**Goal:** estimate OA, UA, PA with confidence intervals. Also estimate error-adjusted class area.

**Design:** stratified by map class. Within each class, sample points at random.

**Analysis:** compute weighted error matrix and area estimates as in Olofsson et al. (2014).

**Pseudo-code (Python-like):**

```python
# Inputs:
# - raster map with classes 1..C
# - polygon AOI
# - desired sample size per class: n_h

samples = []
for class_id in classes:
    mask = (map_raster == class_id) & within(AOI)
    candidate_pixels = pixel_indices(mask)
    chosen = simple_random_sample(candidate_pixels, n_h[class_id])
    samples.extend(chosen)

# Response design:
# For each chosen pixel, assign reference label from field or high-res imagery.
ref_labels = interpret_reference(samples)

# Analysis design:
# Compute within-stratum cell proportions and then weighted sums.
error_matrix = estimate_weighted_error_matrix(samples, map_labels, ref_labels, stratum_weights)
metrics = compute_OA_UA_PA(error_matrix)
area_estimates = compute_error_adjusted_area(error_matrix, total_area)
uncertainty = compute_design_based_SE(error_matrix, design='stratified')
```

### 13.2 Template B: Two-stage cluster sampling for field validation

**Goal:** reduce travel cost while preserving probability inference.

**Design:**

- Stage 1: sample clusters (for example, 10 km by 10 km blocks) with known inclusion probabilities.
- Stage 2: sample points inside each selected cluster.

**Analysis:** use cluster-aware variance estimation as in Stehman (1997).

### 13.3 Template C: Object-based sampling for a polygon map

**Goal:** choose between area-based and object-based accuracy and estimate the chosen parameter.

**Design choices:**

- If the target is area-based accuracy, sample polygons or polygon parts with inclusion probabilities proportional to area, or sample polygons equally and weight by polygon area in estimation (Radoux & Bogaert, 2014).
- If the target is object-based accuracy, sample polygons with equal probability and compute object-level metrics (Radoux & Bogaert, 2017).

### 13.4 Template D: Spatially balanced sampling for large domains

**Goal:** achieve broad spatial coverage while maintaining probability inference.

**Design:** use GRTS or BAS. Implement with `spsurvey` in R when possible (Stevens & Olsen, 2004; Robertson et al., 2013; Dumelle et al., 2023).

---

## 14. Reporting and reproducibility standards

### 14.1 What a reader needs to reproduce a validation study

The Provided Literature Review stresses transparency and reproducibility. It notes that many studies omit key details about sampling and reference labeling. It recommends clear descriptions and sharing of sample locations and labels when possible (Provided Literature Review, p. 5).

Stehman and Foody (2019) make similar arguments. They frame clear reporting as part of rigor (Stehman & Foody, 2019).

A minimal reporting package should include:

- a map of the sampling frame and exclusions
- sampling unit definition (pixel, plot, object)
- sampling design (SRS, stratified, systematic, cluster, spatially balanced)
- inclusion probabilities and stratum weights
- sample allocation and final achieved sample sizes
- reference data sources and labeling rules
- error matrix with weights when needed
- accuracy metrics with uncertainty (SE or CI)
- area estimates with uncertainty when relevant
- a data file of sample locations and labels when sharing is possible

### 14.2 A warning about a common failure

A common failure happens when teams stratify the sample and then compute standard errors as if SRS. This underestimates uncertainty and can bias some derived metrics. Olofsson et al. (2014) provide estimators that match stratified designs. Stehman (1997) provides methods for cluster sampling. Use those tools. Do not mix designs and estimators (Olofsson et al., 2014; Stehman, 1997).

---

## 15. Research frontiers grounded in current literature

This section lists research needs that appear in the cited literature and in the Provided Literature Review. It avoids claims that require results I cannot verify here.

### 15.1 Reference data uncertainty and error propagation

Foody (2013) and Foody (2024) show that reference label error can distort accuracy and area estimates. This motivates methods that model or measure reference uncertainty and propagate it through estimators. Work in survey sampling on measurement error also connects to this problem, but the mapping-specific literature still treats it as a major issue (Foody, 2013; Foody, 2024).

### 15.2 Validation for time series products

The Provided Literature Review notes that multi-dimensional stratification across classes, regions, and time remains an open design question for time series land products (Provided Literature Review, p. 13). Operational programs now produce annual land cover. Validation must evolve to handle time as a core dimension rather than an afterthought.

### 15.3 Spatially variable accuracy and accuracy surfaces

Accuracy varies across landscapes. Smith et al. (2002) link accuracy to patch size and heterogeneity in NLCD. Steele et al. (1998) propose estimating and mapping misclassification probabilities. Together, these works suggest that global accuracy metrics hide important spatial structure and that local accuracy models can improve map use when users need spatially explicit uncertainty (Smith et al., 2002; Steele et al., 1998).

### 15.4 Wider adoption of spatially balanced designs

Spatially balanced sampling designs such as GRTS and BAS can reduce variance and improve coverage in spatial monitoring. They also fit validation problems where error clustering exists. The literature offers strong theory and software support. Yet many validation studies still rely on simple random points or ad hoc designs. This remains a practical frontier where education and tooling can change practice (Stevens & Olsen, 2004; Robertson et al., 2013; Dumelle et al., 2023; CEOS LPV, 2025).

### 15.5 Object-based boundary accuracy

Object-based products need both label accuracy and boundary quality. Radoux and Bogaert (2017) provide guidance, but the diversity of object types and applications means this remains an active area. Sampling strategies that target boundaries, plus metrics that separate label and geometry performance, remain important topics (Radoux & Bogaert, 2017).

---


## 16. Criteria for an effective sampling strategy

The Provided Literature Review offers a useful checklist of criteria that a sampling strategy should satisfy. It lists several practical and statistical requirements. Together, they form a good test for whether a study supports defensible inference (Provided Literature Review, pp. 4-5).

### 16.1 Probability-based selection

A probability sample assigns each unit a known, non-zero chance of selection. This supports design-based estimators and uncertainty measures. The seed review notes that non-probability selection creates unknown bias and blocks strong inference (Provided Literature Review, p. 4).

This idea anchors the design-based literature on map accuracy assessment (Stehman & Czaplewski, 1998; Stehman, 2000).

### 16.2 Adequate spatial coverage

The seed review stresses adequate spatial coverage. Random samples can cluster by chance. Systematic, stratified systematic unaligned, or spatially balanced designs can improve coverage and can improve precision when error shows spatial clustering (Provided Literature Review, p. 4; Stehman, 1992; Stevens & Olsen, 2004).

### 16.3 Stratification for rare classes and heterogeneous landscapes

The seed review recommends stratification when classes are imbalanced or when landscapes are heterogeneous. Stratification helps ensure rare classes appear in the sample. It can also reduce variance for class-specific accuracy metrics (Provided Literature Review, p. 4; Olofsson et al., 2014).

### 16.4 Sufficient sample size and precision targets

The seed review stresses sufficient sample size. It notes the 50-per-class rule and warns that small samples create unstable accuracy estimates (Provided Literature Review, p. 4).

Survey sampling texts make the same point through margin-of-error targets and sample size planning formulas (Cochran, 1977). In land cover validation, Olofsson et al. (2014) connect sample size planning to confidence intervals for accuracy and area (Olofsson et al., 2014).

### 16.5 Bias control beyond sampling

The seed review treats bias control as broader than sampling. It highlights the need for independent validation data and for reference data that match the map unit and MMU (Provided Literature Review, p. 5).

Foody’s work shows another source of bias: reference label error. Even with perfect probability sampling, reference error can bias both accuracy and derived area estimates (Foody, 2013; Foody, 2024).

### 16.6 Cost and practical constraints

The seed review notes a basic reality. Field work costs money and time. Sampling designs must fit budgets. Cluster sampling often reduces cost but increases variance. This creates a cost-precision trade-off (Provided Literature Review, p. 5; Stehman, 1997).

Spatially balanced designs can also help. They can maintain good coverage with smaller samples, but they also require specialized selection tools and design-aware analysis (Stevens & Olsen, 2004; Dumelle et al., 2023).

### 16.7 Transparency and reproducibility

The seed review emphasizes transparency and reproducibility as core criteria. A validation study should document the sampling design, labeling rules, and estimation methods. It should also share sample locations and labels when possible (Provided Literature Review, p. 5).

Stehman and Foody (2019) frame this as a persistent gap in the applied literature. They call for more complete reporting and more honest statements of uncertainty (Stehman & Foody, 2019).

---


## 17. Worked examples and numeric illustrations

This section gives small numeric examples that illustrate how point versus area weighting changes results. These examples are illustrative. They do not come from a specific published case study.

### 17.1 Example: object-average accuracy versus area-average accuracy

Suppose a polygon map contains two polygons of the same class label.

- Polygon A has area 90 ha. The map label is correct.
- Polygon B has area 10 ha. The map label is wrong.

If you compute object-average accuracy, you treat each polygon equally. You get:

- object-average accuracy = 1 correct out of 2 polygons = 0.50

If you compute area-average accuracy, you weight by polygon area. You get:

- area-average accuracy = 90 ha correct out of 100 ha total = 0.90

Both numbers are correct for their targets. They answer different questions. Radoux and Bogaert (2014) show that this issue matters in real polygon maps, and they provide methods to account for polygon area when estimating primary accuracy indices (Radoux & Bogaert, 2014).

### 17.2 Example: why stratified samples need weights

Suppose a raster map has two classes. Class 1 covers 90% of area. Class 2 covers 10% of area. You draw a stratified sample with 50 points from each class to estimate class-specific accuracy.

If you compute overall accuracy as the proportion correct in the full sample, you implicitly weight the classes equally. That produces an estimate of a different parameter than area-weighted overall accuracy. To estimate area-weighted overall accuracy, you must weight each stratum by its area proportion. Olofsson et al. (2014) provide estimators and standard errors for this case (Olofsson et al., 2014).

### 17.3 Example: rare class area bias from small error rates

Czaplewski (1992) emphasizes a key risk. If a common class has a small misclassification rate into a rare class, the rare class area estimate can inflate strongly. This effect grows when the rare class occupies very small area. This is a main reason that validation samples often support both accuracy reporting and error-adjusted area estimation (Czaplewski, 1992; Olofsson et al., 2014).

---

## 18. Case study patterns from large-area programs and other domains

This section highlights patterns from published large-area validation efforts. It does not try to summarize every program. It aims to show how sampling choices connect to operational goals.

### 18.1 National Land Cover Database (NLCD) example

Large-area land cover programs often use stratified random sampling of pixels, with reference labels interpreted from high-resolution imagery. A USGS summary of NLCD 2006 accuracy assessment reports a stratified random sample of pixels with reference classification from multi-temporal high-resolution imagery. It reports overall accuracies near the high 70% range for Level II classes for 2001 and 2006, and higher accuracies for Level I classes (Wickham et al., 2013; USGS NLCD 2006 accuracy assessment summary).

This case illustrates three themes from the literature:

- Stratification supports class representation.
- High-resolution imagery can support reference labeling at scale.
- Programs often assess both single-date maps and change products, which raises sample allocation questions similar to those studied by Stehman (2012) (Wickham et al., 2013; Stehman, 2012).

### 18.2 Large-area validation frameworks

Wulder et al. (2006) note that national and continental land cover products increased in use, but standard operational validation protocols lagged. They propose a framework for large-area accuracy assessment that emphasizes design choices, reference data, and reporting needs (Wulder et al., 2006).

This aligns with later best-practice guidance that stresses transparent sampling design, response design, and estimator selection (Olofsson et al., 2014; Stehman & Foody, 2019).

### 18.3 Digital soil maps as a related validation problem

Validation of digital soil maps also requires spatial sampling and design-based inference. Brus et al. (2011) discuss sampling for validation of digital soil maps and review sampling strategies that manage spatial structure. This literature parallels map accuracy assessment in remote sensing, even when the response variable is continuous rather than categorical (Brus et al., 2011; de Gruijter et al., 2006).

This cross-domain link matters for two reasons:

- It reinforces that spatial sampling design is a general problem, not a land cover niche.
- It offers additional tools such as stratified spatial sampling and model-assisted approaches that can also inform thematic map validation.

### 18.4 Spatially balanced designs in environmental monitoring

Spatially balanced designs like GRTS emerged from environmental monitoring needs, where broad coverage and trend detection matter. Stevens and Olsen (2004) show how GRTS can achieve spatial balance while supporting probability inference. BAS provides another route to spatial balance (Stevens & Olsen, 2004; Robertson et al., 2013).

Land product validation can adopt these designs when spatial coverage is as important as class representation. CEOS LPV (2025) reviews validation practices across the remote sensing community and documents growing interest in more systematic and standardized validation approaches (CEOS LPV, 2025).

---

## 19. Appendix A: Design-based estimation basics for accuracy assessment

This appendix sketches the design-based estimation view that underlies much of the cited map accuracy assessment literature. It is short and practical. For full treatments, see standard sampling texts (Cochran, 1977; Särndal et al., 1992). For mapping-specific connections, see Stehman (2000) and Olofsson et al. (2014) (Stehman, 2000; Olofsson et al., 2014).

### 19.1 Inclusion probability and weights

A probability sampling design assigns each population unit $k$ an inclusion probability $\pi_k = P(k \in s)$, where $s$ is the sample. A design weight is often $w_k = 1/\pi_k$.

If all units have equal probability, then $w_k$ is constant. If some units have higher selection probability, then weights vary and must appear in estimators.

### 19.2 Horvitz-Thompson estimator

The Horvitz-Thompson (HT) estimator estimates a population total $Y = \sum_{k=1}^N y_k$ as:

$$
\hat{Y}_{HT} = \sum_{k \in s} \frac{y_k}{\pi_k}.
$$

For a population mean or proportion, divide by $N$ (or by estimated $N$ if $N$ is not known).

In thematic accuracy assessment, $y_k$ can be an indicator. For example, define $y_k = I(r_k = m_k)$ as 1 for a correct classification and 0 otherwise. Then the population mean of $y_k$ equals overall accuracy under area-based weighting. Under equal-probability pixel sampling, the sample mean estimates OA. Under unequal probabilities, the HT estimator produces a weighted estimate (Stehman, 2000).

### 19.3 Stratified estimators as special cases

Stratified random sampling is a special case where inclusion probabilities are constant inside each stratum. This yields the standard weighted estimator:

$$
\hat{\mu} = \sum_{h=1}^H W_h \bar{y}_h,
$$

where $\bar{y}_h$ is the sample mean in stratum $h$ and $W_h$ is the stratum weight. Olofsson et al. (2014) apply this logic to estimate cell proportions of an error matrix and then compute accuracy and area estimates (Olofsson et al., 2014).

### 19.4 Why “unweighted confusion matrices” can be wrong

If a study draws equal numbers of samples from each map class, then each class gets equal sample weight. If the map classes cover unequal areas, this means the unweighted sample confusion matrix estimates an object- or stratum-average parameter, not an area-average parameter. Olofsson et al. (2014) recommend weighting by area proportions to recover area-based parameters (Olofsson et al., 2014).

---

## 20. Appendix B: Uncertainty estimation by design type

This appendix summarizes how uncertainty depends on design. It does not reproduce every formula because those details depend on the statistic and design. It points to sources that provide full derivations.

### 20.1 Simple random sampling

Under SRS, many accuracy metrics reduce to functions of multinomial cell counts. Standard sampling texts provide variance formulas for means and proportions with finite population correction when needed (Cochran, 1977).

### 20.2 Stratified random sampling

Under stratified sampling, analysts estimate within-stratum cell proportions and then combine them with stratum weights. Olofsson et al. (2014) provide formulas for standard errors for estimated cell proportions and for derived accuracy and area estimates under stratified designs. These formulas form a common basis for reporting confidence intervals in land change validation (Olofsson et al., 2014).

### 20.3 Cluster sampling

Under cluster sampling, the effective sample size drops because of correlation within clusters. Naive formulas that treat all points as independent underestimate variance. Stehman (1997) derives design-based standard error estimators for accuracy assessment statistics under cluster sampling (Stehman, 1997).

### 20.4 Systematic sampling

Systematic sampling complicates variance estimation because the sample is not a set of independent draws. Cochran (1977) discusses this and describes cases where systematic sampling can reduce variance relative to SRS when the population has spatial trends. Stehman (1992) compares systematic and random sampling for map accuracy estimation and highlights both benefits and risks (Cochran, 1977; Stehman, 1992).

### 20.5 Spatially balanced sampling

Spatially balanced designs such as GRTS and BAS aim to approximate the coverage benefits of systematic sampling while preserving probability sampling logic. Stevens and Olsen (2004) provide the statistical basis for GRTS, including design weights. BAS also supports probability inference through its design (Stevens & Olsen, 2004; Robertson et al., 2013).

In practice, analysts should use software that implements the correct design-based variance estimation for the chosen design. The `spsurvey` package provides design and analysis support for several spatial designs (Dumelle et al., 2023).

---

## 21. Appendix C: Spatially balanced sampling in plain language

Spatially balanced sampling means a sample that spreads sites across space more evenly than a simple random sample would. It aims to reduce gaps and clusters while keeping probability sampling properties.

### 21.1 Why spatial balance matters for map validation

Map errors often cluster in space. They can cluster because of topography, sensor effects, training data gaps, or land management patterns. If errors cluster, then a simple random sample can land in one region and miss another region. This can inflate variance and can reduce the credibility of results for local users. Spatial balance improves coverage and can improve precision when error shows spatial structure (Stehman, 1992; Smith et al., 2002).

### 21.2 GRTS in one paragraph

Stevens and Olsen (2004) propose GRTS as a design for spatial monitoring. GRTS creates an ordered list of spatial units that preserves proximity. It then selects a systematic sample from that ordered list with randomization. The result is a sample that spreads across space while keeping known inclusion probabilities (Stevens & Olsen, 2004).

GRTS also supports replacement. If a site becomes inaccessible, the design can provide a nearby substitute with known properties. This matters in field validation and monitoring programs.

### 21.3 BAS in one paragraph

Balanced acceptance sampling (BAS) uses a quasi-random sequence, often based on a Halton sequence, and then applies an acceptance mechanism to achieve the desired sample size and spatial balance. Robertson et al. (2013) present BAS for natural resource sampling and show that it produces well-spread samples with probability sampling properties (Robertson et al., 2013).

### 21.4 Practical advice for validators

Spatially balanced designs help when you need to ensure broad coverage. They do not replace stratification when rare classes matter. Many projects combine stratification and spatial balance. For example, select a spatially balanced sample inside each stratum. Software such as `spsurvey` supports these combinations for point resources and can support polygon resources as well (Dumelle et al., 2023).

---

## 22. Appendix D: Glossary


**Accuracy assessment triad:** the linked components of sampling design, response design, and analysis/estimation (Stehman & Czaplewski, 1998).

**Area-based accuracy:** accuracy parameters defined with each unit weighted by area. For raster pixels of equal size, this means each pixel has equal weight. For polygons, this usually means weighting by polygon area (Olofsson et al., 2014; Radoux & Bogaert, 2014).

**Balanced acceptance sampling (BAS):** a spatial sampling design that aims for spatial balance using quasi-random sequences and an acceptance mechanism (Robertson et al., 2013).

**Cluster sampling:** a design where the sample selects groups of units and then samples within them. It reduces travel cost but increases variance due to within-cluster correlation (Stehman, 1997; Cochran, 1977).

**Confusion matrix / error matrix:** a cross-tabulation of map labels (rows) versus reference labels (columns) for sampled units. It supports estimates of OA, UA, and PA (Congalton, 1991; Stehman & Czaplewski, 1998).

**Design-based inference:** inference that treats population values as fixed and treats the sampling design as the source of randomness. It relies on known inclusion probabilities (Cochran, 1977; Stehman, 2000).

**Frame:** the operational list or representation from which a sample is selected. It should cover the target population without missing or extra units.

**GRTS:** Generalized Random Tessellation Stratified sampling, a spatially balanced probability sampling design for natural resources monitoring (Stevens & Olsen, 2004).

**Inclusion probability ($\pi_k$):** the probability that population unit $k$ appears in the sample under the sampling design.

**Minimum mapping unit (MMU):** the smallest area that the mapping process reliably represents as a distinct polygon or patch. When MMU exceeds pixel size, single-pixel reference labeling may not match the map’s conceptual unit (Provided Literature Review, p. 5).

**Object-based accuracy:** accuracy assessment that treats objects or polygons as the primary units and may evaluate both label accuracy and boundary quality (Radoux & Bogaert, 2017).

**Overall accuracy (OA):** proportion of area (or units, depending on weighting) correctly classified.

**Producer’s accuracy (PA):** for a reference class, the proportion of reference units correctly mapped. Often interpreted as 1 minus omission error.

**Response design:** rules and data sources used to assign reference labels to sampled units (Stehman & Czaplewski, 1998).

**Sampling design:** rules used to select sample units, including stratification, clustering, and spatial balance (Stehman & Czaplewski, 1998).

**Spatial autocorrelation:** similarity of values near each other in space. It affects variance and motivates spatially balanced sampling (Stehman, 1992; Stevens & Olsen, 2004).

**Stratified random sampling:** divide the population into strata and sample within each. Common in land cover validation, often using map classes as strata (Olofsson et al., 2014).

**Systematic sampling:** sample units on a regular grid. It improves spatial coverage but complicates variance estimation and can align with periodic patterns (Cochran, 1977; Stehman, 1992).

**User’s accuracy (UA):** for a map class, the proportion of mapped units that are correct. Often interpreted as 1 minus commission error.

---

## 23. Appendix E: Common pitfalls and how to avoid them


This appendix lists failure modes that appear often in applied validation studies. The literature mentions many of these issues explicitly.

### 23.1 Convenience samples reported as if they were probability samples

Some studies select sites near roads or near field stations. They then compute accuracy metrics as if the sample represents the population. This can produce unknown bias because inclusion probabilities are not known. The Provided Literature Review warns against this and stresses probability-based selection (Provided Literature Review, p. 4).

### 23.2 Stratified sampling analyzed as simple random sampling

A common error occurs when teams stratify the sample and then compute overall accuracy as the raw proportion correct in the combined sample. This estimates a stratum-average parameter, not area-weighted accuracy. It also produces incorrect standard errors. Olofsson et al. (2014) provide correct weighted estimators and standard errors for stratified designs (Olofsson et al., 2014).

### 23.3 Cluster sampling analyzed as if points were independent

When teams validate many points in the same field site or tile, points correlate. Treating them as independent underestimates variance. Stehman (1997) provides methods for standard errors under cluster sampling (Stehman, 1997).

### 23.4 Reference label uncertainty ignored

Reference data can contain error. If analysts treat reference labels as perfect, they can overstate map accuracy and understate uncertainty. Foody (2013) and Foody (2024) show that reference error can strongly distort both accuracy and area estimates (Foody, 2013; Foody, 2024).

### 23.5 Unit-of-analysis mismatch

If the map meaning sits at object or patch scale, but validation uses single pixels, then the assessment may not match how users interpret the map. The Provided Literature Review recommends matching sampling and reference units to MMU and map meaning (Provided Literature Review, p. 5). Radoux and Bogaert (2017) make a similar point for object-based products (Radoux & Bogaert, 2017).

### 23.6 Reporting without uncertainty

Accuracy metrics without uncertainty can mislead. Many users need confidence intervals or standard errors for decision support. Olofsson et al. (2014) and Stehman and Foody (2019) both stress uncertainty reporting (Olofsson et al., 2014; Stehman & Foody, 2019).

---

## 24. Extended literature synthesis by theme

This section gives a more detailed narrative of the literature streams that shape current practice. It focuses on sources that tie directly to sampling for accuracy validation.

### 24.1 Foundational reviews and textbooks

Congalton (1991) remains a widely cited review of accuracy assessment in remote sensing. It connects sampling design to the validity of accuracy statements and helped standardize the confusion matrix as a central tool (Congalton, 1991).

Congalton and Green’s book *Assessing the Accuracy of Remotely Sensed Data: Principles and Practices* became a key reference for practitioners. It compiles guidance on sampling and accuracy metrics and is often used in teaching and applied reports (Congalton & Green, 1999).

Foody (2002) reviews the status of land cover classification accuracy assessment and discusses conceptual and practical issues that persisted after early standardization. This includes sampling issues and the meaning of different accuracy measures (Foody, 2002).

### 24.2 Sampling design research within thematic map validation

Stehman and Czaplewski (1998) formalize the triad and emphasize probability sampling and compatible estimators. Their work also clarifies the distinction between sampling design and response design, which helps explain why two studies with the same sample size can have very different validity (Stehman & Czaplewski, 1998).

Stehman (1999) provides a focused treatment of basic probability sampling designs for thematic map accuracy assessment, including simple random, stratified, systematic, and cluster designs. This paper plays a role similar to a methods chapter because it ties sampling design choices to estimator properties (Stehman, 1999).

Stehman (2000) discusses practical implications of design-based inference for thematic map accuracy assessment. It reinforces the idea that the map domain is a finite population, and it warns against importing variance formulas from models that do not match the design (Stehman, 2000).

Stehman (2009) reviews sampling designs for accuracy assessment of land cover. It synthesizes design options and discusses trade-offs, including sample size allocation and the effect of sampling on different accuracy metrics (Stehman, 2009).

### 24.3 Area estimation and the link between accuracy and quantity

Two streams connect strongly here. The first stream focuses on misclassification bias in areal estimates and calibration methods. Czaplewski (1992) and Czaplewski and Catts (1992) show how misclassification can distort area estimates and how calibration can correct this when reference data exist (Czaplewski, 1992; Czaplewski & Catts, 1992).

The second stream focuses on land-change estimation and best practices for reporting both accuracy and area uncertainty. Olofsson et al. (2013) and Olofsson et al. (2014) provide estimators for accuracy and area under stratified sampling. These works became core references for large-area change studies (Olofsson et al., 2013; Olofsson et al., 2014).

Stehman (2012) adds a key insight for change maps. Allocation strategies that improve change accuracy do not always improve change area estimation. This highlights the need to state the main objective before fixing sample allocation (Stehman, 2012).

### 24.4 Object-based mapping and polygon sampling

As GEOBIA and polygon products became common, validators faced new questions. Radoux and Bogaert (2014) focus on a specific but important issue: variable polygon size. They show how polygon area affects estimates of primary accuracy indices and propose accounting methods (Radoux & Bogaert, 2014).

Radoux and Bogaert (2017) broaden this into a good-practice discussion of object-based accuracy assessment. They emphasize matching the sampling unit and metrics to the product and the user’s goal. They also highlight boundary assessment as a separate dimension of quality (Radoux & Bogaert, 2017).

### 24.5 Reference data uncertainty and the “ground truth” problem

Foody (2013) shows that reference data error can distort estimates of change area. Foody (2024) argues that “ground truth” is often not available and that analysts should treat reference as a measurement with uncertainty. This literature stream pushes response design and uncertainty propagation to the center of validation design (Foody, 2013; Foody, 2024).

### 24.6 Spatially variable accuracy and accuracy mapping

Two cited works illustrate this theme. Smith et al. (2002) link accuracy to patch size and heterogeneity for NLCD classes, which shows that accuracy depends on landscape structure. Steele et al. (1998) propose estimating and mapping misclassification probabilities, which is one route to spatially explicit uncertainty surfaces (Smith et al., 2002; Steele et al., 1998).

---

## 25. Appendix F: Short R code sketches for spatial sampling

These sketches show common workflows. They assume familiarity with R spatial packages. They aim to show the sequence of steps, not to be copy-paste complete for every environment.

### 25.1 Spatially balanced point sampling with `spsurvey`

```r
# Conceptual example only.
# Install packages as needed: spsurvey, sf, terra

library(sf)
library(spsurvey)

# AOI polygon in sf format
# aoi <- st_read("aoi.gpkg")

# Design a spatially balanced sample of points inside AOI
# Using GRTS design through spsurvey

design <- list(None = list(panel = c(main = 200), seltype = "Equal"))

# For an area resource, spsurvey expects an sf object for the sampling frame.
# frame <- aoi

# sample <- grts(design = design, DesignID = "val", type.frame = "area",
#                src.frame = "sf.object", sf.object = frame)

# sample is a set of point locations with design information.
```

### 25.2 Stratified random sampling by map class

```r
# Conceptual idea: draw a spatially balanced sample inside each class stratum.
# Analysts can build class polygons or masks, then run grts per stratum.

# design <- list(class1 = list(panel = c(main = 50), seltype = "Equal"),
#                class2 = list(panel = c(main = 50), seltype = "Equal"))
# sample <- grts(design = design, DesignID = "val", type.frame = "area", ...)
```

### 25.3 Notes on analysis

Design-based analysis depends on storing design weights and strata labels with each sample. Olofsson et al. (2014) provide estimators for accuracy and area under stratified designs. Stehman (1997) provides standard error estimators for cluster designs. Analysts should not compute naive standard errors that assume independent and identically distributed samples when the design does not support that assumption (Olofsson et al., 2014; Stehman, 1997).

---
## References

This bibliography lists sources cited in the text. Where possible, I include a DOI. Some older sources do not have a DOI in common records.

- ASPRS. (2014). *Positional Accuracy Standards for Digital Geospatial Data*. American Society for Photogrammetry and Remote Sensing.
- Brus, D. J., de Gruijter, J. J., & Heuvelink, G. B. M. (2011). Sampling for validation of digital soil maps. *European Journal of Soil Science, 62*(3), 394-407. https://doi.org/10.1111/j.1365-2389.2011.01364.x
- Brus, D. J., & de Gruijter, J. J. (1997). Random sampling or geostatistical modelling? Choosing between design-based and model-based sampling strategies for soil (with discussion). *Geoderma, 80*, 1-59. https://doi.org/10.1016/S0016-7061(97)00070-6
- CEOS Land Product Validation Subgroup (LPV). (2025). *A review of the current validation practices in the remote sensing community* (validation practices protocol, version 1.0). Committee on Earth Observation Satellites.
- Cochran, W. G. (1977). *Sampling Techniques* (3rd ed.). Wiley.
- Cohen, J. (1960). A coefficient of agreement for nominal scales. *Educational and Psychological Measurement, 20*(1), 37-46. https://doi.org/10.1177/001316446002000104
- Congalton, R. G. (1991). A review of assessing the accuracy of classifications of remotely sensed data. *Remote Sensing of Environment, 37*(1), 35-46. https://doi.org/10.1016/0034-4257(91)90048-B
- Congalton, R. G., & Green, K. (1999). *Assessing the Accuracy of Remotely Sensed Data: Principles and Practices*. Lewis Publishers.
- Czaplewski, R. L. (1992). Misclassification bias in areal estimates. *Photogrammetric Engineering & Remote Sensing, 58*(2), 189-192.
- Czaplewski, R. L., & Catts, G. P. (1992). Calibration of remotely sensed proportion or area estimates for misclassification error. *Remote Sensing of Environment, 39*(1), 29-43. https://doi.org/10.1016/0034-4257(92)90138-A
- de Gruijter, J. J., Bierkens, M. F. P., Brus, D. J., & Knotters, M. (2006). *Sampling for Natural Resource Monitoring*. Springer. https://doi.org/10.1007/3-540-33161-1
- Dumelle, M., Kincaid, T., Olsen, A. R., & Weber, M. (2023). spsurvey: Spatial Sampling Design and Analysis. *Journal of Statistical Software, 105*(3). https://doi.org/10.18637/jss.v105.i03
- Foody, G. M. (2002). Status of land cover classification accuracy assessment. *Remote Sensing of Environment, 80*, 185-201. https://doi.org/10.1016/S0034-4257(01)00295-4
- Foody, G. M. (2013). Ground reference data error and the mis-estimation of the area of land cover change as a function of its abundance. *Remote Sensing Letters, 4*(8), 783-792. https://doi.org/10.1080/2150704X.2013.798708
- Foody, G. M. (2015). An assessment of citizen contributed ground reference data for land cover map accuracy assessment. *ISPRS Annals of the Photogrammetry, Remote Sensing and Spatial Information Sciences, II-3/W5*, 219-225. https://doi.org/10.5194/isprsannals-II-3-W5-219-2015
- Foody, G. M. (2024). Ground truth in classification accuracy assessment: Myth and reality. *Geomatics, 4*(1), 81-90. https://doi.org/10.3390/geomatics4010005
- Hay, A. M. (1979). Sampling designs to test land-use map accuracy. *Photogrammetric Engineering and Remote Sensing, 45*, 529-533.
- Hord, R. M., & Brooner, W. (1976). Land-use map accuracy criteria. *Photogrammetric Engineering and Remote Sensing, 42*(5), 671-677.
- ISO. (2023). *ISO 19157-1:2023 Geographic information - Data quality - Part 1: General requirements*. International Organization for Standardization.
- Olofsson, P., Foody, G. M., Stehman, S. V., & Woodcock, C. E. (2013). Making better use of accuracy data in land change studies: Estimating accuracy and area and quantifying uncertainty using stratified estimation. *Remote Sensing of Environment, 129*, 122-131. https://doi.org/10.1016/j.rse.2012.10.031
- Olofsson, P., Foody, G. M., Stehman, S. V., & Woodcock, C. E. (2014). Good practices for estimating area and assessing accuracy of land change. *Remote Sensing of Environment, 148*, 42-57. https://doi.org/10.1016/j.rse.2014.02.015
- Pontius, R. G., & Millones, M. (2011). Death to kappa: Birth of quantity disagreement and allocation disagreement for accuracy assessment. *International Journal of Remote Sensing, 32*(15), 4407-4429. https://doi.org/10.1080/01431161.2011.552923
- Radoux, J., & Bogaert, P. (2014). Accounting for the area of polygon sampling units for the prediction of primary accuracy assessment indices. *Remote Sensing of Environment, 142*, 9-19. https://doi.org/10.1016/j.rse.2013.10.030
- Radoux, J., & Bogaert, P. (2017). Good practices for object-based accuracy assessment. *Remote Sensing, 9*(7), 646. https://doi.org/10.3390/rs9070646
- Robertson, B. L., Brown, J. A., McDonald, T. L., & Jaksons, P. (2013). BAS: Balanced acceptance sampling of natural resources. *Biometrics, 69*(3), 776-784. https://doi.org/10.1111/biom.12059
- Särndal, C.-E., Swensson, B., & Wretman, J. (1992). *Model Assisted Survey Sampling*. Springer. https://doi.org/10.1007/978-1-4612-4378-0
- Smith, J. H., Stehman, S. V., Wickham, J. D., & Yang, L. (2002). Effects of landscape characteristics on land-cover class accuracy. *Remote Sensing of Environment*. https://doi.org/10.1016/S0034-4257(02)00126-8
- Stehman, S. V. (1992). Comparison of systematic and random sampling for estimating the accuracy of maps generated from remotely sensed data. *Photogrammetric Engineering and Remote Sensing, 58*(9), 1343-1350.
- Stehman, S. V. (1996). Estimating the kappa coefficient and its variance under stratified random sampling. *Photogrammetric Engineering and Remote Sensing, 62*, 401-407.
- Stehman, S. V. (1997). Selecting and interpreting measures of thematic classification accuracy. *Remote Sensing of Environment, 62*, 77-89. https://doi.org/10.1016/S0034-4257(97)00083-7
- Stehman, S. V. (1997). Estimating standard errors of accuracy assessment statistics under cluster sampling. *Remote Sensing of Environment, 60*, 258-269. https://doi.org/10.1016/S0034-4257(96)00176-9
- Stehman, S. V. (1999). Basic probability sampling designs for thematic map accuracy assessment. *International Journal of Remote Sensing, 20*(12), 2423-2441. https://doi.org/10.1080/014311699212100
- Stehman, S. V. (2000). Practical implications of design-based sampling inference for thematic map accuracy assessment. *Remote Sensing of Environment, 72*, 35-45. https://doi.org/10.1016/S0034-4257(99)00090-5
- Stehman, S. V. (2009). Sampling designs for accuracy assessment of land cover. *International Journal of Remote Sensing, 30*(20), 5243-5272. https://doi.org/10.1080/01431160903131000
- Stehman, S. V. (2012). Impact of sample size allocation when using stratified random sampling to estimate accuracy and area of land-cover change. *Remote Sensing Letters, 3*(2), 111-120. https://doi.org/10.1080/01431161.2010.541950
- Stehman, S. V. (2014). Estimating area and map accuracy for stratified random sampling when the strata are different from the map classes. *International Journal of Remote Sensing, 35*(13), 4923-4939. https://doi.org/10.1080/01431161.2014.930207
- Stehman, S. V., & Czaplewski, R. L. (1998). Design and analysis for thematic map accuracy assessment: Fundamental principles. *Remote Sensing of Environment, 64*(3), 331-344. https://doi.org/10.1016/S0034-4257(98)00010-8
- Stehman, S. V., & Foody, G. M. (2019). Key issues in rigorous accuracy assessment of land cover products. *Remote Sensing of Environment, 231*, 111199. https://doi.org/10.1016/j.rse.2019.05.018
- Stevens, D. L., Jr., & Olsen, A. R. (2004). Spatially balanced sampling of natural resources. *Journal of the American Statistical Association, 99*(465), 262-278. https://doi.org/10.1198/016214504000000250
- Steele, B. M., Winne, J. C., & Redmond, R. L. (1998). Estimation and mapping of misclassification probabilities for thematic land cover maps. *Remote Sensing of Environment, 66*(2), 192-202. https://doi.org/10.1016/S0034-4257(98)00061-3
- Story, M., & Congalton, R. G. (1986). Accuracy assessment: A user’s perspective. *Photogrammetric Engineering and Remote Sensing, 52*(3), 397-399.
- USGS. (2013). *Accuracy assessment of NLCD 2006 land cover and impervious surface* (summary page). U.S. Geological Survey Publications Warehouse.
- Van Genderen, J. L., & Lock, B. F. (1977). Testing land-use map accuracy. *Photogrammetric Engineering and Remote Sensing, 43*(9), 1135-1137.
- Wickham, J., Stehman, S. V., Gass, L., Dewitz, J., Fry, J., & Wade, T. G. (2013). Accuracy assessment of NLCD 2006 land cover and impervious surface. *Remote Sensing of Environment*. https://doi.org/10.1016/j.rse.2012.12.001
- Wulder, M. A., Franklin, S. E., White, J., Linke, J., & Magnussen, S. (2006). An accuracy assessment framework for large-area land cover classification products derived from medium-resolution satellite data. *International Journal of Remote Sensing, 27*(4), 663-683. https://doi.org/10.1080/01431160500185284

### Seed document

- Provided Literature Review. (n.d.). *Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review* (PDF provided by user).