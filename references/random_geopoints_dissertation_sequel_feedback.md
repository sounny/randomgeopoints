# Random GeoPoints and the Next Generation of Geospatial Validation Workflows
## A sequel focused on research gaps and software opportunities

**Purpose.** This document extends the prior dissertation-style synthesis by addressing a set of specific research gaps and tool opportunities that emerged during review. It does not re-state foundational material already covered elsewhere. It focuses on what the literature supports and on transparent design proposals for a client-side sampling and validation tool ("Random GeoPoints").

**Scope note.** Many sections propose tool features. These are design hypotheses, not proven results. When the literature does not provide a clear answer, the text states that limit and avoids guesswork.

## Abstract

Spatial sampling for thematic map accuracy assessment has mature statistical foundations, yet implementation practice still varies widely. Several gaps persist at the boundary between theory and the tools people use day to day. The first gap is access to spatially balanced probability samples such as GRTS and BAS, which often remain locked behind command-line workflows or specialist software (Stevens & Olsen, 2004; Robertson et al., 2013; Dumelle et al., 2023). The second gap is weak standardization of response design. Many tools generate coordinates but do not help the user define the labeling rules that drive the reference data, even though response design is a core component of rigorous accuracy assessment (Stehman & Czaplewski, 1998; Olofsson et al., 2014). Other gaps include poor support for rare class planning, limited spatial dependence diagnostics for realized samples, limited workflows for time-series and change validation, and limited support for reference label uncertainty (Foody, 2015; Stehman & Foody, 2019).

This sequel converts these gaps into a research and engineering blueprint. It outlines tool modules, data structures, diagnostic checks, and evaluation plans that align with design-based inference while improving transparency and usability. The goal is not to replace statistical guidance. The goal is to make that guidance easier to follow and easier to audit.

## Table of contents

