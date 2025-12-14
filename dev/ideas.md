# Master List of Feature Ideas: Random GeoPoints

**Count:** 100+ High-Impact Features (Condensed for Quality)
**Focus:** Scientific Rigor, Usability, and Performance.

## Module: Sampling Engine

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **GRTS Implementation** | Add GRTS spatial balance algo (Stevens/Olsen). | High | Sci |
| **LHC Sampling** | Latin Hypercube Sampling for multi-dim strata. | Med | Sci |
| **Min. Dist. Filter** | Enforce min distance between sample points. | High | Sci |
| **Exclusion Masks** | Mask out "No-Go" zones (e.g., water, cloud). | High | Ops |
| **Road Buffering** | Sample within X meters of roads (Access bias). | Med | Ops |
| **Cost-Based Alloc.** | Optimize n based on travel time/cost raster. | Low | Ops |
| **Legacy Site Import** | Force inclusion of existing monitoring plots. | High | Ops |
| **Panel Design** | Generate rotating panels for annual monitoring. | High | Sci |
| **Oversample Rare** | Auto-boost n for classes <5% area. | High | Sci |
| **Cluster - Box** | Generate square cluster plots (e.g. 3x3 pixel). | Med | Sci |
| **Cluster - L-Shape** | L-shaped clusters for textural analysis. | Low | Sci |
| **Transect Gen.** | Generate linear transects instead of points. | Med | Sci |
| **Two-Stage Sample** | Select Primary Units then Secondary Units. | Med | Sci |
| **Strata Splitting** | Auto-split large disjoint strata polygons. | Low | Tech |
| **Vector Tile Support**| Sample directly from hosted vector tiles. | Low | Tech |
| **Raster Stratification**| Stratify based on uploaded GeoTIFF values. | High | Sci |
| **Slope Constraints** | Exclude areas with slope > X degrees (Safety). | Med | Ops |
| **Aspect Stratification**| Stratify samples by N/S/E/W facing slopes. | Low | Sci |
| **Sentinel-2 Grid** | Align samples to Sentinel-2 pixel centers. | Med | Sci |
| **Landsat Grid** | Align samples to Landsat pixel centers. | Med | Sci |
| **Hex-Binning** | Aggregated sampling within hex bins. | Med | Vis |
| **Adaptive Sampling** | Add points where variance is high (Iterative). | Low | Sci |
| **Boundary Snapping** | Option to snap/avoid stratum boundaries. | Med | Sci |
| **Seed Export** | Copy PRNG seed to clipboard. | High | Rep |
| **Config JSON Export**| Save full sampling config file. | High | Rep |

## Module: Analysis Engine (Olofsson et al.)

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **Eq. 1 Impl.** | Area-weighted proportion estimation. | High | Sci |
| **Eq. 5 Impl.** | Standard Error of User's Accuracy. | High | Sci |
| **Eq. 6 Impl.** | Standard Error of Producer's Accuracy. | High | Sci |
| **Adj. Area Est.** | Bias-corrected area estimates (Eq 8). | High | Sci |
| **Fuzzy Matrix** | Support fuzzy/linguistic correctness levels. | Med | Sci |
| **Weight Calc.** | Auto-calculate stratum weights ($W_h$). | High | Sci |
| **Quantity Disagr.** | Calc Quantity Disagreement (Pontius). | Med | Sci |
| **Alloc. Disagr.** | Calc Allocation Disagreement (Pontius). | Med | Sci |
| **Kappa (Legacy)** | Include Kappa with "Deprecated" warning. | Low | Sci |
| **McNemar Test** | Test stat. sig. diff between two maps. | Low | Sci |
| **Ref. Bias Adj.** | Adjust estimations for imperfect reference data. | High | Sci |
| **Sens. Analysis** | Slider to test effect of ref error on OA. | Med | Vis |
| **Outlier Detect** | Flag strata with suspiciously low accuracy. | Med | Vis |
| **Auto-Grouping** | Merge confusion matrix classes interactively. | Med | UX |
| **Sparse Class Warn** | Alert if cell count < 5 (Stats validity). | High | Sci |
| **Margin of Err Plot**| Visual bar for +/- SE on all metrics. | High | Vis |

## Module: Response Design (Protocol)

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **MMU Visualizer** | Show pixel block size on map relative to zoom. | Med | UX |
| **Conf. Matrix Tmpl** | Export blank CSV template for labellers. | High | Ops |
| **Label Hierarchy** | Define Class > Sub-class structure. | Med | Sci |
| **Mixed Label Rule** | Form to define "Majority vs Plurality" rules. | High | Sci |
| **Edge Rule Builder** | Wizard for boundary handling protocols. | Med | Sci |
| **Date-Stamp Rule** | Define valid imagery date range for ref. | High | Sci |
| **Blind QA Flag** | Mark % of points for blind re-validation. | High | Ops |
| **Analyst ID Field** | Add field for analyst name in schema. | Med | Ops |
| **Time-Spent Field** | Track seconds/minutes per point (Cost est). | Low | Ops |
| **Evidence Dropdown** | Configurable list of allowed ref sources. | Med | Ops |
| **PDF Header Logo** | Upload org logo for Field Sheet PDF. | Low | UX |
| **QR Code Link** | QR code on PDF linking to GeoJSON download. | Low | UX |
| **Mobile Form Gen** | Generate ODK/Kobo schema from protocol. | High | Ops |
| **Glossary Append** | Append class definitions to Protocol PDF. | Med | Ops |