1. [Accessibility gap for spatially balanced designs](#1-accessibility-gap-for-spatially-balanced-designs)
2. [Response design standardization gap](#2-response-design-standardization-gap)
3. [Rare class planning and visualization gap](#3-rare-class-planning-and-visualization-gap)
4. [Post-sampling spatial dependence checks](#4-post-sampling-spatial-dependence-checks)
5. [Time-series validation gap](#5-time-series-validation-gap)
6. [Reference error and uncertainty gap](#6-reference-error-and-uncertainty-gap)
7. [Integration blueprint for the Random GeoPoints tool](#7-integration-blueprint-for-the-random-geopoints-tool)
8. [Conclusions and research agenda](#8-conclusions-and-research-agenda)
9. [References](#9-references)

## 1. Accessibility gap for spatially balanced designs

### 1.1 The gap, stated plainly

The literature repeatedly argues that a well spread probability sample can outperform a simple random sample when the target variable shows spatial structure. In environmental and land cover work, spatial structure is common. Errors and classes tend to cluster. Spatially balanced designs aim to reduce this risk by improving spatial coverage while retaining a probability basis for inference (Stevens & Olsen, 2004). BAS and GRTS have become widely cited examples (Stevens & Olsen, 2004; Robertson et al., 2013).

The gap is not theoretical. It is practical. Many users can generate random points or grids in a GIS. Far fewer users can generate GRTS or BAS samples without installing R, learning packages, and adopting a scripted workflow. Even when packages exist, they may feel inaccessible to teams that work in browsers, mobile apps, or thin-client environments.

This gap matters for tool design because it creates a selection bias funnel. When a method feels hard, teams avoid it. They fall back to simple random points or fixed grids. That choice can raise variance, reduce coverage, and increase the chance that the realized sample misses key parts of the map.

A client-side tool can reduce that gap. It can turn a method that now feels like a specialist trick into a default option. Theobald et al. (2007) already framed the value of GIS support for spatially balanced designs. A browser-first implementation extends that idea to modern deployment contexts.

### 1.2 What counts as “accessible” in a sampling tool

Accessibility in this context has four parts:

- **No server requirement.** Many agencies and labs cannot upload study area geometry or sensitive maps to a third-party server. A client-side design keeps data local.
- **A simple user story.** A user imports a polygon, sets a sample size, and exports points. The tool should not require advanced statistical setup.
- **A reproducible seed.** The tool should store and export a seed, a design label, and all design settings. This supports transparency and replication (Stehman & Czaplewski, 1998).
- **A credible link to established algorithms.** The tool should not invent a new method without evidence. It should implement published algorithms or clearly label approximations.

The final point is crucial. A browser tool can feel smooth and still be wrong. Statistical rigor needs more than a friendly interface.

### 1.3 BAS as a strong candidate for client-side implementation

BAS has a property that matters for software. It is conceptually simple and computationally light relative to some alternatives. Robertson et al. (2013) describe BAS as a spatially balanced design that uses the Halton sequence to spread sample sites in space while supporting targeted inclusion probabilities through acceptance sampling. Later work discusses BAS for master samples used over time (van Dam-Bates et al., 2018) and package implementations support BAS workflows (e.g., the CRAN package **spbal**, which provides BAS draws for areal resources).

BAS also fits a browser workflow because it can generate an ordered “master list” of candidate points. A user can take the first *n* points for a sample, then take the next *m* points for replacements or later panels. This aligns with the master sample idea used in monitoring programs, where a single ordered frame supports staged selection over time (van Dam-Bates et al., 2018).

GRTS also has broad support and a long track record (Stevens & Olsen, 2004). The R package **spsurvey** implements GRTS and supports features such as stratification, unequal inclusion probabilities, legacy sites, minimum distances, and replacement options (Dumelle et al., 2023). However, GRTS has more moving parts and may be harder to port without careful work. A practical “accessibility first” roadmap can begin with BAS and later extend to a fuller GRTS port.

### 1.4 A BAS-Lite algorithm for polygons in the browser

This section describes a conservative implementation target. It focuses on equal-probability BAS for an areal region. It avoids unequal probability features unless the tool can support them with care.

**Core idea.** Generate a low-discrepancy sequence of candidate points in a bounding box. Keep only the points that fall inside the study polygon. Take the first *n* accepted points as the sample. Because the candidate sequence spreads points evenly in the unit square, the accepted points tend to spread evenly in the study region as well, even when the region has an irregular shape (Robertson et al., 2013).

A browser implementation can use these steps:

1. **Project coordinates to a planar system.** A uniform design in longitude and latitude does not map to uniform area. The tool should re-project to a local equal-area or local projected CRS when feasible. If the user cannot provide a CRS, the tool can warn and continue with a best-effort option.
2. **Compute a polygon bounding box.** Use that box to map Halton points from the unit square to the study extent.
3. **Generate Halton points.** Use bases 2 and 3 for 2D, as commonly used for Halton sequences. Randomize the sequence start if the tool aims to mimic randomized quasi-random sampling, as discussed in the broader quasi-Monte Carlo literature. For BAS, Robertson et al. (2013) emphasize the Halton sequence as the spatial spread engine.
4. **Point-in-polygon test.** Accept a candidate point if it falls inside the polygon. Reject it otherwise. Repeat until the tool collects *n* accepted points.
5. **Optional minimum distance.** If the user needs a minimum distance between sites, apply a fast distance filter. The tool should report when this constraint forces many rejections. The tool should also warn that a strong minimum distance constraint changes inclusion properties.

This is not yet the full BAS described in Robertson et al. (2013) because it does not implement targeted inclusion probabilities via acceptance sampling. It is a “Lite” version that prioritizes a well spread equal-probability sample.

### 1.5 Measuring spatial balance inside the tool

A GUI tool needs more than a sample generator. It needs diagnostics that help the user trust the result. The **spsurvey** ecosystem recognizes this need by providing ways to measure spatial balance and to visualize designs (Dumelle et al., 2023). A browser tool should do the same.

At minimum, a browser tool can report:

- **Nearest Neighbor Index (NNI).** The tool can compare the observed nearest-neighbor distance to the expectation under complete spatial randomness. NNI gives a quick check for clustering or strong regularity.
- **Coverage summaries.** The tool can compute a grid overlay and count points per cell. This helps a user see gaps.
- **Master sample order.** The tool can show that early points cover the region. This supports staged use.

These diagnostics do not replace formal proofs. They help users detect obvious problems early.

### 1.6 Reproducibility and cross-validation with established software

A browser port must earn trust. It needs a verification plan.

A practical plan includes:

- **Design equivalence tests.** Use identical inputs in R and in the browser tool, then compare summary diagnostics. For GRTS, exact point matches may not be possible without a full matching implementation, but spatial balance metrics and inclusion checks can still align.
- **Unit tests for geometry.** Point-in-polygon logic and projection steps can fail at boundaries. Tests should include holes, multi-polygons, and very thin polygons.
- **Seed tests.** A fixed seed should yield identical output across runs and across browsers.

Dumelle et al. (2023) provide a modern reference for a widely used implementation and for the set of design features that users may expect from mature sampling software.

### 1.7 Open research and engineering questions

A browser tool can implement BAS-Lite without open research. Other features require careful work and, in some cases, new research:

- **Unequal probability BAS in polygons.** Robertson et al. (2013) describe targeted inclusion probabilities via acceptance sampling. A browser tool needs a clear, auditable implementation for this case.
- **Stratified spatial balance.** Many accuracy assessments need stratification. The tool needs to define what “spatial balance within strata” means in practice and how to handle tiny strata.
- **Replacement sites and legacy sites.** Field teams often need backup points and want to keep past sites. The spsurvey design supports these features for GRTS (Dumelle et al., 2023). A browser tool needs analogous, transparent logic.

These questions define a realistic roadmap. They also define publishable work. A tool can become a research platform when it exposes these decisions clearly.
### 1.8 GRTS versus BAS in a tool context

A tool designer needs to understand why users ask for “GRTS” or “BAS.” Users often want three outcomes at once:

- a probability sample that supports design-based inference,
- better spatial coverage than simple random points, and
- operational flexibility, such as replacement sites or staged sampling over time.

GRTS and BAS both target these outcomes, but they arrive there in different ways.

Stevens and Olsen (2004) introduced GRTS as a probability design that achieves spatial balance through a tessellation and a hierarchical randomization strategy. The method creates an ordered list of spatial addresses. That list supports selection of a well spread sample and supports staged use.

Robertson et al. (2013) present BAS as a spatially balanced design that uses the Halton sequence to create an ordered list of candidate sites, coupled with acceptance sampling when targeted inclusion probabilities are needed. BAS also supports a master sample logic that fits monitoring programs and staged sampling (van Dam-Bates, n.d.; Robertson et al., 2013).

From a tool standpoint, these differences imply a choice. If the tool aims for a near-term client-side implementation, BAS may be the first target because it is simpler to compute and easier to explain. If the tool aims to match the features users expect from mature monitoring software, a GRTS path is also valuable because it has long use in environmental monitoring and a strong software ecosystem (Stevens & Olsen, 2004; Dumelle et al., 2023).

The most important design decision is not which acronym to include in the menu. The decision is how the tool will explain assumptions, and how it will export the metadata needed for design-consistent analysis.

### 1.9 Spatial balance is not a single metric

“Spatially balanced” can mean several things in practice. A sample can spread points evenly in space and still miss rare strata. A sample can cover strata evenly and still cluster within a stratum if the stratum has a complex shape. A tool should therefore avoid a single “balance score.”

A practical approach is to report several complementary summaries:

- a nearest-neighbor summary (local spacing),
- a grid coverage summary (gap detection), and
- a stratum coverage summary (representation).

These summaries align with the idea that spatial balance is a design objective. The objective needs operational checks, not one magic number.

### 1.10 Stratification plus spatial balance: a safe, explicit approach

In accuracy assessment, stratification often happens by mapped class, change class, or reporting region (Olofsson et al., 2014; Tyukavina et al., 2025). A tool that offers spatial balance should also support stratification, because users rarely work with a single homogeneous region.

A conservative approach is “within-stratum spatial balance.” The tool draws a spatially balanced sample inside each stratum independently. The tool then merges the strata samples and exports stratum weights. This matches a common design logic and stays close to established stratified estimation frameworks (Olofsson et al., 2013; Olofsson et al., 2014).

This approach also makes failure modes visible. If a stratum is tiny or fragmented, the tool can show that the stratum cannot support the requested sample size without strong clustering or boundary crowding.

### 1.11 Performance constraints in browser implementations

Browser tools face hard constraints. They must keep memory use low. They must avoid long blocking computations. These constraints matter most for three operations:

- point-in-polygon checks for large multi-polygons,
- minimum distance enforcement, and
- stratified designs with many strata or very large candidate pools.

A tool can address these limits through engineering choices that do not change the statistics:

- Use spatial indexing to speed up point-in-polygon checks when polygons are complex.
- Use tiling or chunking for candidate point generation so the UI stays responsive.
- Put clear limits on minimum distance constraints and explain when the constraint changes the design properties.

These are not statistical innovations. They are the steps that make rigorous designs usable on real devices.

### 1.12 A test matrix that matches real user risk

Users trust tools that fail loudly when inputs are unsafe. A sampling tool should therefore test inputs and report risks. A practical test matrix includes:

- CRS risk: are coordinates in degrees, and does the region span large lat ranges?
- Geometry risk: are there self-intersections, holes, or tiny slivers?
- Strata risk: are there strata with near-zero area or thousands of fragments?
- Constraint risk: does the user’s exclusion mask remove most of the region?

When a test fails, the tool can still allow export. It should also export the warnings. This supports honest reporting and aligns with calls for transparency in accuracy assessment reporting (Olofsson et al., 2014).


## 2. Response design standardization gap

### 2.1 Why response design belongs inside the tool

Sampling design decides *where* to check. Response design decides *how* to assign a reference label. Stehman and Czaplewski (1998) made this separation explicit. They describe accuracy assessment as three linked parts: sampling design, response design, and analysis. Olofsson et al. (2014) also treat response design as a major component and recommend that the reference classification be considerably more accurate than the map classification.

Most point generators stop after they export coordinates. They assume the user will handle labeling in a separate workflow. That split creates a failure mode. Teams may follow a sound sampling plan and still produce weak accuracy estimates because the labeling rules change over time, vary by analyst, or remain undocumented.

A tool can address this by treating response design as a first-class artifact. It can store it, export it, and require it before final export. This does not force one “correct” protocol. It forces clarity and audit.

### 2.2 A response design has structure that software can capture

A response design includes choices that many projects leave implicit:

- **Sample unit support.** Does the label apply to a pixel, a block of pixels, a polygon, or a plot? Stehman and Wickham (2011) discuss how the choice of spatial unit affects thematic accuracy assessment, and why pixels, blocks, and polygons can lead to different results.
- **Minimum mapping unit (MMU) and mixed units.** What does the analyst do when a sample unit contains more than one class? Many projects rely on an informal “dominant class” rule or a threshold, but the project rarely records that rule.
- **Edge and boundary rules.** If a point falls near a boundary, does the analyst buffer it? Does the analyst treat boundary cases as “mixed” or as one side?
- **Temporal alignment.** Which date or season counts as the reference moment? This matters for change maps and for any dynamic land cover product (Olofsson et al., 2014).
- **Confidence and evidence.** What data sources did the analyst use? How confident is the label?

Each item can be written down. A tool can encode it in a compact schema. That schema can drive UI prompts and export formats.

### 2.3 From “point generator” to “protocol builder”

A protocol builder module can produce two outputs at once:

1. **A machine-readable protocol.** A JSON file that specifies response rules. This supports reproducibility. It also supports later automation.
2. **A human-readable field sheet.** A PDF or markdown report that states the protocol in plain language and gives analysts a checklist.

The module can guide the user through a set of structured choices. It can also provide safe defaults, with warnings when defaults do not match the study.

A minimal protocol schema can include these fields:

- Project metadata: map name, map date, CRS, version, analyst team.
- Sample unit definition: point, pixel, buffered point, block, polygon, plot.
- MMU and mixed rules: threshold for dominance, how to label ties, how to flag mixed.
- Evidence sources: field visit, high-res image, time series, local data, other.
- Confidence scale: for example 1–5 or low/med/high, with definitions.
- Dispute handling: what to do when two analysts disagree.

This concept aligns with good practice calls for transparency and for documenting deviations from guidelines (Olofsson et al., 2014).

### 2.4 Response design as an active decision tree

Radoux et al. (2020) review response designs and stress that response design choices affect accuracy estimates. Software can represent these choices as a decision tree:

- Determine if the sample unit is homogeneous at the chosen support.
- If not homogeneous, apply the MMU rule or mixed rule.
- If boundary uncertainty remains, assign a mixed label or a “cannot decide” label with a defined code.

A decision tree reduces drift across analysts. It also supports training.

### 2.5 Quality control needs to be part of the workflow

Reference labeling is a measurement process. Measurement needs quality control. Pengra et al. (2020) present a quality control and interpreter consistency workflow for global land cover reference data and show why consistency checks matter.

A tool can support QC in a lightweight way:

- **Duplicate labels.** The tool can assign a subset of points to two analysts. It can track agreement.
- **Blind re-check.** The tool can re-insert a point later to check drift over time.
- **Audit log.** The tool can store who labeled a point, when, and with what evidence sources.

The tool should not hide disagreement. It should surface it. Reference error is a real part of the system, not a rare glitch (Foody, 2015; Stehman & Foody, 2019).

### 2.6 Field sheet generation as a practical standard

A protocol PDF can follow a stable structure so teams can cite it in reports. It can include:

- A one-page protocol summary.
- A table of class codes and definitions.
- The MMU and mixed rules in one place.
- A short “edge case” section with examples.
- A label confidence rubric.

This is a small feature, but it closes a big gap. It turns response design into something teams can share, reuse, and review.

### 2.7 Research questions that follow from protocol formalization

Once a tool formalizes response design, it also creates a dataset about response design. That opens research questions:

- Which response rules drive the largest shifts in reported accuracy?
- How much agreement do analysts achieve under different protocols?
- How does sample unit support interact with class fragmentation?

These are not abstract questions. They shape how users should interpret accuracy statements. They also shape how tools should report uncertainty.
### 2.8 Protocol versioning and change control

A response design is not a one-time choice. Teams evolve definitions across time. They refine class legends. They revise MMU rules after field reality hits. If a tool does not track these changes, later analysis can become impossible to interpret.

A protocol builder should therefore support explicit versioning:

- a protocol ID,
- a version number,
- a change log entry for each update, and
- a link from each labeled point to the protocol version used for its label.

This feature is simple, but it prevents a common failure mode. Without protocol versioning, two analysts may label points under different rules and the study will not know it.

### 2.9 Response design for area and cluster sampling units

Many field validation programs use cluster or area sampling for cost reasons. The literature review describes cluster sampling as a common strategy when field travel dominates cost, and notes the need for cluster-adjusted analysis (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

A protocol builder can support cluster or area units by adding fields that define how the cluster is labeled. Examples include:

- how many sub-points inside a plot the analyst checks,
- how the analyst aggregates sub-point labels to a plot label, and
- whether the analyst records within-plot heterogeneity.

Stehman and Wickham (2011) show why the choice of pixel, block, or polygon support affects the meaning of “correct.” A tool should not assume a point label equals a plot label. It should store the support definition.

### 2.10 Interoperability with labeling platforms

Teams label points in many places. Some use a spreadsheet. Some use a mobile app. Some use a web-based photo interpretation system.

A protocol artifact can improve interoperability by providing a shared data dictionary. The tool can export:

- a table schema for labels,
- allowed class codes and text labels,
- valid confidence levels, and
- valid evidence source codes.

This supports consistent data capture across platforms. It also reduces manual cleaning.

### 2.11 Protocol-driven user interfaces reduce error by design

A protocol builder can do more than export a PDF. It can also drive UI constraints. For example:

- If the protocol requires a confidence score, the UI can block submission without it.
- If the protocol requires an evidence source, the UI can require at least one evidence code.
- If the protocol defines a mixed label code, the UI can offer it explicitly and require a short note.

This is a key shift. The tool does not rely on analyst memory. The tool enforces the protocol in the same way a data entry form enforces field types.

Radoux et al. (2020) argue that response design choices shape accuracy results. Protocol-driven UI makes those choices explicit and stable.


## 3. Rare class planning and visualization gap

### 3.1 The planning problem that many teams face

Stratified sampling supports rare classes because it guarantees some samples in each class (Congalton, 1991; Olofsson et al., 2014). Yet stratification creates a planning trade-off. If a team allocates samples in proportion to class area, rare classes may receive too few samples to estimate class-specific accuracy with useful precision. If a team allocates equal samples per class, the team improves per-class estimates but must apply weights during analysis to avoid biased overall estimates (Olofsson et al., 2013; Olofsson et al., 2014).

Many users handle this trade-off by guessing. They pick a total sample size or a per-class count based on habit. The literature contains guidance and rules of thumb, but most point tools do not help users connect those rules to their own strata map and to explicit uncertainty targets.

### 3.2 What the literature gives us, and what it does not

The literature offers several anchors:

- A long-standing rule of thumb aims for roughly 50 samples per class in a stratified design, though context matters (Congalton, 1991).
- Olofsson et al. (2014) stress that the sampling design should match the priority objectives of accuracy and area estimation, and that analysts should report confidence intervals for accuracy and area estimates.
- Foody (2009) reviews statistical principles for sample size determination in classification accuracy assessment and shows why both too small and too large samples can harm a study.

What the literature does not give is a universal one-click sample size. The right sample size depends on:

- the metric of interest (overall accuracy, user’s accuracy, producer’s accuracy, area estimates),
- the required precision (margin of error),
- the expected accuracy level,
- cost and logistics, and
- whether the design is stratified, clustered, or involves panels over time (Olofsson et al., 2014).

This gap is not a missing formula. It is a missing bridge between formulas and user decisions.

### 3.3 A tool can add a “power” and precision planning module

A planning module can help users make the trade-off visible before they generate the final sample.

At its simplest, the module can treat a class accuracy estimate as a binomial proportion. Under that approximation, an approximate sample size for a desired half-width of a confidence interval is:

$$n \approx \frac{z^2 \; p(1-p)}{E^2}$$

where *p* is the expected accuracy (for example, expected user’s accuracy for a class), *E* is the desired margin of error, and *z* is a normal critical value for the chosen confidence level (Foody, 2009).

This approximation has limits. It assumes independent trials and does not account for complex designs. Still, it gives users a first-order sense of how fast uncertainty shrinks as *n* grows.

A tool can then show a plot or table for each class:

- sample size (x-axis),
- expected margin of error (y-axis),
- and optional cost weights.

This makes “diminishing returns” visible. The user sees that the first 30 points can cut uncertainty sharply, while an increase from 300 to 330 points may matter far less.

### 3.4 Linking the planner to stratified estimation

Olofsson et al. (2013) and Olofsson et al. (2014) provide a design-based framework for stratified estimation of accuracy and area with uncertainty. A tool that supports stratified sampling should also support the core reporting objects those papers recommend, such as an estimated error matrix expressed as area proportions, and confidence intervals for key parameters.

A planning module can connect to this framework in two ways:

1. **Use the strata map to compute area weights.** If the user uploads a classified map or a strata layer, the tool can compute the proportion of area in each stratum. This supports weighted summaries.
2. **Simulate allocations.** Given a fixed total sample size, the tool can simulate alternative allocations, such as proportional, equal, or hybrid, and show expected precision outcomes for each class.

This does not require the tool to “decide” for the user. It equips the user to justify a decision.

### 3.5 A rare class is not just small. It is often hard to label.

Rare classes often align with edge cases. They may be small patches, narrow strips, or transient land cover. These conditions raise response design uncertainty. The planning module should therefore connect to the protocol builder. If the protocol uses a mixed rule or a strict MMU, the effective prevalence of the rare class may change.

This link highlights a broader point: sampling design, response design, and analysis form a coupled system (Stehman & Czaplewski, 1998; Olofsson et al., 2014). Tool modules should reflect that coupling.

### 3.6 Practical outputs that improve methods sections

A planning module can generate text that users can paste into a methods section. It can state:

- the target precision goals per class,
- the chosen allocation rule,
- the expected confidence interval widths under those assumptions, and
- any constraints that forced deviations (for example, field access).

Olofsson et al. (2014) explicitly recommend documenting deviations from good practice that may affect results. A tool can make that documentation easier.

### 3.7 Research questions for rare class design

The planning module also defines research questions that a tool can help answer:

- When a class is rare, which objective should dominate: class accuracy or total area accuracy?
- Which allocation rules yield the best stability under common land cover class distributions?
- How do spatially balanced designs interact with rare class oversampling?

These questions matter for tool defaults. They also matter for standards and guidance.
### 3.8 Allocation strategies deserve explicit comparison

A tool should expose common allocation strategies with clear definitions.

- **Proportional allocation.** Allocate samples in proportion to stratum area. This often supports precise overall estimates but can starve rare strata.
- **Equal allocation.** Allocate the same number of samples to each stratum. This supports per-stratum accuracy estimates but requires careful weighting for overall accuracy and area estimation (Olofsson et al., 2013; Olofsson et al., 2014).
- **Hybrid allocation.** Set a minimum per stratum, then allocate remaining samples proportionally. This can prevent starvation while keeping some proportional logic.

Olofsson et al. (2014) emphasize matching design to study objectives. These allocation options map directly to objective choices. A planning tool can help the user state which objective dominates.

### 3.9 Planning should account for design effects when clustering is used

The binomial approximation in Section 3.3 assumes independent sample units. Cluster sampling breaks that assumption. Points within a cluster tend to share similar land cover and similar map errors. This reduces effective sample size.

The literature review notes this issue and describes intra-cluster correlation as a key weakness of cluster designs, alongside the need for cluster-adjusted variance estimation (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

A planning module can therefore include a simple “design effect” option. The user can specify an assumed design effect for cluster sampling, or the tool can suggest a default range with a warning. The planner can then report an effective sample size:

$$n_{\text{eff}} = \frac{n}{\text{DEFF}}$$

This is not a full design-based variance estimator. It is a planning aid. The tool should clearly label it as such.

### 3.10 Visualization design: make uncertainty readable

Power and precision plots often confuse non-specialists. A tool can improve readability by following a few rules:

- Keep axes and units consistent across classes so users can compare.
- Allow a “budget slider” so the user sees how the plan changes with total sample size.
- Highlight classes that fail a target precision threshold.
- Provide a short explanation of assumptions near each plot.

Foody (2009) stresses that sample size choice should match study aims. A visualization that ties aims to uncertainty supports that match.

### 3.11 A worked planning example with hypothetical numbers

Assume a map has four strata: A, B, C, and D. Stratum D covers 1% of area and is the class the user cares about most. The user wants a margin of error near ±0.10 for user’s accuracy of D.

If the user expects user’s accuracy for D near 0.80, then under the simple proportion approximation the required sample size in D is roughly:

$$n_D \approx \frac{1.96^2 \; 0.8(0.2)}{0.1^2} \approx 62$$

This rough result already tells the user something. If the user uses proportional allocation and draws 500 total samples, stratum D would get about 5 points. That will not meet the target. The user must oversample D, increase total sample size, or accept wider uncertainty.

A tool can compute this result instantly and show it as a trade-off curve. It can also show how the required *n* changes if expected accuracy differs. This is the sort of decision support that many users currently do by guesswork.


## 4. Post-sampling spatial dependence checks

### 4.1 Why a realized sample can still fail

A user can select a “random” design and still end up with a realized sample that clusters in a way that creates field problems or analytic concerns. Pure simple random sampling allows clumps by chance. Practical constraints can also induce clumps. For example, a user may enforce access buffers, minimum distances from roads, or exclusion zones. These filters can warp the design in ways the user does not see.

A tool can help by checking the realized sample pattern before the user commits time and budget to field work. This aligns with a broader theme in accuracy assessment: many errors enter before analysis begins.

### 4.2 Two distinct problems: point pattern and attribute dependence

Spatial dependence appears in two different places:

- **Dependence in the point pattern.** Are the sample points themselves clustered or overly regular?
- **Dependence in the response.** Are map errors or class labels spatially correlated?

The first problem is purely geometric. The second problem involves the variable of interest. Classical variance estimators often assume independent sample units. Spatial autocorrelation can violate this assumption, which can lead to underestimated uncertainty in some settings (Stehman, 2000).

A tool can address the first problem before data collection. It can address the second problem after labeling, when it has reference and map labels.

### 4.3 Geometry diagnostics for the realized sample

A practical browser tool can implement several established point pattern checks.

#### 4.3.1 Nearest Neighbor Index as a fast first check

NNI compares the mean observed nearest-neighbor distance to the expectation under complete spatial randomness. NNI offers an intuitive red flag. If NNI is far below 1, the sample clusters. If NNI is far above 1, it is very regular. NNI cannot diagnose all issues, but it is fast and easy to explain.

#### 4.3.2 Ripley’s K for scale-dependent clustering

NNI looks only at the closest neighbor. Ripley’s K function looks across distance scales. Ripley (1976) introduced second-order point process analysis tools that include K. A tool can compute an estimated K across a set of distances and compare it to simulations under complete spatial randomness.

This check matters because a sample can look fine at one scale and still cluster at another. For example, points may spread well across the whole region but cluster inside a few sub-regions.

#### 4.3.3 Grid and quadrat counts for simple coverage checks

A grid overlay can provide a simple coverage view. The tool can report the fraction of cells that contain at least one point. This is not a formal test, but it gives the user a quick sense of gaps.

### 4.4 Attribute diagnostics after labeling: Moran’s I and related tools

Once the tool has map labels and reference labels, it can compute an error indicator per point, such as 1 for correct and 0 for incorrect. It can then test whether these errors cluster.

Moran’s I is a standard spatial autocorrelation statistic (Moran, 1950). A tool can compute Moran’s I on the error indicator using a distance-based or k-nearest neighbor weight matrix.

A high positive Moran’s I would suggest clustered error. That result has two uses:

- It warns that classical variance estimates may understate uncertainty if the design and estimator assume independence.
- It guides field follow-up. A team may target the cluster region to diagnose a systematic cause, such as cloud contamination or a training data gap.

This diagnostic does not “fix” the problem. It makes the problem visible.

### 4.5 Linking diagnostics to design choices

A tool becomes more useful when it connects diagnostics to actionable options. For example:

- If the realized sample clusters, the tool can suggest a spatially balanced design such as BAS or GRTS (Stevens & Olsen, 2004; Robertson et al., 2013).
- If the sample looks too regular due to a grid, the tool can suggest stratified systematic unaligned sampling, which randomizes the point within each grid cell and reduces alignment risk (Olofsson et al., 2014).

The tool should also record the diagnostic results in the export report. This supports later transparency.

### 4.6 Limits and honest messaging

A diagnostic can mislead when a user treats it as a proof. The tool should avoid that trap.

- A sample can pass NNI and still have problems. A single metric is not enough.
- A high Moran’s I on errors does not automatically invalidate design-based inference. It does signal that uncertainty estimation deserves attention (Stehman, 2000).
- A spatially balanced sample can reduce risk of clumps. It does not guarantee independence of errors. Errors can still cluster due to map production issues.

The tool should treat diagnostics as early warnings, not as certificates of validity.

### 4.7 Research opportunities: spatial models and design-based rigor

Recent reviews note a frontier where design-based sampling meets spatial models for better uncertainty quantification under spatial dependence (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025; Stehman, 2000). One review points to variogram-based methods and spatial bootstrap approaches such as moving block bootstrap as candidate strategies for map accuracy uncertainty under spatial dependence. The review also notes an ongoing debate on how to combine the rigor of randomization-based inference with spatial modeling without losing unbiasedness.

A tool cannot settle this debate alone. It can support research by producing the data needed to test methods, and by making spatial dependence visible in routine workflows.
### 4.8 Implementation notes for Moran’s I and Ripley’s K in a browser

A browser tool can compute Moran’s I once it has an attribute value per site. The tool needs a weight matrix. Two common choices are:

- **Distance band weights.** Connect sites within a fixed distance.
- **k-nearest neighbor weights.** Connect each site to its k closest neighbors.

Moran (1950) defines the statistic in terms of these weights. The tool should report the weight rule used because results depend on it.

For Ripley’s K, the tool needs edge correction if it wants accurate inference near boundaries. Edge correction can be complex. A safe first implementation can focus on simulation envelopes that use the same boundary and the same estimator for both observed and simulated patterns. The tool should still warn users that boundary effects matter (Ripley, 1976).

### 4.9 Warning logic should connect to action

Diagnostics have value only when users know what to do next. A tool can map diagnostic outcomes to a small set of action prompts:

- If the pattern clusters strongly under NNI or K, suggest BAS or GRTS.
- If the pattern is very regular due to a grid, suggest unaligned systematic within cells.
- If constraint masks cause clusters, suggest a redesign of masks or a two-stage design where clusters become explicit sampling units.

The key is to avoid a flood of alerts. Users need a short set of clear prompts that connect to design options.

### 4.10 Store diagnostics as part of the design record

A realized sample has a history. The tool should store diagnostic outputs with the sample. This supports later reporting and avoids the problem where a team runs diagnostics once, then loses the results.

A minimal diagnostic record can include NNI, coverage summaries, and parameter settings for Moran’s I or K (such as distance thresholds and neighbor counts).


## 5. Time-series validation gap

### 5.1 Why time breaks many “static map” workflows

Many sampling tools assume a map is a snapshot. Modern land cover products are often updated each year or more often. The literature review used in this project frames this as a scaling challenge across space and time, and asks whether teams should validate each year separately or leverage samples across time (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

Change maps add another layer. They include categories such as loss, gain, or persistence. Olofsson et al. (2014) provide good practice guidance for change maps and emphasize probability sampling, response design, and analysis aligned with the design. The CEOS land cover and change protocol also focuses on accuracy and area estimation for land cover and change products and stresses consistent estimators and reporting (Tyukavina et al., 2025).

Yet many teams still run a one-off validation each year with no linkage. That approach loses the chance to track change at fixed locations. It also increases cost.

### 5.2 The core design choice: panel, refresh, or hybrid

A time-series validation program usually faces a choice:

- **Pure refresh design.** Draw a new probability sample each year. This supports year-specific inference and avoids dependence across years. It can be costly.
- **Pure panel design.** Revisit the same sample units each year. This supports change detection at fixed sites and can reduce cost. It introduces temporal dependence and may under-sample newly changed areas.
- **Hybrid rotating panel.** Keep a subset of sites fixed and refresh the rest. Many monitoring programs use this logic because it balances trend tracking and coverage of new conditions.

The sampling literature outside remote sensing has long discussed these ideas. In remote sensing and land cover validation, tool support for them remains limited.

A tool does not need to pick one design for the user. It can expose the trade-off and implement common patterns.

### 5.3 Master samples as the backbone for time-series workflows

A master sample is an ordered list of candidate sites from which the user can draw sub-samples over time. BAS supports this naturally because the Halton-driven ordering gives a well spread sequence. van Dam-Bates et al. (2018) discuss BAS as a master sample framework in an ecological monitoring context.

The **spsurvey** package also supports advanced survey design features and has been used for monitoring and repeated surveys, including options that support replacement sites and legacy sites (Dumelle et al., 2023). These features align with the needs of time-series work, even when the underlying map changes.

A browser tool can implement a master sample workflow as follows:

1. Generate a large ordered master list using BAS-Lite or another spatially balanced method.
2. Store the master list with stable IDs.
3. For each time step, select the first *n* eligible sites, where eligibility may reflect access filters or strata needs.
4. Use later segments of the master list for replacements or for sample augmentation.

This approach supports reproducibility and avoids ad hoc point selection each year.

### 5.4 A longitudinal sample feature for “augment, do not restart”

The feedback for this project suggests a feature that many teams need: import last year’s sample and augment it for the new year.

A robust “longitudinal sample” module can support three actions:

- **Keep.** Keep a fixed set of sites as a panel.
- **Drop.** Drop sites that became invalid due to access, land change that makes the support inconsistent with the study question, or missing data.
- **Add.** Add new sites using the same design family and seed logic, with clear accounting.

This module requires strong metadata. Each point should carry:

- a stable site ID,
- a generation rule (design label, seed, and parameters),
- a time stamp or map version tag, and
- a status field (kept, dropped, replaced, added).

This creates a lineage record. It makes later interpretation of trend results far easier.

### 5.5 Sampling for change: oversample change, weight in analysis

Change strata often contain rare categories. If the map shows only a small fraction of change, a proportional allocation will give few change samples. Good practice guidance for change maps stresses probability sampling and design-consistent estimators (Olofsson et al., 2014; Tyukavina et al., 2025). A tool can support this by allowing oversampling of change strata while exporting the weights needed for unbiased area and accuracy estimation.

In practice, a tool can implement this workflow:

1. User uploads a change map with strata such as stable forest, loss, gain, and other.
2. Tool computes stratum area proportions.
3. User chooses an allocation rule. The tool shows expected precision trade-offs, as described in Section 3.
4. Tool draws a spatially balanced sample within each stratum.
5. Tool exports points plus design weights and a clear estimator template.

The key is not the UI. The key is the explicit export of design information so the analysis remains design-consistent.

### 5.6 Temporal alignment and evidence sources

Time-series validation depends on temporal alignment. If the map represents a year, the reference process must define what date or season counts. Olofsson et al. (2014) explicitly note the need for sufficient temporal representation in the reference data for response design. The literature review also notes that high-frequency mapping can outpace field validation capacity and motivates use of new reference sources, such as frequent high-resolution imagery and crowdsourced data, with statistical care (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

A protocol builder can capture temporal rules. It can also capture evidence sources, such as which imagery archive the analyst used.

### 5.7 Research questions for time-series validation

Time-series support in a tool raises open questions that deserve explicit research:

- How should analysts treat temporal dependence when they reuse sites?
- How should analysts combine panel and refresh data in one estimator?
- How should analysts design samples that support inference at multiple spatial scales across time?

A tool can support this work by providing reproducible master samples, stable IDs, and exported design metadata.
### 5.8 Multi-stage designs for global or multi-scale products

The literature review flags a core challenge for global products. Analysts often want inference at multiple scales at once, such as global, continental, and national. The review notes that one approach is multi-stage stratified sampling that samples hierarchically across regions (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

A browser tool can support a simplified version of this logic without implementing full survey operations. It can allow the user to supply nested region layers. The tool can then allocate samples across levels, such as:

- allocate a base number per continent,
- then allocate within continent by country,
- then allocate within country by map strata.

This is not a substitute for a full survey sampling package. It is a workflow aid that can produce a design with explicit structure and exported weights.

### 5.9 Temporal correlation is not a reason to avoid panels

Panel designs introduce correlation across time because the same sites repeat. That correlation can complicate variance estimation for trend metrics. It can also improve sensitivity to change because the design controls for spatial heterogeneity.

A tool should therefore present panels as a trade-off, not as a flaw. It should export the information needed for users to apply methods that handle repeated measures, or at least to state the dependence as a limitation.

The literature review explicitly notes that a spatiotemporal panel introduces temporal correlation issues. This is a research frontier rather than a solved problem (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

### 5.10 A common pitfall: mixing map versions without a lineage model

Many teams validate each year but store results in separate spreadsheets with no shared IDs. Later, they cannot tell which site repeats and which site is new. They also cannot align protocol changes across years.

A lineage model prevents this. It is a data management feature, but it has scientific impact because it preserves the meaning of trend analysis.


## 6. Reference error and uncertainty gap

### 6.1 The uncomfortable truth: “ground truth” is not always true

Many accuracy assessments treat reference labels as error-free. The literature has challenged this assumption for years. Foody (2015) argues that imperfect reference data can bias accuracy and area estimates. Stehman and Foody (2019) list reference data quality as a key issue for rigorous accuracy assessment of land cover products. The literature review used in this project also highlights imperfect and evolving reference data as an open research question, and notes that mislabeled validation samples can bias accuracy estimates (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025).

Reference error can come from many sources:

- interpreter disagreement,
- limited image resolution,
- temporal mismatch between map and reference,
- GPS and geolocation error, and
- mixed land cover within the sample unit.

These sources connect directly to response design, which is why Sections 2 and 6 should be read together.

### 6.2 Good practice already asks for more honesty

Olofsson et al. (2014) include evaluation of variability and potential error in the reference classification among their good practice recommendations. They also recommend documenting deviations from good practice that may affect results. These recommendations imply that reference uncertainty should not remain hidden.

The CEOS protocol for land cover and change maps also emphasizes careful reporting and consistent estimation, which depends on credible reference data (Tyukavina et al., 2025).

### 6.3 Tool-level support: record uncertainty, do not bury it

A tool can help by making reference uncertainty explicit in the data model.

A practical approach is to store, for each sample unit:

- the chosen reference label,
- a confidence score with a defined rubric,
- the evidence sources used, and
- optional alternative labels with notes.

This does not “solve” reference error, but it captures information needed for later analysis.

### 6.4 Multi-annotator workflows as a built-in option

Pengra et al. (2020) provide an example of quality control and interpreter consistency assessment for reference data. That workflow fits naturally inside a tool that manages a sample and a protocol.

A tool can implement multi-annotator support without heavy machinery:

- Assign 10–20% of points to two analysts.
- Compute agreement rates by class.
- Flag classes with high disagreement for protocol review or extra evidence.

The tool should export these QC results. A report that lists agreement by class can strengthen the credibility of the study.

### 6.5 Analysis options when reference data have error

The research community has not converged on a single standard method to adjust accuracy estimates for reference error in all settings. The tool should not pretend that a single slider can fix the problem.

Still, a tool can support several transparent options:

1. **Sensitivity analysis.** The user specifies plausible bounds on reference label accuracy. The tool re-computes accuracy metrics under these bounds and reports the range. This provides an honest view of how much results depend on reference reliability.
2. **Uncertainty propagation by Monte Carlo.** If each label has an associated probability distribution over classes (from confidence scores or from a confusion model of interpreter error), the tool can simulate many possible reference datasets and compute a distribution of accuracy metrics. This yields wider, more realistic intervals, but it depends on assumptions that the tool must state clearly.
3. **Report reference uncertainty as a separate result.** Even if the user does not adjust the final metrics, the tool can report reference disagreement rates and confidence summaries. This aligns with Olofsson et al. (2014) guidance to evaluate variability in the reference classification.

A key design rule is transparency. The tool should show assumptions, not hide them.

### 6.6 A “reference reliability” parameter needs guardrails

The feedback suggests a feature where a user inputs a reference reliability level, such as “I am 90% sure of labels,” and the tool widens confidence intervals. This idea can be useful as a sensitivity tool. It can also mislead if users treat it as a measured fact.

A responsible implementation can include guardrails:

- Require the user to justify the reliability value in a short note.
- Offer a default range rather than a single value, such as 0.85–0.95.
- Encourage a duplicate-label QC step to estimate reliability empirically, when feasible.

Stehman and Foody (2019) stress that reference data quality is a key issue. The tool should therefore treat this input as a serious parameter, not as a cosmetic setting.

### 6.7 Research agenda: reference error as part of the estimator, not a footnote

Reference uncertainty links to larger methodological questions:

- How should analysts combine reference uncertainty with design-based estimators for area and accuracy?
- When does reference error dominate map error, and how can teams detect that case early?
- Which QC strategies reduce bias most per unit cost?

These questions create space for new contributions. A tool that captures uncertainty at the point of labeling can accelerate that research.
### 6.8 Distinguish label uncertainty from location uncertainty

Reference error is not only a label problem. It can also be a location problem. A field team may visit a point but record the wrong location due to GPS error. A photo interpreter may label a point that falls on a boundary because of georegistration error.

Stehman and Wickham (2011) emphasize that the spatial unit matters. When the unit changes, the meaning of correctness changes. A tool can support this by allowing users to buffer points or to define blocks or plots as the sample unit. The protocol artifact should record this choice.

### 6.9 Reporting templates that make reference uncertainty visible

A sampling tool can help teams report reference uncertainty in a consistent way. A report template can include:

- number and fraction of duplicate-labeled sites,
- agreement rates overall and by class,
- distribution of confidence scores, and
- a short narrative that states how the team handled disagreements.

Olofsson et al. (2014) recommend evaluating variability and potential error in the reference classification. A template operationalizes that recommendation.

### 6.10 Ethics and governance: accuracy claims have real effects

Accuracy statements support policy, resource allocation, and sometimes legal claims. Overstated accuracy can cause harm. Understated accuracy can also cause harm by undermining trust in useful maps.

A tool should therefore encourage conservative reporting. It should keep uncertainty visible. It should export confidence intervals and QC summaries by default (Olofsson et al., 2014; Stehman & Foody, 2019).


## 7. Integration blueprint for the Random GeoPoints tool

### 7.1 Design goals derived from the gaps

The six gaps discussed above imply a set of tool goals:

- Make spatially balanced probability designs usable without a command line.
- Treat response design as a required part of the workflow (Stehman & Czaplewski, 1998; Olofsson et al., 2014).
- Help users plan sample sizes for rare classes with explicit uncertainty trade-offs (Foody, 2009; Olofsson et al., 2014).
- Diagnose realized samples for clustering and dependence before field work (Moran, 1950; Ripley, 1976).
- Support time-series workflows through master samples and augmentation (van Dam-Bates et al., 2018).
- Make reference uncertainty visible and analyzable (Foody, 2015; Stehman & Foody, 2019).

These goals shift the tool from a “coordinate exporter” to a “workflow steward.”

### 7.2 A modular architecture that maps to the research agenda

A browser-first tool can implement these modules as separate layers. The point is not software purity. The point is audit and testability.

#### 7.2.1 Input layer

Inputs should include:

- study area geometry (polygon or multi-polygon),
- optional strata layer (raster or vector),
- optional prior sample for longitudinal workflows,
- optional exclusion or constraint layers (buffers, masks).

The tool should validate geometry early and warn on common issues, such as self-intersections or mixed CRS.

#### 7.2.2 Design layer

The design layer implements sampling methods. A staged roadmap can look like this:

- **Phase 1.** BAS-Lite for polygons, plus stratified random and stratified systematic unaligned sampling.
- **Phase 2.** Unequal probability BAS features, if the tool can implement acceptance sampling in a way that is transparent and testable (Robertson et al., 2013).
- **Phase 3.** A full GRTS port or a hybrid that calls a local R backend in environments where users can run it. The GRTS reference implementation in **spsurvey** defines expected features such as legacy sites and minimum distance handling (Dumelle et al., 2023).

The tool should always export a design description that includes the algorithm name, parameter values, and seed.

#### 7.2.3 Protocol layer

The protocol layer implements the response design builder described in Section 2. This layer should produce a protocol artifact that users can cite, share, and archive.

#### 7.2.4 Planning layer

The planning layer implements rare class precision tools described in Section 3. It should read strata areas and show sample size versus expected uncertainty under simple assumptions (Foody, 2009). It should also warn when the user selects a design that violates those assumptions, such as heavy clustering.

#### 7.2.5 Diagnostics layer

The diagnostics layer should run pre-field checks and post-label checks:

- pre-field: NNI, coverage metrics, and Ripley’s K checks for clustering (Ripley, 1976),
- post-label: Moran’s I on errors or on class indicators (Moran, 1950),
- QC summaries: agreement rates for duplicate labels (Pengra et al., 2020).

Diagnostics should never replace the design description. They add context.

#### 7.2.6 Time-series layer

The time-series layer should manage master samples and longitudinal augmentation. It should store stable IDs and a lineage model described in Section 5.

#### 7.2.7 Export and reporting layer

Exports should include:

- points with stable IDs,
- design metadata (seed, algorithm, strata weights),
- the protocol artifact,
- a diagnostics report, and
- optional analysis templates that match the design, consistent with good practice guidance (Olofsson et al., 2014; Tyukavina et al., 2025).

The export report should also include a short “limitations” section, which lists any assumptions and any deviations from good practice.

### 7.3 Data structures for reproducible workflows

A sampling tool needs a durable data model. A minimal model for each sample unit can include:

- `site_id`: stable identifier.
- `geometry`: point or polygon.
- `stratum_id`: optional.
- `design_weight`: optional but recommended for stratified designs.
- `design_info`: algorithm name, parameters, seed.
- `protocol_version`: link to the response design artifact.
- `status`: planned, visited, dropped, replaced, labeled.

The model should also store a `site_history` table for time-series workflows. That table can track when and why a site changed status.

### 7.4 Evaluation plan: software correctness and scientific value

Evaluation should include both software tests and applied case studies.

#### 7.4.1 Software correctness tests

- Geometry edge cases: holes, multi-polygons, coastal shapes, thin corridors.
- Reproducibility: same seed yields same sample across sessions.
- Statistical sanity checks: spatial balance diagnostics fall in expected ranges.

#### 7.4.2 Applied validation case studies

A tool-oriented research plan can use case studies that expose each gap:

- A heterogeneous urban area with rare classes to test the planning module (Congalton, 1991; Olofsson et al., 2014).
- A change map with small change strata to test time-series and change oversampling workflows (Olofsson et al., 2014; Tyukavina et al., 2025).
- A multi-analyst labeling workflow to test protocol enforcement and reference uncertainty capture (Pengra et al., 2020; Foody, 2015).

The tool can also support a user study. It can compare method reporting quality and reproducibility between a group using the tool and a group using ad hoc point generation.

### 7.5 Why these tool choices are publishable contributions

A tool can be publishable when it does at least one of these things:

- makes a statistically sound method accessible in a new environment,
- formalizes a workflow step that the literature identifies as critical but that tools ignore, or
- produces measurable improvements in transparency and reproducibility.

The literature review used in this project explicitly points to a gap between theory and practice, and suggests that easy-to-use software could help practitioners implement proper sampling (Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review, 2025). The blueprint above responds directly to that gap.
### 7.6 Minimum viable product specification

A tool can become complex fast. A minimum viable product (MVP) can still address the feedback if it includes the right core parts.

An MVP for Random GeoPoints can include:

- BAS-Lite sampling for polygons,
- stratified random sampling by strata layer,
- a protocol builder that exports a protocol file and a field sheet,
- a simple planner that shows per-class margin of error curves under binomial assumptions,
- NNI and grid coverage diagnostics, and
- an export package that bundles points, protocol, and a methods-ready report.

This MVP already moves beyond most point generators. It closes the response design gap and the planning gap at the same time.

### 7.7 A staged roadmap aligned with literature and user need

A staged roadmap can keep the tool credible and maintainable:

- Stage 1: Implement BAS-Lite and strong metadata export. Validate geometry and seeds.
- Stage 2: Add multi-annotator QC features and agreement summaries (Pengra et al., 2020).
- Stage 3: Add longitudinal sample management with stable IDs and master sample support.
- Stage 4: Add post-label spatial autocorrelation diagnostics (Moran’s I) and scale checks (Ripley’s K).
- Stage 5: Explore unequal probability designs and model-assisted options, with clear warnings and published validation tests.

This roadmap aligns tool features with the maturity of methods. It avoids shipping a black box.


## 8. Conclusions and research agenda

This sequel focuses on six gaps where existing literature and existing tools do not yet meet. Each gap leads to a concrete tool feature and a concrete research agenda.

1. **Accessibility of spatially balanced designs.** A browser tool can implement BAS-Lite now and can expand toward full BAS and GRTS later, guided by established references (Stevens & Olsen, 2004; Robertson et al., 2013; Dumelle et al., 2023).
2. **Response design standardization.** A protocol builder can convert response design from an informal habit to an auditable artifact, consistent with the sampling-response-analysis framing in the literature (Stehman & Czaplewski, 1998; Olofsson et al., 2014).
3. **Rare class planning.** A precision planner can make sample size trade-offs visible and reduce guesswork, building on sample size guidance and on stratified estimation frameworks (Foody, 2009; Olofsson et al., 2013; Olofsson et al., 2014).
4. **Spatial dependence diagnostics.** Simple geometric and attribute checks can warn users when a realized sample clusters or when errors cluster, using well-known statistics (Moran, 1950; Ripley, 1976).
5. **Time-series workflows.** Master samples and longitudinal augmentation can reduce cost and improve trend insight, while keeping design metadata explicit (van Dam-Bates et al., 2018; Tyukavina et al., 2025).
6. **Reference uncertainty.** Tools can record and report reference uncertainty and support sensitivity analysis rather than assuming perfect labels (Foody, 2015; Stehman & Foody, 2019; Pengra et al., 2020).

The broader message is simple. A large part of “rigor” is not a new estimator. It is making the existing estimators usable, traceable, and hard to misuse. A tool can help by enforcing documentation, exporting design metadata, and providing diagnostic feedback.

The research agenda that follows is also clear. Each module creates data that can support new papers: user behavior under tool support, effects of protocol choices, performance of spatially balanced designs under constraints, and impacts of reference uncertainty on reported accuracy. These are not edge topics. They sit at the center of making maps trustworthy.
## Appendix A. BAS-Lite pseudocode for polygon sampling

This appendix presents high-level pseudocode for the BAS-Lite method described in Section 1.4. It is not language-specific.

```text
Inputs:
  polygon P (in projected coordinates)
  target sample size n
  seed s (optional)

Step 0: Setup
  B = bounding_box(P)
  i = 1
  S = empty list

Step 1: Generate candidates
  while length(S) < n:
    u = halton(i, base=2, seed=s)
    v = halton(i, base=3, seed=s)
    x = B.xmin + u * (B.xmax - B.xmin)
    y = B.ymin + v * (B.ymax - B.ymin)
    if point_in_polygon((x,y), P):
       append (x,y) to S
    i = i + 1

Outputs:
  S: ordered sample list of size n
```

Robertson et al. (2013) discuss the Halton sequence as the spatial engine for BAS. A full BAS implementation also supports targeted inclusion probabilities via acceptance sampling. BAS-Lite does not include that step. It aims for a simple, well spread equal-probability design.

## Appendix B. Example response design protocol schema

This appendix provides a compact example schema. A tool can store this as JSON. It can also render it as a field sheet.

```json
{
  "protocol_id": "LC2025_Gainesville_01",
  "protocol_version": "1.0.0",
  "map_product": {
    "name": "LandCover_2025",
    "map_date": "2025-07-01",
    "crs": "EPSG:32617",
    "notes": "Example protocol schema for illustration."
  },
  "sample_unit": {
    "type": "buffered_point",
    "buffer_radius_m": 15,
    "mmu_rule": {
      "dominant_threshold": 0.6,
      "mixed_label_code": "MIXED"
    }
  },
  "labeling": {
    "class_legend": [
      {"code": "FOR", "name": "Forest"},
      {"code": "URB", "name": "Urban"},
      {"code": "WAT", "name": "Water"},
      {"code": "AGR", "name": "Agriculture"},
      {"code": "MIXED", "name": "Mixed"}
    ],
    "confidence_scale": {
      "levels": [1,2,3,4,5],
      "definitions": {
        "1": "low",
        "3": "med",
        "5": "high"
      }
    },
    "evidence_sources": ["field", "hires_imagery", "time_series", "local_data"],
    "dispute_rule": "If two analysts disagree, flag and escalate to lead analyst."
  }
}
```

This schema encodes the types of choices discussed by Stehman and Czaplewski (1998) and reinforced in Olofsson et al. (2014). It also supports QC workflows like those described by Pengra et al. (2020).

## Appendix C. Precision planning notes and assumptions

The simple sample size approximation in Section 3.3 treats a class accuracy as a proportion with independent trials. It supports first-order planning (Foody, 2009). It does not replace design-based variance estimators for stratified or clustered designs.

A tool should therefore:

- label the approximation as a planning aid,
- warn when the design includes clustering or strong spatial dependence, and
- export design information so users can apply design-consistent variance estimators in analysis (Olofsson et al., 2013; Olofsson et al., 2014).

## Appendix D. Diagnostic formula reminders

These formulas support transparency. A tool should display them or link to them in its report.

### D.1 Moran’s I

Given values $x_i$, mean $\bar{x}$, weights $w_{ij}$, and $W = \sum_i \sum_j w_{ij}$:

$$I = \frac{n}{W} \; \frac{\sum_i \sum_j w_{ij}(x_i - \bar{x})(x_j - \bar{x})}{\sum_i (x_i - \bar{x})^2}$$

Moran (1950) provides the classic definition. A tool should state how it defines weights.

### D.2 Nearest Neighbor Index (NNI)

Let $\bar{r}$ be the mean nearest-neighbor distance in the sample. Under complete spatial randomness in an area $A$ with $n$ points, an approximate expected nearest neighbor distance is $r_e \approx 0.5 \sqrt{A/n}$. The index is $\text{NNI} = \bar{r} / r_e$.

This approximation is a planning-level diagnostic. Boundary effects and irregular polygons can affect the expectation. A tool can address this by using simulation under the same boundary instead of relying on the approximation.

### D.3 Ripley’s K

Ripley (1976) defines $K(r)$ as a second-order summary of point processes. A tool can estimate $K(r)$ across distances and compare to simulations under complete spatial randomness within the same boundary.

## Appendix E. Methods text blocks for reporting

A tool can generate draft methods text that users can edit. This improves reporting and reduces the gap between what the literature recommends and what papers report (Olofsson et al., 2014; Stehman & Foody, 2019).

An example text block can include:

- the sampling design name and parameters,
- the strata definition and allocation rule,
- the response design protocol ID and version,
- the reference data sources used,
- the QC steps taken (duplicate labeling rate), and
- the estimator family used for accuracy and area with confidence intervals.

The goal is not to automate scientific writing. The goal is to reduce omissions that weaken reproducibility.

## 9. References

Benedetti, R., Piersimoni, F., & Postiglione, P. (2017). Spatially balanced sampling: A review and a reappraisal. *International Statistical Review*, 85(3), 439–454. https://doi.org/10.1111/insr.12216

Brown, J. A., et al. (2015). Spatially balanced sampling: Application to environmental surveys. *Procedia Environmental Sciences*, 27, 46–50. https://doi.org/10.1016/j.proenv.2015.07.104

Congalton, R. G. (1991). A review of assessing the accuracy of classifications of remotely sensed data. *Remote Sensing of Environment*, 37(1), 35–46.

Dumelle, M., Kincaid, T. M., Olsen, A. R., & Weber, M. H. (2023). spsurvey: Spatial sampling design and analysis in R. *Journal of Statistical Software*, 105(3), 1–29. https://doi.org/10.18637/jss.v105.i03

Foody, G. M. (2009). Sample size determination for image classification accuracy assessment and comparison. *International Journal of Remote Sensing*, 30(20), 5273–5291. https://doi.org/10.1080/01431160903130937

Foody, G. M. (2015). Valuing map validation: The need for rigorous area estimation in map accuracy assessment. *Ecological Informatics*, 25, 82–87. https://doi.org/10.1016/j.ecoinf.2014.10.007

Moran, P. A. P. (1950). Notes on continuous stochastic phenomena. *Biometrika*, 37(1–2), 17–23.

Olofsson, P., Foody, G. M., Stehman, S. V., & Woodcock, C. E. (2013). Making better use of accuracy data in land change studies: Estimating accuracy and area and quantifying uncertainty using stratified estimation. *Remote Sensing of Environment*, 129, 122–131. https://doi.org/10.1016/j.rse.2012.10.031

Olofsson, P., et al. (2014). Good practices for estimating area and assessing accuracy of land change. *Remote Sensing of Environment*, 148, 42–57. https://doi.org/10.1016/j.rse.2014.02.015

Pengra, B. W., et al. (2020). Quality control and assessment of interpreter consistency for global land cover reference data. *Remote Sensing of Environment*. https://doi.org/10.1016/j.rse.2019.111261

Radoux, J., et al. (2020). Response designs for the thematic map accuracy assessment: Review and perspectives. *Remote Sensing*, 12(2), 257. https://doi.org/10.3390/rs12020257

Ripley, B. D. (1976). The second-order analysis of stationary point processes. *Journal of Applied Probability*, 13(2), 255–266.

Robertson, B. L., Brown, J. A., McDonald, T., & Jaksons, P. (2013). BAS: Balanced acceptance sampling of natural resources. *Biometrics*, 69(3), 776–784. https://doi.org/10.1111/biom.12059

Stehman, S. V. (2000). Practical implications of design-based sampling inference for thematic map accuracy assessment. *Remote Sensing of Environment*.

Stehman, S. V., & Czaplewski, R. L. (1998). Design and analysis for thematic map accuracy assessment: Fundamental principles. *Remote Sensing of Environment*, 64(3), 331–344. https://doi.org/10.1016/S0034-4257(98)00010-8

Stehman, S. V., & Foody, G. M. (2019). Key issues in rigorous accuracy assessment of land cover products. *Remote Sensing of Environment*, 231, 111199. https://doi.org/10.1016/j.rse.2019.05.018

Stehman, S. V., & Wickham, J. D. (2011). Pixels, blocks of pixels, and polygons: Choosing a spatial unit for thematic accuracy assessment. *Remote Sensing of Environment*, 115(12), 3044–3055. https://doi.org/10.1016/j.rse.2011.06.007

Stevens, D. L., Jr., & Olsen, A. R. (2004). Spatially balanced sampling of natural resources. *Journal of the American Statistical Association*, 99(465), 262–278. https://doi.org/10.1198/016214504000000250

Strahler, A. H., et al. (2006). *Global Land Cover Validation: Recommendations for Evaluation and Accuracy Assessment of Global Land Cover Maps*. CEOS Working Group on Calibration and Validation (WGCV).

Theobald, D. M., Stevens, D. L., Jr., White, D., Urquhart, N. S., Olsen, A. R., & Norman, J. B. (2007). Using GIS to generate spatially balanced random survey designs for natural resource applications. *Environmental Management*, 40, 134–146.

Tyukavina, A., et al. (2025). *Land Cover and Change Map Accuracy Assessment and Area Estimation: Protocol* (CEOS WGCV LPV, Version 1, September 2025).

van Dam-Bates, P. (n.d.). *spbal: Balanced Acceptance Sampling* (R package). Comprehensive R Archive Network (CRAN).

Random Point and Area Sampling in Geospatial Accuracy Validation: A Literature Review. (n.d.). Unpublished literature review PDF provided in the project materials.