## Module: Visualization & UI/UX

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **Dark/Light Mode** | Toggle theme for field/office use. | Med | UX |
| **Glassmorphism** | Polish UI with blur effects (Modern look). | Low | UX |
| **Strata Color Map** | Auto-color map based on stratum ID. | Med | Vis |
| **Cluster Heatmap** | Toggle heatmap to see density hotspots. | Med | Vis |
| **Projection Disp.** | Show current EPSG code in status bar. | High | Sci |
| **Scale Bar** | Dynamic scale bar (Metric/Imperial). | High | Vis |
| **Coord Display** | Mouseover lat/lon and projected coords. | Med | UX |
| **Mini-Map** | Overview map for large AOIs. | Low | UX |
| **Layer Opacity** | Slider for strata/AOI transparency. | High | UX |
| **Point Size** | Slider to adjust sample point radius. | Med | UX |
| **Label Toggle** | Show/Hide Point IDs on map. | Med | UX |
| **Fullscreen Mode** | Toggle map to full viewport. | Med | UX |
| **Keyboard Shorts.** | Hotkeys for Draw (D), Gen (G), Export (E). | Low | UX |
| **Mobile Touch** | Optimize draw controls for touch screens. | High | UX |
| **Loading Spinner** | Precise progress bar for large generations. | Med | UX |
| **Toast Notifs** | Non-blocking success/error alerts. | Med | UX |
| **Help Tooltips** | Hover '?' icons for sci concepts. | High | Edu |
| **Lit. Citations** | Clickable citations (e.g., "Olofsson 2014"). | Med | Edu |
| **Quick Tour** | 5-step onboarding wizard. | Low | UX |
| **Layout Persist** | Save sidebar width/state to LocalStorage. | Low | UX |

## Module: Data & IO

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **Shapefile Drag** | Support .shp via shpjs (client-side). | High | Ops |
| **KML/KMZ Import** | Support Google Earth formats. | High | Ops |
| **GPX Export** | Export for Garmin/Handheld GPS. | High | Ops |
| **CSV Dialect** | Configurable delimiter (comma/semi-colon). | Low | Tech |
| **Excel Export** | Direct .xlsx download (SheetJS). | Med | Ops |
| **GeoPackage Exp** | Export as .gpkg (future proofing). | Low | Tech |
| **WKT Support** | Paste Well-Known Text geometry. | Low | Tech |
| **Drag Folder** | Support dragging folder of shapefiles. | Med | Ops |
| **Bad Geom Fix** | Auto-repair self-intersections (simple). | High | Tech |
| **CRS Auto-Detect** | Detect input CRS from Prj file (if drop). | High | Tech |
| **Attribute Table** | View GeoJSON properties in modal grid. | Med | UX |
| **Filter Strata** | Filter loaded strata by attribute query. | Med | UX |

## Module: Diagnostics (Pre-Field)

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **Moran's I** | Global Moran's I for spatial autocorr. | Med | Sci |
| **Ripley's K** | Multi-scale clustering analysis graph. | Low | Sci |
| **NNI Score** | Nearest Neighbor Index (Clumpiness). | High | Sci |
| **Coverage Hex** | Bin count map to show sampling gaps. | Med | Vis |
| **Cost Est.** | Est. field time based on path length. | Low | Ops |
| **TSP Route** | Traveling Salesman path for site visit. | Low | Ops |
| **Access Check** | Flag points >X km from roads (OSM). | Med | Ops |
| **Basemap Check** | Flag points with cloud in max-res zoom. | Low | Ops |

## Module: Performance & Core

| Title (Max 20) | Description (Max 50) | Priority | Impact |
| :--- | :--- | :--- | :--- |
| **Web Workers** | Move sampling math to background thread. | High | Perf |
| **WASM Proj4** | Rust/WASM projection engine for speed. | Low | Perf |
| **Spatial Index** | Use RBush/Flatbush for massive polygon sets. | High | Perf |
| **Chunked Gen** | Yield points in chunks to keep UI responsive. | Med | Perf |
| **Mem Monitor** | Warn if AOI nodes > Browser limit. | High | Stab |
| **Offline Cache** | Service Worker for full offline usage. | High | Ops |
| **Canvas Render** | Use Canvas/GL for >10k points display. | Med | Perf |
| **Lazy Load** | Load heavy libraries only when needed. | Med | Perf |
